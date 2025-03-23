## 서비스 구성도

```mermaid
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
