# 실무운영

실무 운영 과정에서 반복적으로 다루는 인프라 작업을 채용 포트폴리오용으로 재구성한 문서 모음입니다.
각 문서는 단순 명령어보다 작업 배경, 판단 이유, 검증 방법, 운영 체크포인트를 중심으로 작성했습니다.

## Categories

| Category | Description |
| --- | --- |
| [01_linux_security](./01_linux_security/) | Linux 서버 베이스라인, SSH/계정 정책, 보안 도구, 취약점 점검 |
| [02_web_proxy](./02_web_proxy/) | Nginx 설치 전략, Reverse Proxy, HTTPS, Header Hardening |
| [03_ha_failover](./03_ha_failover/) | Keepalived, PostgreSQL Replication, MariaDB MHA, Redis Sentinel |
| [04_database_operations](./04_database_operations/) | DB 이관, 백업, 문자셋 정합성, 감사 로그/트리거 |
| [05_devops_cicd](./05_devops_cicd/) | Jenkins, GitLab, 수동/자동 배포 운영 런북 |
| [06_kubernetes_container_cloud](./06_kubernetes_container_cloud/) | Kubernetes 접근 구성, Docker, Cloud Function, Cloud Native 운영 |
| [07_observability_performance](./07_observability_performance/) | Prometheus, Grafana, Jennifer, NGrinder 기반 관측/성능 점검 |

## Document Template

각 문서는 가능한 한 아래 흐름을 따릅니다.

1. 개요
2. 작업 배경
3. 환경
4. 문제/요구사항
5. 판단 및 선택 이유
6. 적용 절차
7. 검증 방법
8. 결과 및 운영 효과
9. 운영 체크포인트 / 트러블슈팅

자세한 작성 기준은 [document_standard.md](./document_standard.md)를 참고합니다.
