# Infra Operations Portfolio

Linux, Web Proxy, Database HA, Kubernetes, CI/CD, Monitoring 운영 경험을 정리한 인프라 포트폴리오입니다.
단순 설치 명령어 모음이 아니라, 운영 안정성, 보안 점검, 장애 대응, 검증 절차를 중심으로 정리했습니다.

## Overview

이 저장소는 실제 인프라 운영 과정에서 반복적으로 다루는 작업을 주제별로 재구성한 문서 모음입니다.
채용 담당자와 면접관이 빠르게 역량을 확인할 수 있도록 `실무운영` 폴더에는 운영 관점의 요약 문서를,
`구축가이드` 폴더에는 세부 설치 및 구축 절차를 분리해 두었습니다.

## Core Skills

| Area | Experience |
| --- | --- |
| Linux / Security | Rocky Linux 베이스라인 설정, SSH/계정 정책, OpenSSL, YARA, 보안 점검 |
| Web / Proxy | Nginx 설치 전략, Reverse Proxy, HTTPS, Header Hardening |
| Database Operations | PostgreSQL, MariaDB, Redis, MongoDB 운영 및 이중화 구성 이해 |
| HA / Failover | Keepalived VIP Failover, PostgreSQL Replication, MariaDB MHA, Redis Sentinel |
| DevOps / CI/CD | Jenkins, GitLab, 수동 배포 런북, 배포 검증 절차 |
| Kubernetes / Cloud | Naver Cloud Kubernetes, Docker, ArgoCD, Ingress, Cloud Function |
| Observability | Prometheus, Grafana, Jennifer, NGrinder 기반 모니터링/성능 점검 |

## Recommended Reading

| Category | Document | Why It Matters |
| --- | --- | --- |
| Linux Security | [Rocky Linux Baseline and Hardening](./실무운영/01_linux_security/rocky_linux_baseline_and_hardening.md) | 신규 서버 투입 전 공통 보안/운영 기준을 정리했습니다. |
| Web Proxy | [Nginx Reverse Proxy for WAS Path](./실무운영/02_web_proxy/nginx_reverse_proxy_for_was_path.md) | WAS 내부 경로를 직접 노출하지 않고 Nginx로 제어하는 운영 관점을 정리했습니다. |
| HA / Failover | [PostgreSQL Replication and pg_auto_failover](./실무운영/03_ha_failover/postgresql_replication_and_pg_auto_failover.md) | DB 복제와 장애 전환을 설치 절차뿐 아니라 검증 관점까지 정리했습니다. |
| Database | [DB Migration Runbook](./실무운영/04_database_operations/db_migration_runbook.md) | DB 분리/이관 작업 시 누락하기 쉬운 절차를 런북 형태로 정리했습니다. |
| DevOps | [Application Deployment Runbook](./실무운영/05_devops_cicd/application_deployment_runbook.md) | 수동 배포 작업의 백업, 기동, 로그 확인, 기능 검증 기준을 정리했습니다. |
| Kubernetes | [NCP Kubernetes Access Setup](./실무운영/06_kubernetes_container_cloud/ncp_kubernetes_access_setup.md) | Kubernetes 클러스터 생성 후 운영자가 접근 가능한 상태로 만드는 흐름을 정리했습니다. |
| Observability | [Nginx Prometheus Grafana Notes](./실무운영/07_observability_performance/nginx_prometheus_grafana_notes.md) | 웹 계층 메트릭 수집과 시각화 흐름을 모니터링 관점에서 정리했습니다. |

## Repository Structure

```text
.
├── 실무운영/
│   ├── 01_linux_security/
│   ├── 02_web_proxy/
│   ├── 03_ha_failover/
│   ├── 04_database_operations/
│   ├── 05_devops_cicd/
│   ├── 06_kubernetes_container_cloud/
│   └── 07_observability_performance/
└── 구축가이드/
    ├── 1. Nginx/
    ├── 2. DB/
    ├── 3. DevOps/
    ├── 4. WAS/
    └── 5. AI/
```

## Azure Learning Direction

지원 회사에서 Azure를 주로 사용하는 환경을 고려해, 기존 Linux/Kubernetes/DB 운영 경험을 다음 Azure 서비스에 연결해 학습하고 있습니다.

| Existing Experience | Azure Target |
| --- | --- |
| Linux 서버 운영, 보안 베이스라인 | Azure VM, NSG, Bastion |
| Nginx Reverse Proxy, HTTPS | Azure Application Gateway, Load Balancer |
| PostgreSQL/MariaDB 운영 | Azure Database for PostgreSQL/MySQL |
| Kubernetes 접근 및 운영 | AKS, Azure Container Registry |
| Jenkins/ArgoCD 배포 경험 | Azure DevOps, GitHub Actions |
| Prometheus/Grafana/Jennifer 모니터링 | Azure Monitor, Log Analytics |

목표는 새로운 클라우드 제품명을 외우는 것이 아니라, 기존에 수행해 온 인프라 운영 경험을 Azure 환경에 빠르게 매핑하고 실무에 적용하는 것입니다.

## Resume Summary

```text
인프라 운영 포트폴리오
- Linux 서버 보안 베이스라인, Nginx Reverse Proxy/Hardening, DB 이중화, CI/CD, Kubernetes, 모니터링 운영 문서화
- PostgreSQL Replication/pg_auto_failover, MariaDB MHA, Redis Sentinel, Keepalived 등 고가용성 구성 요소 학습 및 운영 관점 정리
- 단순 설치 절차가 아닌 작업 배경, 요구사항, 검증 방법, 운영 체크포인트 중심의 런북 형태로 정리
- 기존 Linux/Kubernetes 운영 경험을 Azure VM, AKS, Azure Monitor 등 클라우드 운영 환경에 빠르게 적용하는 것을 목표로 학습 중
```

