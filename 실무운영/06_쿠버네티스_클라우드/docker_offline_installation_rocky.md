# docker_offline_installation_rocky

## 1. 개요
인터넷 제약이 있거나 버전을 고정해야 하는 환경에서 Rocky Linux에 Docker를 수동 설치하는 절차와 운영 포인트를 정리했습니다.

## 2. 작업 배경
운영 환경에 따라 공식 저장소 접근이 제한되거나,
특정 버전의 Docker / Compose를 고정해서 써야 하는 경우가 있었습니다.

## 3. 문제/요구사항
- 필요한 RPM을 사전에 확보해야 함
- Compose 플러그인 버전과 바이너리 버전을 맞춰야 함
- 컨테이너 이미지/파일을 다른 서버로 이관하는 절차가 필요할 수 있음

## 4. 판단 및 선택 이유
- 온라인 설치보다 번거롭지만 버전 통제가 쉬움
- 폐쇄망 또는 제한망에서 반복 가능한 설치 절차로 활용 가능
- 운영 중에는 설치보다 이미지 이식과 파일 추출(`docker cp`, `save/load`)이 더 실무적일 수 있음

## 5. 적용 절차 요약
- 필수 RPM 확보
- Docker CE / CLI / Containerd / Compose Plugin 설치
- `docker-compose` 바이너리 배치 및 실행권한 부여
- 심볼릭 링크 설정
- 버전 확인
- 필요 시 이미지 commit/save/load 절차 수행

## 6. 검증 방법
- `docker --version`
- `docker-compose -v`
- 샘플 컨테이너 실행
- 이미지 save/load 테스트

## 7. 운영 체크포인트
- RPM 간 버전 호환성 확인
- Compose Plugin과 standalone compose 바이너리를 혼동하지 않기
- 운영 문서에는 Registry 사용 여부와 이미지 보관 전략도 추가하는 것이 좋음
