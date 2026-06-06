# Istio 구축

## Istio 구축 정리

### 1) Istio(istiod) 설치

- Istio 컨트롤 플레인(istiod)을 `istio-system` 네임스페이스에 Helm으로 설치합니다.

명령어

```bash
helm install istio istio/istiod --kubeconfig .\dev_kubeconfig.yaml -n istio-system
```

### 2) Istio 정상 동작 확인

- `istio-system` 네임스페이스의 파드 상태를 확인해서 정상 여부를 점검합니다.

명령어

```bash
kubectl get pods -n istio-system --kubeconfig .\dev_kubeconfig.yaml
```

### 3) Ingress Gateway 설치

- 외부 트래픽을 Istio로 받아들이기 위한 Gateway 컴포넌트를 `istio-system`에 설치합니다.

명령어

```bash
helm install istio-ingress istio/gateway --namespace istio-system --kubeconfig .\dev_kubeconfig.yaml
```

### 4) Envoy 사이드카(Proxy) 자동 주입 설정

- 네임스페이스에 라벨을 추가하면, 해당 네임스페이스에 생성되는 Pod에는 Envoy 사이드카가 자동으로 주입됩니다.
- 아래는 `default` 네임스페이스에 자동 주입을 활성화하는 예시입니다.

명령어

```bash
kubectl label namespace default istio-injection=enabled --overwrite --kubeconfig .\dev_kubeconfig.yaml
```

명령어 형식(템플릿)

```bash
kubectl label namespace <적용할 네임스페이스> istio-injection=enabled --overwrite --kubeconfig <kubeconfig파일위치>
```

### 5) (참고) Ingress 라우팅으로 LoadBalancer 비용 최소화

요약

- 서비스마다 `type: LoadBalancer`를 만들면 로드밸런서가 여러 개 생겨 비용이 늘 수 있으니, 가능한 한 **Ingress(또는 Istio Gateway)** 로 라우팅을 모아 관리하는 방향을 검토합니다.
- 핵심 아이디어: 외부 노출 지점(로드밸런서)은 최소화하고, 내부에서는 Host/Path 기반 라우팅으로 서비스들을 분기합니다.

### 6) Argo CD Sync 옵션: 특정 리소스 차이 무시(ignoreDifferences)

요약

- 리소스의 일부 필드가 자동으로 바뀌거나(웹훅/컨트롤러가 패치) 환경별로 값이 달라져서 **불필요하게 OutOfSync** 가 나는 경우, 해당 필드 차이를 무시하도록 설정할 수 있습니다.

설정 파일(적용 위치)

- `Applications/values-dev.yaml` (환경별 values 파일)

설정 예시

```yaml
ignoreDifferences:
  - group: admissionregistration.k8s.io
    kind: ValidatingWebhookConfiguration
    name: istio-validator-istio-system
    jsonPointers:
      - /webhooks/0/failurePolicy
```

설명

- 대상 리소스: `ValidatingWebhookConfiguration` 중 `istio-validator-istio-system`
- 무시할 필드: `/webhooks/0/failurePolicy`
- 목적: “싱크가 안 맞을 때(자동 변경되는 필드 때문에) 강제로 무시”해서 운영 잡음을 줄임
- 주의: 무시 범위를 넓히면 실제 변경을 놓칠 수 있으니 **필요 최소 범위(jsonPointers)** 로만 설정