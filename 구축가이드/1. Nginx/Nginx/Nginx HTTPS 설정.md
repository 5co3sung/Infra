# Nginx HTTPS 설정

### 개요

이 문서는 Nginx의 HTTPS 적용, 인증서 구성, 업스트림 프록시 설정을 표준 절차로 정리합니다.

---

### 1) HTTP → HTTPS 리다이렉트 및 기본 서버 블록

1. conf 경로 확인 후 사이트 설정 파일 생성
- 경로 예시: `/etc/nginx/conf.d/your-site.conf`
1. HTTP 요청을 HTTPS로 리다이렉트

```
server {
    listen 80;
    server_name [example.com](http://example.com) [www.example.com](http://www.example.com);  # 서비스 도메인

    # 모든 트래픽을 HTTPS로 이동
    return 301 [https://$host$request_uri](https://$host$request_uri);
}
```

1. HTTPS 서버 블록 기본 예시

```
server {
    listen 443 ssl http2;
    server_name [example.com](http://example.com);

    # 인증서 경로
    ssl_certificate     /etc/nginx/cert/fullchain.pem;   # 서버 인증서 + 체인 포함
    ssl_certificate_key /etc/nginx/cert/key.key;     # 개인키

    # 권장 SSL 설정(요약)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 접근 로그(필요 시 경로 조정)
    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    # 애플리케이션 프록시 위치 예시는 아래 3) 참조
}
```

> 배포 전 `nginx -t`로 문법 검증 후 `systemctl reload nginx`로 무중단 반영하세요.
> 

---

### 2) 인증서 적용 표준 절차

1. 인증서 파일 준비
- 일반적으로 다음 파일을 받습니다.
    - 도메인 인증서: `cert.crt` 또는 `certificate.pem`
    - 체인 인증서: `chain.crt` 또는 `ca-bundle.pem`
    - 개인키: `privkey.key`
1. 체인 포함 풀체인 파일 생성

```bash
# 순서: 도메인 인증서 → 체인 인증서(중간CA → 루트 순)
cat cert.crt chain.crt > fullchain.pem
# 발급사 번들을 그대로 사용할 경우 fullchain.pem만 배치
```

1. 개인키 패스워드 제거(키에 암호가 걸린 경우만)

```bash
openssl rsa -in privkey.key -out privkey.key
# 프롬프트에 기존 패스프레이즈 입력 → 동일 경로 파일로 무암호 키 저장
```

1. 파일 배치 권장 경로 및 권한

```bash
mkdir -p /etc/nginx/cert/
cp fullchain.pem /etc/nginx/cert/
cp privkey.key  /etc/nginx/cert/
chmod 600 /etc/nginx/cert/privkey.key
chown root:root /etc/nginx/cert/privkey.key
```

1. 서버 블록에 적용 후 테스트 및 재시작

```bash
nginx -t && systemctl reload nginx
```

> 아파치 인증서 묶음을 전환할 때는 각 PEM의 끝부분 서명 블록을 비교해 동일 체인 여부를 확인할 수 있습니다.
> 

---

### 3) 업스트림과 프록시 설정 예시

1. 업스트림 그룹 정의

```
upstream qapi {
    server 172.16.0.7:8082;   # 애플리케이션 서버 #1
    server 172.16.0.17:8082;  # 애플리케이션 서버 #2
    # 필요 시 max_fails, fail_timeout 등 옵션 추가
}
```

1. 프록시 위치 블록 예시(HTTP/1.1, 헤더 전달)

```
location /qapi/ {
    proxy_http_version 1.1;
    proxy_set_header Connection "";                 # keep-alive 유지
    proxy_pass_request_headers on;                  # 원본 요청 헤더 유지

    # 원본 클라이언트 정보 전달
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Port $server_port;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $http_host;

    proxy_pass [http://qapi](http://qapi);   # 위에서 정의한 업스트림 사용
}
```

1. HTTPS 서버 블록에 업스트림 프록시 결합 예시

```
server {
    listen 443 ssl http2;
    server_name [example.com](http://example.com);

    ssl_certificate     /etc/nginx/cert/fullchain.pem;
    ssl_certificate_key /etc/nginx/cert/privkey.key;

    # include /etc/nginx/snippets/ssl-params.conf;  # 조직 공통 SSL 파라미터가 있다면 분리 관리

    location /qapi/ {
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_pass [http://qapi](http://qapi);
    }
}
```

---

### 4) 운영 팁

- 변경 전 `nginx -t`로 문법 확인 → `systemctl reload nginx`로 무중단 반영
- 방화벽 포트 오픈 확인: 80, 443
- SELinux 사용 시 httpd_can_network_connect 등 필요 정책 설정을 검토
- 공통 설정은 `snippets` 또는 `conf.d/*.conf`로 분리해 재사용

---

### 관련 문서

- [Nginx](../Nginx%2023fae5fab89d8084ba2cc58d7268d559.md)
- 고가용성 VIP 구성: [Keep-alived 설치](Keep-alived%20%EC%84%A4%EC%B9%98%2023fae5fab89d8009a34afe843a0dff5e.md)