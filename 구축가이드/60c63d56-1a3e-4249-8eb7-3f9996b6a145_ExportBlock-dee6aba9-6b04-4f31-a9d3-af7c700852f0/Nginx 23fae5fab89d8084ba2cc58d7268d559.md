# Nginx

- YUM 리포지토리를 등록하여 설치
- RPM 패키지로 수동 설치

---

### 1. 리포지토리 등록 후 설치

1) 리포 관리 도구 설치

```bash
yum -y install yum-utils
```

2) Nginx 리포지토리 파일 생성 및 내용 입력

```bash
vi /etc/yum.repos.d/nginx.repo
```

아래 내용을 그대로 붙여넣고 저장합니다.

```
[nginx-stable]
name=nginx stable repo
baseurl=[http://nginx.org/packages/centos/$releasever/$basearch/](http://nginx.org/packages/centos/$releasever/$basearch/)
gpgcheck=1
enabled=1
gpgkey=[https://nginx.org/keys/nginx_signing.key<br>module_hotfixes=true<br><br>](https://nginx.org/keys/nginx_signing.key<br>module_hotfixes=true<br><br>)

[nginx-mainline]
name=nginx mainline repo
baseurl=[http://nginx.org/packages/mainline/centos/$releasever/$basearch/](http://nginx.org/packages/mainline/centos/$releasever/$basearch/)
gpgcheck=1
enabled=0
gpgkey=[https://nginx.org/keys/nginx_signing.key<br><br>module_hotfixes=true</td>](https://nginx.org/keys/nginx_signing.key<br><br>module_hotfixes=true</td>)
```

3) 메인라인 리포지토리 활성화(선택)

```bash
yum-config-manager --enable nginx-mainline
```

- 환경에 따라 리포지토리 이름이 다를 수 있습니다. 리포 목록에서 정확한 이름을 확인하세요.

4) Nginx 설치

```bash
yum install -y nginx
```

5) 서비스 기동 및 상태 확인

```bash
systemctl start nginx
systemctl enable nginx
systemctl status nginx --no-pager
```

---

### 2. RPM으로 설치

1) OS 버전에 맞는 RPM 파일 준비

- 예: `nginx-1.28.0-1.el8.ngx.x86_64.rpm`
    - `el8`은 RHEL/CentOS 8 계열 전용이므로 서버 OS 버전을 먼저 확인하세요.
- 외부 통신 가능 시 `wget`으로 직접 다운로드 가능
    - 다운로드 경로 확인: [nginx 공식 다운로드 디렉터리](https://nginx.org/packages/rhel/8/x86_64/RPMS/)

2) RPM 파일을 서버로 전송 후 설치

```bash
rpm -iUvh nginx-<version>.rpm
```

3) 서비스 기동 및 등록

```bash
systemctl start nginx
systemctl enable nginx
```

> 참고: 패키지 의존성 문제가 있는 경우, 사전에 의존 패키지를 설치하거나 `yum localinstall nginx-<version>.rpm`을 사용할 수 있습니다.
> 

---

### 다음 단계

- 
    
    [Nginx HTTPS 설정](Nginx/Nginx%20HTTPS%20%EC%84%A4%EC%A0%95%20242ae5fab89d804aa47aebd9666cf997.md)
    
- 
    
    [Keep-alived 설치](Nginx/Keep-alived%20%EC%84%A4%EC%B9%98%2023fae5fab89d8009a34afe843a0dff5e.md)