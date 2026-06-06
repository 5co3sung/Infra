# ArgoCD 구축

## ArgoCD 설치/구축 절차

### 1) ArgoCD Helm Repository 추가

- PowerShell 또는 VS Code 터미널에서 실행
- Windows에 kubeconfig를 설치해 둔 상태 기준 (환경별로 kubeconfig 파일명/위치는 다를 수 있음)

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### 2) 네임스페이스 생성

형식

```bash
kubectl --kubeconfig <kubeconfig.yaml> create ns argocd
```

예시

```bash
kubectl --kubeconfig dev_kubeconfig.yaml create ns argocd
```

### 3) Helm으로 ArgoCD 설치

설치

```bash
helm fetch argo/argo-cd
helm install argocd argo/argo-cd --namespace argocd --kubeconfig dev_kubeconfig.yaml
```

삭제

```bash
helm uninstall --kubeconfig .\dev_kubeconfig.yaml argocd -n argocd
```

조회

```bash
helm list --kubeconfig .\dev_kubeconfig.yaml -n argocd
```

서비스 확인(예: 설정 적용 확인)

```bash
kubectl --kubeconfig .\dev_kubeconfig.yaml -n argocd get svc
```

### 4) argocd-server 서비스 타입 변경 (ClusterIP → LoadBalancer)

```bash
kubectl --kubeconfig dev_kubeconfig.yaml edit svc argocd-server -n argocd
```

편집기(메모장 등)가 열리면 `type: ClusterIP`를 `type: LoadBalancer`로 변경 후 저장

![image.png](ArgoCD%20%EA%B5%AC%EC%B6%95/image.png)

### 5) 접속 확인

서비스/외부 IP 확인

```bash
kubectl --kubeconfig dev_kubeconfig.yaml get svc -n argocd
```

확인된 EXTERNAL-IP(또는 할당된 IP/도메인)로 브라우저 접속

![image.png](ArgoCD%20%EA%B5%AC%EC%B6%95/image%201.png)

밑줄친 내용을 브라우저에 입력하여 접속해본다.

![image.png](ArgoCD%20%EA%B5%AC%EC%B6%95/image%202.png)

### 6) 초기 로그인(admin) 비밀번호 확인

- 초기 계정: `admin`
- 초기 비밀번호: `argocd-initial-admin-secret`에서 조회

Windows PowerShell에서 확인

```powershell
$secret = kubectl --kubeconfig dev_kubeconfig.yaml -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($secret))
```

### 7) 특정 버전(Chart Version)으로 설치

설치 가능한 버전 조회

```bash
helm search repo argo/argo-cd --versions
```

원하는 `CHART VERSION`을 지정해 설치

```bash
helm install argocd argo/argo-cd --namespace argocd --kubeconfig dev_kubeconfig.yaml --version 7.6.12
```

## (추가) Private Registry 이미지 Pull 설정 (Argo CD용)

Argo CD가 사설 레지스트리(예: NCP Container Registry 등)에서 이미지를 Pull하려면, 대상 네임스페이스에 레지스트리 인증정보가 들어있는 `imagePullSecret`이 필요합니다.

### 1) Docker Registry Secret 생성

- Secret 이름 예시: `docker-image-registry`
- 네임스페이스 예시: `default` (Argo CD가 배포할 워크로드가 있는 네임스페이스로 맞춰야 함)

```bash
kubectl --kubeconfig /root/.kube/kubeconfig.yaml create secret docker-registry docker-image-registry \
  -n default \
  --docker-server=<docker-image-registry-주소> \
  --docker-username=<USERNAME> \
  --docker-password=<PASSWORD>
```

### 2) 생성 확인

```bash
kubectl --kubeconfig /root/.kube/kubeconfig.yaml get secret docker-image-registry -n default
```

### 3) (중요) 워크로드에 imagePullSecrets 적용

배포되는 Deployment/StatefulSet/DaemonSet 등의 `spec.template.spec.imagePullSecrets`에 위 Secret을 지정해야 합니다.

```yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: docker-image-registry
```

(Helm 차트라면 values.yaml에서 `imagePullSecrets` 옵션을 제공하는 경우가 많습니다.)

[ArgoCD 운영](ArgoCD%20%EA%B5%AC%EC%B6%95/ArgoCD%20%EC%9A%B4%EC%98%81%20323ae5fab89d80939a73c8517c60eecd.md)