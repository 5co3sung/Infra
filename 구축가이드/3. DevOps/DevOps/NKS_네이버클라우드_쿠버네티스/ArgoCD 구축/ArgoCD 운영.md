# ArgoCD 운영

## 개요

- Jenkins에서 빌드한 이미지를 Naver Cloud Private Registry(오브젝트 스토리지 기반)에 푸시하고, Argo CD로 배포 대상을 GitOps 방식으로 운영한다.

---

## 1) Jenkins ↔ NCP Private Registry 연동

1. Naver Cloud에서 오브젝트 스토리지 생성
2. Private Registry(리포지토리) 생성
3. Jenkins 서버에서 레지스트리 로그인
- `docker login <사설 레지스트리 주소>`
- 사용자/비밀번호 입력 시:
    - username/password 대신 `access_key` / `secret_key`를 사용
    - 
    
    ![image.png](ArgoCD%20%EC%9A%B4%EC%98%81/image.png)
    

---

## 2) Argo CD CLI 로그인 (Ingress/화이트리스트 확인 포함)

1. values 파일에서 Ingress 접근 제어 확인
- `argocd-values-dev.yaml` 내 `server.ingress.annotations`에서 허용 IP(whitelist) 등록 여부 확인
- 예시

```yaml
annotations:
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
  nginx.ingress.kubernetes.io/whitelist-source-range: "IP1/Prefix, IP2/Prefix, IP3/Prefix"
```

1. CLI 로그인 예시
- `argocd login 아르고시디 도메인 --username admin --password '<admin-password>' --grpc-web --grpc-web-root-path=/argocd`
- 로그인 성공 여부 확인

---

## 3) Argo CD 사용자 계정 추가

1. 계정 추가 (configmap 설정)
- `argocd-values-dev.yaml`의 `configs.cm`에 계정 추가
- 예시
    - `accounts.<사용자ID>: login`
    - `accounts.<사용자ID>.enabled: true`
1. 권한 부여 (RBAC)
- `configs.rbac.policy.csv`에 권한 정책 추가
- 예시(Developer 역할)

```
p, role:developer, applications, get, app/*, allow
p, role:developer, applications, update, app/*, allow
p, role:developer, applications, sync, app/*, allow
p, role:developer, projects, get, app, allow
p, role:developer, repositories, get, default/[헬름차트 리포지토리주소/헬름차트.git], allow
     #################### 계정 role 설정 ####################
      #젠킨스
      g, jenkins, role:admin
      #신상원
      g, sangwonshin, role:admin
      # 계정이름
      G, 사용자 계정, role:어떤권한을 줄것인지 적용
```

1. 사용자 ↔ Role 매핑
- `policy.csv`에서 그룹 매핑 추가
- 예시
    - `g, jenkins, role:admin`
    - `g, sangwonshin, role:admin`
    - `g, <사용자ID>, role:<role-name>`
1. 변경사항 저장 후 Git 배포(Argo CD가 Sync하도록 반영)

---

## 4) 계정 생성/반영 확인

- 계정 리스트 확인:
- 젠킨스 서버에서 argocd login 후  `argocd account list`
- 

![image.png](ArgoCD%20%EC%9A%B4%EC%98%81/image%201.png)

---

## 5) 신규 계정 초기 비밀번호 설정

- 신규 계정 비밀번호 설정 흐름(예시)
    
    ![image.png](ArgoCD%20%EC%9A%B4%EC%98%81/image%202.png)
    
- `Enter password of currently logged in user (admin)`: 현재 로그인한 admin 비밀번호 입력
- `Enter new password for user <user>`: 신규 계정 초기 비밀번호 입력
- `Confirm new password for user <user>`: 비밀번호 재입력