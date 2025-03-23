# 클라우드 인프라 구축 설계 (SOLID Cloud - 단국대 자체 클라우드)

## 1. 개요
이 설계는 최대 1,200명의 학생에게 각 vCPU 4개, 메모리 8GB, 디스크 128GB를 할당하는 가상 머신(VM) 환경을 구축하는 것을 목표로 합니다.  
- **총 VM 할당:** 최대 1,200대 (전체 할당)  
- **동시 활성:** 최대 400대 정도 (실제 사용 시)  
- **목표:** 학생 개인 실습 및 수업용 VM 제공, 외부 접속 시 NAT/VPN/Bastion을 통한 공인 IP 효율적 활용, 중앙 스토리지 서버를 통한 디스크 관리  
- **플랫폼:** OpenStack 기반  
- **주요 하드웨어:**  
  - **고성능 컴퓨트 노드:** Dual-socket Xeon Gold 기반 (약 40 물리 코어, 1TB RAM, NVMe SSD) – 약 10대 (동시 활성 400대 기준)  
  - **컨트롤러/네트워크 노드:** OpenStack 컨트롤러, 네트워크 서비스 (Neutron)를 전담하는 노드  
  - **스토리지 서버:** 중앙 스토리지 (NFS / iSCSI / Ceph 기반)로, 전용 스토리지 네트워크(10GbE 이상)를 통해 연결

## 2. 인프라 구성도
아래 Mermaid 다이어그램은 주요 구성 요소와 그 연결 관계를 시각화한 예시입니다.

```mermaid
flowchart TD
    %% 컨트롤러/관리 계층
    subgraph 관리 계층
      A[컨트롤러 노드<br/>(Keystone, Nova, Horizon, DB)]
      B[네트워크 노드<br/>(Neutron, NAT, VPN, Bastion)]
    end

    %% 컴퓨트 계층
    subgraph 컴퓨트 계층
      C1[컴퓨트 노드 1<br/>(40코어, 1TB RAM)]
      C2[컴퓨트 노드 2<br/>(40코어, 1TB RAM)]
      C3[컴퓨트 노드 3<br/>(40코어, 1TB RAM)]
      C4[컴퓨트 노드 4<br/>(40코어, 1TB RAM)]
      C5[컴퓨트 노드 5<br/>(40코어, 1TB RAM)]
      C6[컴퓨트 노드 6<br/>(40코어, 1TB RAM)]
      C7[컴퓨트 노드 7<br/>(40코어, 1TB RAM)]
      C8[컴퓨트 노드 8<br/>(40코어, 1TB RAM)]
      C9[컴퓨트 노드 9<br/>(40코어, 1TB RAM)]
      C10[컴퓨트 노드 10<br/>(40코어, 1TB RAM)]
    end

    %% 스토리지 계층
    subgraph 스토리지 계층
      D[중앙 스토리지 서버<br/>(NFS/iSCSI/Ceph + NVMe 캐시)]
    end

    %% 내부 네트워크 연결 (전용 스토리지 네트워크)
    A --- C1 & C2 & C3 & C4 & C5 & C6 & C7 & C8 & C9 & C10
    B --- C1 & C2 & C3 & C4 & C5 & C6 & C7 & C8 & C9 & C10
    C1 ---|스토리지 네트워크| D
    C2 ---|스토리지 네트워크| D
    C3 ---|스토리지 네트워크| D
    C4 ---|스토리지 네트워크| D
    C5 ---|스토리지 네트워크| D
    C6 ---|스토리지 네트워크| D
    C7 ---|스토리지 네트워크| D
    C8 ---|스토리지 네트워크| D
    C9 ---|스토리지 네트워크| D
    C10 ---|스토리지 네트워크| D

    %% 외부 접속
    B --- E[외부 인터넷<br/>(공인 IP via NAT/VPN/Bastion)]
