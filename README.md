# 인프라 운영 기록

업무하면서 자주 봤던 Linux, Nginx, DB, 배포, Kubernetes, 모니터링 관련 내용을 정리한 저장소입니다.
처음에는 개인 작업 메모에 가까웠고, 나중에 다시 보기 쉽도록 분야별로 나눠 정리했습니다.

`구축가이드`에는 설치 과정이나 캡처가 있는 상세 문서를 두었고, `실무운영`에는 작업하면서 따로 기억해둘 만한 운영 포인트를 짧게 정리했습니다.

## 정리한 내용

| 구분 | 내용 |
| --- | --- |
| 리눅스/보안 | Rocky Linux 기본 설정, SSH/계정 정책, OpenSSL, YARA |
| 웹/Nginx | Nginx 설치, Reverse Proxy, HTTPS, Header 설정 |
| DB | PostgreSQL, MariaDB, Redis, 백업, 이관, 이중화 |
| 이중화 | Keepalived, PostgreSQL Replication, MariaDB MHA, Redis Sentinel |
| 배포 | Jenkins, GitLab, 수동 배포 절차 |
| Kubernetes | NCP Kubernetes, Docker, ArgoCD, Ingress |
| 모니터링 | Prometheus, Grafana, Jennifer, NGrinder |

## 먼저 볼 만한 문서

| 구분 | 문서 | 메모 |
| --- | --- | --- |
| 리눅스 | [Rocky Linux 기본 설정](./실무운영/01_리눅스_보안/Rocky_Linux_기본설정_보안점검.md) | 새 서버 작업 전에 자주 확인하는 항목 |
| Nginx | [WAS 경로 프록시](./실무운영/02_웹프록시_Nginx/Nginx_WAS_경로_프록시.md) | WAS 포트를 직접 열지 않고 Nginx에서 경로 제어 |
| DB | [DB 이관 순서](./실무운영/04_DB_운영/DB_이관_작업순서.md) | 백업, 중지, 이관, 검증 순서 정리 |
| 이중화 | [PostgreSQL 복제와 Failover](./실무운영/03_이중화_장애전환/PostgreSQL_복제_pg_auto_failover.md) | 복제 구성과 장애 전환 확인 포인트 |
| 배포 | [수동 배포 메모](./실무운영/05_배포_CICD/애플리케이션_배포_작업순서.md) | 배포 전후 확인할 것들 |
| Kubernetes | [NCP Kubernetes 접근](./실무운영/06_쿠버네티스_클라우드/NCP_Kubernetes_접근설정.md) | kubeconfig와 인증 도구 설정 |
| 모니터링 | [Nginx 메트릭 수집](./실무운영/07_모니터링_성능점검/Nginx_Prometheus_Grafana_메트릭수집.md) | Prometheus/Grafana 연동 메모 |

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
