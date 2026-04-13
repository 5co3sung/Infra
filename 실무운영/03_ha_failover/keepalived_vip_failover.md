# keepalived_vip_failover

## 1. 개요
단일 웹 서버 장애 시 VIP를 다른 노드로 넘겨 서비스 연속성을 확보하기 위한 Keepalived 구성을 정리했습니다.

## 2. 작업 배경
고객사의 요청으로 Proxy 서버 상단의 L4스위치가 없는경우 Proxy서버 이중화의 대한 해결방안으로 Keep-alived를 도입하게 되었고,
웹 서버 이중화와 함께 고정 접속 지점(VIP)을 유지할 수 있는 Failover 구성이 필요했습니다.

## 3. 환경
- 2대 이상의 Proxy Svr
- Keepalived + VRRP
- 서비스 프로세스 상태에 따라 우선순위 조정

## 4. 문제/요구사항
- 단일 장애점(SPOF) 제거 필요
- 장애 발생 시 사용자가 접속할 IP를 바꾸지 않도록 구성 필요
- Nginx 프로세스 다운 시 VIP가 다른 노드로 넘어가야 함

## 5. 판단 및 선택 이유
### 고려한 대안
- DNS 전환
- 수동 장애조치
- Keepalived 기반 VIP Failover

### 최종 선택
Keepalived 기반 VRRP 방식으로 VIP Failover를 구성

### 선택 이유
- 전환 속도가 빠름
- 운영자가 직접 개입하지 않아도 자동 전환 가능
- 프로세스 헬스체크와 연계 가능

## 6. 적용 절차
1. `global_defs`, `router_id` 설정
2. `vrrp_script`로 Nginx 프로세스 상태 확인 스크립트 연결
3. `vrrp_instance`에 `interface`, `virtual_router_id`, `priority`, `virtual_ipaddress` 설정
4. 인증 방식과 track_script 연결
5. 양 노드에서 우선순위를 다르게 설정

## 7. 검증 방법
- 서비스 정상 상태에서 VIP 보유 노드 확인
- 주 노드에서 Nginx 중지 후 VIP 전환 여부 확인
- 네트워크 인터페이스와 ARP 갱신 상태 확인
- 재기동 후 원복 정책 확인

## 8. 결과 및 운영 효과
- 프론트 계층 단일 장애점 완화
- 사용자는 동일 VIP로 접속 가능
- 서비스 장애 시 수동 전환 부담 감소

## 9. 운영 체크포인트 / 트러블슈팅
- Nginx 프로세스 체크 스크립트가 실제 장애를 잘 반영하는지 확인
- split-brain 방지를 위해 네트워크/우선순위/인증 설정 검토
- Keepalived는 Nginx 종속이 아니라 HA 공통 기술로 문서 분리하는 것이 적절함

