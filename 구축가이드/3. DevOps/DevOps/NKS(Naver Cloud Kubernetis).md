# NKS(Naver Cloud Kubernetis)

---

## 1. Chocolatey(Choco) 설치 준비

- VS Code 또는 PowerShell을 관리자 권한으로 실행합니다.

실행 정책 확인 및 설정:

```powershell
Get-ExecutionPolicy
# Restricted가 아니라면(스크립트 실행 허용 상태가 아니면) 다음 실행
Set-ExecutionPolicy AllSigned
# 확인 프롬프트에서 Y 입력
```

Choco 설치 스크립트 실행:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; \
[[System.Net](http://System.Net).ServicePointManager]::SecurityProtocol = \
[[System.Net](http://System.Net).ServicePointManager]::SecurityProtocol -bor 3072; \
iex ((New-Object [System.Net](http://System.Net).WebClient).DownloadString('[https://community.chocolatey.org/install.ps1](https://community.chocolatey.org/install.ps1)'))
```

설치 후 Choco 명령이 정상 동작하는지 버전 확인을 진행합니다.

---

## 2. Helm과 kubectl 설치(Choco)

Helm 설치:

```powershell
choco install kubernetes-helm
```

헬프 옵션이 출력되면 정상입니다.

kubectl 설치 및 버전 확인:

```powershell
choco install kubernetes-cli
kubectl version --client
# 예) Client Version: v1.33.1  (클러스터와 마이너 버전만 맞으면 보통 문제 없음)
#     Kustomize Version: v5.6.0
```

---

## 3. kubeconfig 디렉터리 준비

작업할 위치로 이동 후 .kube 디렉터리와 설정 파일을 만듭니다.

```powershell
cd <원하는_디렉터리>
mkdir .kube
cd .kube
New-Item config -Type File
```

---

## 4. NCP용 IAM Authenticator 다운로드 및 검증

NCP에 Kubernetes가 있으므로, NCP용 가이드를 참고하여 인증 도구를 준비합니다.

바이너리 다운로드:

```powershell
Invoke-WebRequest -Uri "[https://github.com/NaverCloudPlatform/ncp-iam-authenticator/releases/latest/download/ncp-iam-authenticator_windows_amd64.exe](https://github.com/NaverCloudPlatform/ncp-iam-authenticator/releases/latest/download/ncp-iam-authenticator_windows_amd64.exe)" -OutFile "ncp-iam-authenticator.exe"
```

해시 파일 다운로드:

```powershell
Invoke-WebRequest -Uri "[https://github.com/NaverCloudPlatform/ncp-iam-authenticator/releases/latest/download/ncp-iam-authenticator_SHA256SUMS](https://github.com/NaverCloudPlatform/ncp-iam-authenticator/releases/latest/download/ncp-iam-authenticator_SHA256SUMS)" -OutFile "ncp-iam-authenticator.sha256"
```

SHA256 검증(두 결과가 동일해야 합니다):

```powershell
Get-FileHash ncp-iam-authenticator.exe -Algorithm SHA256
# 출력 예) BAE98639AE5B71943EC033C1C8EEC7EF6656FA3651A820673DCDC055848755F5
```

---

## 5. 실행 경로 구성

C:에 bin 디렉터리를 만들고 바이너리를 배치합니다.

```powershell
New-Item -ItemType Directory -Path C:\bin -Force
Copy-Item .\ncp-iam-authenticator.exe C:\bin\
```

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image.png)

환경 변수(PATHEXT/PATH) 등록이 필요하면 시스템 환경 변수에 C:bin 경로를 추가하고, 새 PowerShell 세션에서 인식되는지 확인합니다.

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image%201.png)

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image%202.png)

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image%203.png)

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image%204.png)

---

### 1️⃣ `.ncloud` 디렉토리 생성

1. `C:\Users\<사용자명>\` 경로로 이동
2. `.ncloud` 디렉토리 생성

```powershell
cd C:\Users\<사용자명>\
mkdir .ncloud

```

---

### 2️⃣ `configure` 파일 생성

`.ncloud` 안에 `configure` 파일 생성 후 내용 작성:

```
[DEFAULT]
ncloud_access_key_id = ACCESSKEYACCESSKEYAC
ncloud_secret_access_key = SECRETKEYSECRETKEYSECRETKEYSECRETKEYSECR
ncloud_api_url = https://ncloud.apigw.ntruss.com

[project]
ncloud_access_key_id = ACCESSKEYACCESSKEYAC
ncloud_secret_access_key = SECRETKEYSECRETKEYSECRETKEYSECRETKEYSECR
ncloud_api_url = https://ncloud.apigw.ntruss.com

```

- `ncloud_access_key_id` 와 `ncloud_secret_access_key`는 본인 계정의 키로 변경
- 저장 후 종료

---

### 3️⃣ kubeconfig 생성

1. `ncp-iam-authenticator`가 있는 경로로 이동

```powershell
cd <ncp-iam-authenticator 설치 경로>

```

1. 아래 명령어로 kubeconfig 생성

```powershell
./ncp-iam-authenticator create-kubeconfig --region KR --clusterUuid <클러스터 UUID> --output C:\dev\.kube\kubeconfig.yaml

```

- 옵션 설명:
    - `-region`: 클러스터가 위치한 지역 (예: KR)
    - `-clusterUuid`: 콘솔에서 확인 가능한 클러스터 UUID
    - `-output`: 생성할 kubeconfig 파일 경로
1. 이후 `kubectl` 명령어에서 `-kubeconfig` 옵션으로 해당 파일을 지정하여 사용 가능

```powershell
kubectl get nodes --kubeconfig C:\dev\.kube\kubeconfig.yaml

```

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image%205.png)

### 4️⃣ ncp-iam-authenticator 경로 문제 해결

- 에러창에서 `ncp-iam-authenticator`를 찾지 못하는 경우:
    1. 생성된 `kubeconfig.yaml` 파일을 열고
    2. `ncp-iam-authenticator` 실행 경로를 절대 경로로 변경
    3. 파일 저장 후 다시 시도

예:

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image%206.png)

---

### 5️⃣ kubectl 사용 예시

```powershell
kubectl get nodes --kubeconfig C:\dev\.kube\kubeconfig.yaml
```

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image%207.png)

- 위 명령어로 클러스터 노드 상태 확인 가능

## **2. 아르고시디(ArgoCD) 설치**

### **1. Helm 레포지토리 추가**

```powershell
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

---

### **2. 네임스페이스 생성**

```bash
kubectl --kubeconfig <kubeconfig.yaml 위치> create ns argocd
# 예시
kubectl --kubeconfig dev_kubeconfig.yaml create ns argocd
```

---

### **3. Helm으로 ArgoCD 설치**

```bash
helm fetch argo/argo-cd
helm install argocd argo/argo-cd --namespace argocd --kubeconfig <kubeconfig.yaml>
```

---

### **4. 관리 명령어**

- **삭제:**

```bash
helm uninstall --kubeconfig .\dev_kubeconfig.yaml argocd -n argocd
```

- **조회:**

```bash
helm list --kubeconfig .\dev_kubeconfig.yaml -n argocd
```

- **서비스 조회/설정 reload:**

```bash
kubectl --kubeconfig .\dev_kubeconfig.yaml -n argocd get svc
```

---

### **5. ArgoCD 서비스 타입 변경**

```bash
kubectl --kubeconfig dev_kubeconfig.yaml edit svc argocd-server -n argocd
# ClusterIP → LoadBalancer 로 변경 후 저장

```

![image.png](NKS(Naver%20Cloud%20Kubernetis)/image%208.png)

[ArgoCD 구축](NKS(Naver%20Cloud%20Kubernetis)/ArgoCD%20%EA%B5%AC%EC%B6%95%20323ae5fab89d804f9079d02689a25632.md)

[Istio 구축](NKS(Naver%20Cloud%20Kubernetis)/Istio%20%EA%B5%AC%EC%B6%95%20323ae5fab89d809e98adf17323a30f8f.md)

[Ingress Nginx](NKS(Naver%20Cloud%20Kubernetis)/Ingress%20Nginx%20324ae5fab89d80fda719d8ec66dbfbe3.md)

[Pod에 NAS 마운트 하는 방법](NKS(Naver%20Cloud%20Kubernetis)/Pod%EC%97%90%20NAS%20%EB%A7%88%EC%9A%B4%ED%8A%B8%20%ED%95%98%EB%8A%94%20%EB%B0%A9%EB%B2%95%20324ae5fab89d80e0a2ecf6267fa7009a.md)

[쿠버네티스 노드풀을 나누어서 구축 하는 경우](NKS(Naver%20Cloud%20Kubernetis)/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%20%EB%85%B8%EB%93%9C%ED%92%80%EC%9D%84%20%EB%82%98%EB%88%84%EC%96%B4%EC%84%9C%20%EA%B5%AC%EC%B6%95%20%ED%95%98%EB%8A%94%20%EA%B2%BD%EC%9A%B0%20324ae5fab89d80998978d3fc739e4ae2.md)