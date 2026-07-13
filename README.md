# 인프라 운영 포트폴리오

실무에서 다뤄온 Linux, Nginx, DB 이중화, Kubernetes, CI/CD, 모니터링 경험을 정리한 저장소입니다.
단순히 설치 명령어를 모아둔 문서가 아니라, 왜 이 작업이 필요했는지, 어떤 점을 확인했는지, 운영할 때 무엇을 조심해야 하는지를 중심으로 정리했습니다.

## 이 저장소를 만든 이유

인프라 업무는 결과 화면보다 과정이 더 중요하다고 생각합니다.
서버를 설치하고 서비스를 띄우는 것에서 끝나는 것이 아니라, 보안 기준을 맞추고, 장애 상황을 가정하고, 배포 후 로그와 기능을 확인하는 과정까지 운영자의 책임이라고 봅니다.

그래서 이 저장소는 두 가지 형태로 나누어 정리했습니다.

- `실무운영`: 면접관이 빠르게 볼 수 있도록 운영 관점으로 다시 정리한 문서
- `구축가이드`: 노션에 기록해 둔 상세 설치/구축 절차와 캡처 기반 자료

## 주요 경험

| 영역 | 정리한 내용 |
| --- | --- |
| 리눅스/보안 | Rocky Linux 기본 설정, SSH/계정 정책, OpenSSL, YARA, 보안 점검 |
| 웹 프록시 | Nginx 설치, Reverse Proxy, HTTPS, Header Hardening |
| DB 운영 | PostgreSQL, MariaDB, Redis, MongoDB 운영 및 이중화 구성 |
| 이중화/장애전환 | Keepalived, PostgreSQL Replication, MariaDB MHA, Redis Sentinel |
| 배포/CI/CD | Jenkins, GitLab, 수동 배포 런북, 배포 후 검증 절차 |
| 쿠버네티스/클라우드 | Naver Cloud Kubernetes, Docker, ArgoCD, Ingress, Cloud Function |
| 모니터링/성능 | Prometheus, Grafana, Jennifer, NGrinder |

## 먼저 보면 좋은 문서

| 구분 | 문서 | 내용 |
| --- | --- | --- |
| 리눅스 보안 | [Rocky Linux 기본 설정과 보안 기준](./실무운영/01_리눅스_보안/rocky_linux_baseline_and_hardening.md) | 신규 서버 투입 전 공통으로 맞춰야 하는 운영 기준 |
| 웹 프록시 | [Nginx Reverse Proxy 구성](./실무운영/02_웹프록시_Nginx/nginx_reverse_proxy_for_was_path.md) | WAS 내부 경로를 직접 노출하지 않도록 제어한 구성 |
| 이중화 | [PostgreSQL 복제와 pg_auto_failover](./실무운영/03_이중화_장애전환/postgresql_replication_and_pg_auto_failover.md) | DB 복제와 장애 전환을 검증 관점까지 정리 |
| DB 운영 | [DB 이관 런북](./실무운영/04_DB_운영/db_migration_runbook.md) | DB 분리/이관 작업 시 필요한 절차와 확인 항목 |
| 배포 | [애플리케이션 배포 런북](./실무운영/05_배포_CICD/application_deployment_runbook.md) | 수동 배포 시 백업, 기동, 로그 확인, 기능 검증 기준 |
| 쿠버네티스 | [NCP Kubernetes 접근 구성](./실무운영/06_쿠버네티스_클라우드/ncp_kubernetes_access_setup.md) | 클러스터 생성 후 운영자가 접근 가능한 상태로 만드는 흐름 |
| 모니터링 | [Nginx 메트릭 수집과 Grafana 시각화](./실무운영/07_모니터링_성능점검/nginx_prometheus_grafana_notes.md) | 웹 계층 메트릭을 수집하고 시각화하는 흐름 |

## 폴더 구조

```text
.
├── 실무운영/
│   ├── 01_리눅스_보안/
│   ├── 02_웹프록시_Nginx/
│   ├── 03_이중화_장애전환/
│   ├── 04_DB_운영/
│   ├── 05_배포_CICD/
│   ├── 06_쿠버네티스_클라우드/
│   └── 07_모니터링_성능점검/
└── 구축가이드/
    ├── 1. Nginx/
    ├── 2. DB/
    ├── 3. DevOps/
    ├── 4. WAS/
    └── 5. AI/
```
