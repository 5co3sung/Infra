# ncp_kubernetes_access_setup

## 1. 개요
Naver Cloud Kubernetes Service를 생성하고, `kubectl`과 `ncp-iam-authenticator`로 클러스터에 접근하는 흐름을 정리한 문서입니다.

## 2. 작업 배경
클라우드 콘솔에서 클러스터를 만든 뒤 실제 운영자가 접근하려면
노드풀 설계, 서브넷 분리, 인증 도구 설치, kubeconfig 생성까지 이어지는 절차가 필요했습니다.

## 3. 문제/요구사항
- 클러스터/노드풀 생성 시 하이퍼바이저, 서브넷, 라벨/테인트 등 초기 설계가 필요
- `kubectl`만 설치해서는 접근이 안 되고, 클라우드 인증도구와 kubeconfig 구성이 필요
- 접근 키 관리 방식까지 함께 고려해야 함

## 4. 판단 및 선택 이유
- Kubernetes 문서는 설치 명령보다 **클러스터 설계 선택 이유**가 더 중요함
- 워크로드를 분리하려면 노드풀/라벨/테인트 개념을 먼저 이해해야 함
- 전용 서브넷 사용은 운영 분리와 보안 측면에서 장점이 큼

## 5. 적용 절차
1. 클라우드 콘솔에서 클러스터 생성
2. 하이퍼바이저, CNI, Subnet, Audit Log, 노드풀 스펙 결정
3. Bastion 또는 관리 서버에 `kubectl` 설치
4. `ncp-iam-authenticator` 설치 및 실행 권한 부여
5. 환경 변수 또는 인증 키 설정
6. `create-kubeconfig`로 kubeconfig 생성
7. `kubectl get nodes` 등으로 접근 확인

## 6. 검증 방법
- kubeconfig 생성 여부
- `ncp-iam-authenticator help`
- `kubectl version`, `kubectl get nodes`
- 노드 라벨/테인트 반영 확인

## 7. 결과 및 운영 효과
- 콘솔 기반 생성과 CLI 접근 절차를 한 문서에서 이어서 볼 수 있음
- 단순 '클러스터 생성'보다 운영자 접근 구조까지 설명할 수 있음

## 8. 운영 체크포인트
- Access Key는 문서에 직접 남기지 않기
- 노드풀별 역할 분리를 초기에 정의
- Public 노출보다 Bastion 경유 접근 정책을 우선 검토

