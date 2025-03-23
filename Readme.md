# 클라우드 인프라 구축 계획 – SW융합대학 실습 환경

이 문서는 최대 1,200대의 가상 머신(VM)을 제공하는 클라우드 인프라를 구축하기 위한 종합 설계안을 요약한 것입니다. 실제 동시 활성 VM은 약 400대로 가정하며, 고성능 컴퓨트 노드와 분리된 스토리지, 네트워크, 사용자 및 자원 관리 체계를 통해 안정적이고 효율적인 운영을 목표로 합니다.

---

## 1. 전체 시스템 개요

- **목표:**  
  - 학생들에게 개인 실습 및 수업용 VM 제공  
  - 각 학생당 VM 사양: **vCPU 4개, 메모리 8GB, 디스크 128GB**  
  - 최대 전체 VM 수: 1,200대 (동시 활성 약 400대)

- **핵심 구성요소:**  
  - **컴퓨트 노드:** Dual-socket Xeon Gold 기반 고성능 서버 (약 40 물리 코어, 1TB RAM, NVMe SSD)  
    - 동시 활성 400대 VM 운영 기준, CPU 오버커밋 4:1 정책 적용  
    - 약 **10대**의 컴퓨트 노드로 운영
  - **스토리지 서버:**  
    - 중앙 스토리지(예: Ceph/NFS/iSCSI)로 VM 이미지, 볼륨, 스냅샷 관리  
    - 전용 스토리지 네트워크(10GbE 이상)로 컴퓨트 노드와 연결
  - **네트워크 구성:**  
    - 내부 사설 IP: IPv4 /23 대역 (최대 약 510개 IP → 동시 활성 400대 충분)  
    - 외부 접속: NAT, VPN, 또는 Bastion Host를 통해 동적 공인 IP 할당 (실제 공인 IP는 제한적)
  - **사용자 및 자원 관리:**  
    - OpenStack 기반 멀티테넌시:  
      - 학생 개별 계정과 프로젝트(테넌트) 단위로 관리  
      - 수업별로 별도 프로젝트 생성, 자원 쿼터 정책 및 RBAC 적용  
    - 자동화 도구 (Terraform/Heat/Ansible)를 통한 일괄 배포 및 관리

---

## 2. 인프라 구성도

아래는 주요 인프라 구성 요소와 이들 간의 관계를 나타낸 다이어그램입니다.

```mermaid
flowchart TD
  %% 컨트롤러 및 관리 계층
  A[컨트롤러 노드<br/>(Keystone, Nova API, Horizon)]
  B[네트워크 관리 노드<br/>(Neutron: NAT, VPN, 라우터)]
  
  %% 컴퓨트 계층 (고성능 컴퓨트 노드)
  subgraph 컴퓨트 노드 (약 10대)
    C1[컴퓨트 노드 1<br/>(40 코어, 1TB RAM)]
    C2[컴퓨트 노드 2<br/>(40 코어, 1TB RAM)]
    C3[컴퓨트 노드 3<br/>(40 코어, 1TB RAM)]
    C4[컴퓨트 노드 4<br/>(40 코어, 1TB RAM)]
    C5[컴퓨트 노드 5<br/>(40 코어, 1TB RAM)]
    C6[컴퓨트 노드 6<br/>(40 코어, 1TB RAM)]
    C7[컴퓨트 노드 7<br/>(40 코어, 1TB RAM)]
    C8[컴퓨트 노드 8<br/>(40 코어, 1TB RAM)]
    C9[컴퓨트 노드 9<br/>(40 코어, 1TB RAM)]
    C10[컴퓨트 노드 10<br/>(40 코어, 1TB RAM)]
  end

  %% 스토리지 계층
  D[중앙 스토리지 서버<br/>(Ceph / NFS / iSCSI)]
  
  %% 외부 접속 계층
  E[VPN & Bastion Host<br/>(제한된 공인 IP)]
  
  %% 네트워크 연결
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
  
  %% 내부 네트워크
  subgraph 내부 네트워크
    F[사설 IP 대역 (/23)]
  end
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
