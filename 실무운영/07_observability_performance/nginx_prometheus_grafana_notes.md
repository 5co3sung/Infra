# nginx_prometheus_grafana_notes

## 1. 개요
Nginx 메트릭을 Prometheus로 수집하고 Grafana에서 시각화하는 기본 흐름을 정리한 문서입니다.

## 2. 작업 배경
웹 계층의 상태를 단순 로그가 아니라 메트릭으로 관찰할 필요가 있었습니다.
연결 수, 요청 수, 활성 세션 같은 지표를 대시보드로 볼 수 있어야 장애 징후를 빠르게 읽을 수 있습니다.

## 3. 문제/요구사항
- Nginx 메트릭 수집 엔드포인트 필요
- Exporter와 Prometheus scrape 설정 필요
- Grafana 데이터소스 연결과 대시보드 구성이 필요

## 4. 판단 및 선택 이유
- `stub_status`는 Nginx 상태를 간단히 노출할 수 있음
- Exporter를 두면 Prometheus 친화적인 메트릭으로 전환 가능
- 운영 관점에서는 로그 분석보다 시계열 대시보드가 추세 파악에 유리함

## 5. 적용 절차
1. Nginx 설치 및 `stub_status` 활성화
2. Exporter 실행
3. Prometheus `scrape_configs` 추가
4. Targets 상태 확인
5. Grafana 데이터소스에 Prometheus 연결
6. Nginx 관련 메트릭으로 대시보드 구성

## 6. 검증 방법
- `/metrics` 응답 확인
- Prometheus Targets 상태 확인
- Grafana 쿼리 결과 확인

