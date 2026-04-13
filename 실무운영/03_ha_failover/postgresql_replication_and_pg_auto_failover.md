# postgresql_replication_and_pg_auto_failover

## 1. 개요
PostgreSQL 이중화 문서와 `pg_auto_failover` 설치 메모를 기반으로,
수동 복제와 자동 Failover 두 가지 관점을 함께 정리했습니다.

## 2. 작업 배경
DB는 서비스 핵심 상태를 저장하므로 단일 노드 장애에 취약합니다.
읽기 전용 대기 서버 구성, 장애 복구 시간 단축, 접속 문자열 테스트까지 고려한 복제 구조가 필요했습니다.

## 3. 환경
- PostgreSQL 15 계열
- Primary / Standby 복제
- 일부 환경에서 `pg_auto_failover` 검토

## 4. 문제/요구사항
- 복제 전용 계정과 슬롯 관리 필요
- WAL 전송/보관 설정 필요
- 애플리케이션 JDBC가 장애 상황에서도 적절히 동작하는지 확인 필요
- `pg_auto_failover` 설치 시 빌드/의존성 오류가 발생할 수 있음

## 5. 판단 및 선택 이유
### 수동 복제
- 구조를 직접 통제하기 쉬움
- 운영팀이 내부 동작을 이해하기 좋음

### 자동 Failover 도입 검토
- 장애 시 운영자 개입 시간을 줄일 수 있음
- 다만 설치/빌드 복잡도와 운영 난도가 증가함

## 6. 적용 절차 요약
### 복제 구성
1. replication 계정 생성
2. `postgresql.conf`에서 `wal_level`, `max_wal_senders`, `max_replication_slots`, `archive_mode` 등 설정
3. `pg_hba.conf`에 복제 허용 규칙 추가
4. standby 쪽에서 복제 설정 및 슬롯 연동
5. 읽기 전용 동작과 슬롯 상태 확인

### `pg_auto_failover` 검토
- 필요한 개발 패키지 설치
- `pg_config`, PATH, `postgresql-devel`, `clang`, `llvm`, `lz4` 등 의존성 확인
- monitor / keeper 구성 전 빌드 환경 점검

## 7. 검증 방법
- `show replication slots`
- standby에서 read only 동작 확인
- Primary 장애 가정 후 Failover 절차 리허설
- JDBC 옵션(`targetServerType`)에 따른 애플리케이션 기동 테스트

## 8. 결과 및 운영 효과
- 복제 구성 이해도를 문서화할 수 있음
- 단순 설치가 아니라 접속 문자열/Failover 관점까지 보여줄 수 있음
- DB 고가용성 문서를 Web/WAS와 분리해 구조적으로 더 자연스러움

## 9. 운영 체크포인트 / 트러블슈팅
- `pg_auto_failover`는 패키지/소스/개발 라이브러리 호환성이 중요
- standby를 읽기 대상으로 둘 때 애플리케이션 쓰기 동작과 충돌하지 않도록 확인
- archive 경로와 WAL 보존 정책이 빠지면 복구 전략이 약해짐


