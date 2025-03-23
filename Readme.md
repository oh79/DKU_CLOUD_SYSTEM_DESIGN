# 클라우드 인프라 구축 계획 – SW융합대학 실습 환경

이 문서는 최대 1,200대의 가상 머신(VM)을 제공하는 클라우드 인프라를 구축하기 위한 종합 설계안을 설명합니다.  
전체 VM 수는 1,200대로 할당하되, 실제 동시 활성은 최대 400대로 가정합니다.

## 1. 전체 시스템 개요

- **목표:**  
  - 학생들에게 개인 실습 및 수업용 VM 제공  
  - 각 학생당 VM 사양: **vCPU 4개, 메모리 8GB, 디스크 128GB**  
  - 최대 전체 VM 수: 1,200대 (동시 활성 400대 기준)

- **핵심 구성요소:**  
  - **컴퓨트 노드:**  
    - 고성능 컴퓨트 노드 (Dual-socket Xeon Gold 기반)  
    - 약 40 물리 코어, 1TB RAM, NVMe SSD (부팅/캐시 용)  
    - 동시 활성 400대 VM 운영을 위해 약 10대 필요 (CPU 오버커밋 4:1 기준)
  - **스토리지 서버:**  
    - 중앙 스토리지 서버는 Ceph, NFS 또는 iSCSI를 활용해 VM 이미지, 볼륨, 스냅샷 등을 관리  
    - 전용 스토리지 네트워크 (10GbE 이상)로 컴퓨트 노드와 연결
  - **네트워크 구성:**  
    - 내부 사설 IP: IPv4 /23 대역 (약 510개 IP, 동시 활성 400대 충분)  
    - 외부 접속: NAT, VPN, Bastion Host를 통해 동적 공인 IP 할당 (실제 공인 IP 수는 제한됨)
  - **사용자 및 자원 관리:**  
    - OpenStack 기반 멀티테넌시로 학생 개별 계정 및 프로젝트(테넌트) 단위 관리  
    - 수업별 자원 할당은 프로젝트별 쿼터 정책 및 RBAC를 적용하며, Terraform/Heat 등으로 자동화

## 2. 인프라 구성도

아래는 주요 인프라 구성 요소와 이들 간의 물리적 연결 관계를 나타낸 Mermaid 다이어그램입니다.

```mermaid
flowchart TD
    A[컨트롤러 노드 - Keystone, Nova API, Horizon]
    B[네트워크 관리 노드 - Neutron NAT, VPN, 라우터]
    
    subgraph 컴퓨트노드_약_10대
      C1[컴퓨트 노드 1 - 40 코어, 1TB RAM]
      C2[컴퓨트 노드 2 - 40 코어, 1TB RAM]
      C3[컴퓨트 노드 3 - 40 코어, 1TB RAM]
      C4[컴퓨트 노드 4 - 40 코어, 1TB RAM]
      C5[컴퓨트 노드 5 - 40 코어, 1TB RAM]
      C6[컴퓨트 노드 6 - 40 코어, 1TB RAM]
      C7[컴퓨트 노드 7 - 40 코어, 1TB RAM]
      C8[컴퓨트 노드 8 - 40 코어, 1TB RAM]
      C9[컴퓨트 노드 9 - 40 코어, 1TB RAM]
      C10[컴퓨트 노드 10 - 40 코어, 1TB RAM]
    end

    D[중앙 스토리지 서버 - Ceph, NFS, iSCSI]
    E[VPN 및 Bastion Host - 제한된 공인 IP]
    F[내부 네트워크 - 사설 IP 대역 /23]
    
    A --- C1
    A --- C2
    A --- C3
    A --- C4
    A --- C5
    A --- C6
    A --- C7
    A --- C8
    A --- C9
    A --- C10

    B --- E
    
    C1 --- D
    C2 --- D
    C3 --- D
    C4 --- D
    C5 --- D
    C6 --- D
    C7 --- D
    C8 --- D
    C9 --- D
    C10 --- D

    C1 --- F
    C2 --- F
    C3 --- F
    C4 --- F
    C5 --- F
    C6 --- F
    C7 --- F
    C8 --- F
    C9 --- F
    C10 --- F
flowchart TD
    U[학생 사용자]
    H[OpenStack Horizon (대시보드)]
    K[Keystone (인증)]
    N[Nova (컴퓨트 관리)]
    G[Glance (이미지 관리)]
    C[Cinder (블록 스토리지)]
    NT[Neutron (네트워크 관리)]
    APT[자동화 도구 (Terraform/Heat)]
    PT[프로젝트/테넌트 관리]
    
    U --> H
    H --> K
    H --> PT
    K --> N
    K --> G
    K --> C
    K --> NT
    APT --> N
    APT --> G
    APT --> C
    APT --> NT
```
