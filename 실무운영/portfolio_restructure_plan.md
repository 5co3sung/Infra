# portfolio_restructure_plan

## 목표
OJT 정리 문서를 그대로 공개하지 않고, 이직용 포트폴리오에 맞는 구조로 재배치합니다.

## 추천 구조
```text
구축가이드/
├── 01_linux_security/
├── 02_web_proxy/
├── 03_ha_failover/
├── 04_database_operations/
├── 05_devops_cicd/
├── 06_kubernetes_container_cloud/
├── 07_observability_performance/
└── 99_archive/
```

## 분류 기준
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

### 99_archive
- 교육 메모
- 발표 대본
- 퀴즈 / 개인 메모
- 아직 포트폴리오 문서로 다듬지 않은 초안

## 네이밍 컨벤션
- 소문자 영문 + 언더스코어
- 의미 단위 중심
- 기술명 오탈자 수정
  - `kubernetis` → `kubernetes`
  - `keep-alived` → `keepalived`
  - `ngirinder` → `ngrinder`

## 문서 템플릿
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
