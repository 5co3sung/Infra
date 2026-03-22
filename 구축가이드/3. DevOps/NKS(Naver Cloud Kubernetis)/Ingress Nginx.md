# Ingress Nginx

## 목적

- ingress-nginx에 기본 SSL 인증서(디폴트 인증서) 적용 및 갱신 방법 정리
- (추가) 앱별 Ingress 설정/어노테이션 예시
- (추가) Istio Ingress Gateway 생성 시 값 예시

---

## 1) ingress-nginx 기본 SSL 인증서 적용

### 1-1. 인증서 파일 준비

- `cert.crt`: 인증서 체인(중간 인증서 포함)까지 “전부 합쳐진” CRT
- `key.key`: 개인키

(두 파일을 `kubectl` 명령을 실행하는 PC/서버의 작업 경로에 둡니다.)

### 1-2. Secret 생성 (TLS)

```bash
kubectl --kubeconfig .\dev_kubeconfig.yaml create secret tls lemonhccom-2025 \
  --cert .\cert.crt \
  --key  .\key.key \
  -n istio-system
```

### 1-3. Secret 생성 확인

```bash
kubectl --kubeconfig .\dev_kubeconfig.yaml get secret -n istio-system
```

### 1-4. ingress-nginx values-dev.yaml 설정 (nginx.conf 관련 설정)

`ingress-nginx values-dev.yaml`에서 `controller.config`를 수정합니다.

```yaml
controller:
  config:
    server-tokens: "false"
    ssl-ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA"
    ssl-protocols: "TLSv1.2 TLSv1.3"
    use-forwarded-headers: "true"
    use-proxy-protocol: "true"
```

### 1-5. ingress-nginx values-dev.yaml 설정 (기본 인증서 지정)

`controller.extraArgs.default-ssl-certificate`에 `네임스페이스/Secret이름` 형식으로 지정합니다.

```yaml
controller:
  extraArgs:
    default-ssl-certificate: istio-system/lemonhccom-2025
```

---

## 2) 인증서 갱신(교체) 방법

아래만 수행하면 됩니다.

- 1-2 (Secret 재생성 또는 새 Secret 생성)
- 1-5 (Secret 이름이 바뀌면 `default-ssl-certificate` 값도 변경)

---

## 3) 도메인/Ingress 로케이션 처리(앱별 설정 예시)

각 애플리케이션의 `values-dev.yaml`에서 Ingress를 활성화하고, 필요한 어노테이션/옵션을 추가합니다.

### 3-1. Ingress 활성화 예시

```yaml
server:
  ingress:
    enabled: true
```

### 3-2. annotations 예시 (nginx ingress)

```yaml
annotations:
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
  nginx.ingress.kubernetes.io/whitelist-source-range: "58.225.27.198/32"
  nginx.ingress.kubernetes.io/proxy-body-size: "200m"
  nginx.ingress.kubernetes.io/proxy-connect-timeout: "300"
  nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
  nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
  # nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
  # nginx.ingress.kubernetes.io/ssl-passthrough: "true"
```

### 3-3. ingressClassName / hostname / path 예시

```yaml
ingressClassName: "nginx"
hostname: "talktalk-mgr-dev.lemonhc.com"
path: /argocd
```

### 3-4. (옵션) Argo CD 관련 예시

```yaml
tls: true
server:
  insecure: true
  basehref: /argocd
  rootpath: /argocd
```

---

## 4) Istio Ingress Gateway 생성 방법 (values 예시)

### 4-1. ingress-gateway.values-dev.yaml (Service annotations 예시)

```yaml
service:
  annotations:
    # service.beta.kubernetes.io/ncloud-load-balancer-backend-protocol: "tcp"
    # service.beta.kubernetes.io/ncloud-load-balancer-proxy-protocol: "true"
    service.beta.kubernetes.io/ncloud-load-balancer-idle-timeout: "60"
    service.beta.kubernetes.io/ncloud-load-balancer-subnet-id: "LB서브넷 ID"
    service.beta.kubernetes.io/ncloud-load-balancer-description: "로드밸런서 설명"
    service.beta.kubernetes.io/ncloud-load-balancer-size: "small"
```

### 4-2. gateway values-dev.yaml (hosts/tls 예시)

```yaml
hosts:
  - "도메인"
tls:
  mode: SIMPLE
  credentialName: "인증서 이름"
  minProtocolVersion: TLSV1_2
  maxProtocolVersion: TLSV1_3
```