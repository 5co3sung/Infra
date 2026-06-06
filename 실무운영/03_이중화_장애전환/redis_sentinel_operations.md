# redis_sentinel_operations

## 1. 개요
Redis 서버와 Sentinel을 운영 관점에서 점검/기동/확인하는 방법을 정리한 문서입니다.

## 2. 작업 배경
애플리케이션이 Redis를 캐시나 세션 저장소로 사용할 때,
프로세스 상태만 아는 수준을 넘어서 Sentinel 구성과 실제 사용 중인 설정파일을 구분해서 볼 필요가 있었습니다.

## 3. 문제/요구사항
- 실제 기동 중인 `redis.conf`, `sentinel.conf` 경로를 혼동하기 쉬움
- Sentinel을 여러 개 띄우는 환경에서는 설정파일과 포트 매핑을 명확히 알아야 함
- 운영 중 중지/기동/재기동 절차를 표준화할 필요가 있음

## 4. 판단 및 선택 이유
- Redis 단독 운영보다 Sentinel 구성이 장애 감지와 Master 재선출 측면에서 유리
- 다만 설치 문서보다 실제 사용 중인 conf 경로와 프로세스 확인 방법이 더 실무적 가치가 큼

## 5. 적용 절차 요약
- `find`, `ps`로 `redis-server`, `redis-sentinel` 위치와 프로세스 확인
- 실제 운영 중인 `redis.conf`, `sentinel.conf` 파일 구분
- 필요 시 프로세스 종료 또는 `redis-cli shutdown` 수행
- Sentinel 다중 기동 시 파일별 포트/경로 분리

## 6. 검증 방법
- `ps -ef | grep redis`
- `find / -name redis.conf`, `find / -name sentinel*`
- 포트 LISTEN 상태 확인
- Master 변경 시 Sentinel 반응 확인

## 7. 운영 체크포인트
- 동일 서버에 여러 Sentinel을 띄울 경우 설정파일 충돌 주의
- 운영 중인 conf와 기본 샘플 conf를 혼동하지 않기
- 애플리케이션이 localhost 또는 VIP 중 무엇을 바라보는지 함께 확인


