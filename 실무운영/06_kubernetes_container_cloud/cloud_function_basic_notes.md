# cloud_function_basic_notes

## 1. 개요
Cloud Function 서비스의 기본 개념을 운영자 관점에서 빠르게 이해하기 위한 정리 문서입니다.

## 2. 작업 배경
서버를 직접 관리하지 않고 이벤트 기반으로 코드를 실행하는 구조를 이해해야 했고,
기본 용어를 모르면 서비스 선택이나 아키텍처 설명이 어려웠습니다.

## 3. 핵심 개념
- Namespace
- Action
- Sequence Action
- Web Action
- Activation
- Trigger
- Metadata
- Container 재사용 개념

## 4. 판단 포인트
- 서버리스는 서버 관리 부담을 줄이지만, 실행 모델과 권한/이벤트 구조를 이해해야 함
- 단발성 자동화나 이벤트 연동 작업에 적합
- 전통적인 장기 실행형 애플리케이션과는 운영 포인트가 다름

