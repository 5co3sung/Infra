# document_standard

실무 운영 문서를 포트폴리오로 정리할 때 사용하는 작성 기준입니다.
단순 설치 기록이 아니라, 운영자가 어떤 배경에서 판단했고 어떻게 검증했는지 드러나는 문서를 목표로 합니다.

## Directory Standard

```text
실무운영/
├── 01_linux_security/
├── 02_web_proxy/
├── 03_ha_failover/
├── 04_database_operations/
├── 05_devops_cicd/
├── 06_kubernetes_container_cloud/
└── 07_observability_performance/
```

## Category Rules

### 01_linux_security

- OS 기본 설정
- SSH / 계정 정책
- OpenSSL / YARA / 취약점 점검
- 보안 가이드라인

### 02_web_proxy

- Nginx 설치 전략
- Reverse Proxy / Upstream
- HTTPS / Header Hardening
- Nginx 보안 점검

### 03_ha_failover

- Keepalived
- PostgreSQL Replication / pg_auto_failover
- MariaDB MHA
- Redis Sentinel

### 04_database_operations

- DB 이관 런북
- Character Set 정합성
- Audit Log / Trigger
- 백업 전략

### 05_devops_cicd

- Jenkins
- GitLab
- 배포 런북

### 06_kubernetes_container_cloud

- NCP Kubernetes
- kubectl / iam-authenticator
- Docker 오프라인 설치
- Cloud Function 메모

### 07_observability_performance

- NGrinder
- Prometheus / Grafana
- Jennifer 알림 설정
- 성능 테스트 결과 요약

## Naming Convention

- 파일명은 소문자 영문과 언더스코어를 우선 사용합니다.
- 문서명은 의미 단위 중심으로 작성합니다.
- 기술명 오탈자를 정리합니다.
  - `kubernetis` -> `kubernetes`
  - `keep-alived` -> `keepalived`
  - `ngirinder` -> `ngrinder`

## Document Template

```md
# 제목

## 1. 개요
## 2. 작업 배경
## 3. 환경
## 4. 문제/요구사항
## 5. 판단 및 선택 이유
## 6. 적용 절차
## 7. 검증 방법
## 8. 결과 및 운영 효과
## 9. 운영 체크포인트 / 트러블슈팅
## 10. 참고한 원본 문서
```

