# Keep-alived 설치

### 개요

Keepalived는 VRRP를 사용해 가상 IP(VIP)를 부여하고 장애 시 자동으로 VIP를 승계하여 L4 없이도 웹 이중화를 구현합니다. 본 문서에서는 Nginx 2대(마스터, 백업) 기준 표준 구성을 정리합니다.

---

### 1) 설치 및 기본 준비

1. 패키지 설치 및 기본 설정 백업

```bash
yum -y install keepalived
[ -f /etc/keepalived/keepalived.conf ] && cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak
```

1. 네트워크 인터페이스와 VIP 정의
- 인터페이스 예시: `enp0s3`
- VIP 예시: `172.16.0.100/24`
- VRID 예시: `151`

> CentOS/Rocky 8+에서는 VIP를 별도 ifcfg에 고정 등록하지 않아도 됩니다. Keepalived가 VIP를 부착합니다.
> 

---

### 2) 헬스체크 스크립트

Nginx 프로세스를 확인하는 간단한 스크립트를 vrrp_script로 사용합니다.

```bash
vrrp_script check_nginx {
    script "pidof nginx"
    interval 2
}
```

---

### 3) 마스터 서버 설정 예시

`/etc/keepalived/keepalived.conf`

```
! Configuration File for keepalived

global_defs {
    router_id nginx   # 백업 서버와 동일하게 식별자 사용 
}

vrrp_script check_nginx {
    script "pidof nginx"
    interval 2
}

vrrp_instance VI_01 {
    state MASTER
    interface enp0s3          #적용 할 네트워크 인터페이스 
    virtual_router_id 151
    priority 110               # 높을수록 우선순위 높음
    advert_int 1

    authentication {
        auth_type AH
        auth_pass secret
    }

    virtual_ipaddress {
        172.16.0.100/24
    }

    track_script {
        check_nginx
    }
}
```

---

### 4) 백업 서버 설정 예시

`/etc/keepalived/keepalived.conf`

```
! Configuration File for keepalived

global_defs {
    router_id nginx
}

vrrp_script check_nginx {
    script "pidof nginx"
    interval 2
}

vrrp_instance VI_01 {
    state BACKUP
    interface enp0s3           #작용 할 네트워크 인터페이스
    virtual_router_id 151
    priority 100               # 마스터보다 낮게
    advert_int 1

    authentication {
        auth_type AH
        auth_pass secret
    }

    virtual_ipaddress {
        172.16.0.100/24
    }

    track_script {
        check_nginx
    }
}
```

> 프리엠션 동작을 제어하려면 `nopreempt` 옵션을 검토하세요. 마스터 복구 후 즉시 VIP를 되찾지 않도록 할 수 있습니다.
> 

---

### 5) 서비스 기동 및 상태 확인

```bash
systemctl enable --now keepalived
systemctl status keepalived --no-pager
ip addr show dev enp0s3 | grep 172.16.0.100   # VIP 부착 확인
```

---

### 6) 방화벽 및 커널 설정

1. 방화벽 예시

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
# VRRP 는 멀티캐스트/프로토콜 112 사용. 환경에 맞춰 허용 필요
firewall-cmd --permanent --add-rich-rule='rule protocol value="vrrp" accept'
firewall-cmd --reload
```

1. 커널 포워딩 및 ARP 관련 튜닝(필요 시)

```bash
# VIP 이동 시 ARP 응답 일관성 확보
sysctl -w net.ipv4.ip_nonlocal_bind=1
sysctl -w net.ipv4.conf.all.arp_ignore=1
sysctl -w net.ipv4.conf.all.arp_announce=2
```

- 영구 적용은 `/etc/sysctl.d/99-keepalived.conf`로 관리 권장

---

### 7) 테스트 절차

1. 마스터 서버에서 Nginx 또는 keepalived 중지

```bash
systemctl stop nginx   # 또는
systemctl stop keepalived
```

1. 백업 서버로 VIP 승계 확인

```bash
ip addr show dev enp0s3 | grep 172.16.0.100
```

1. 클라이언트에서 VIP로 웹 접속 후 새로고침하며 가용성 확인

---

### 트러블슈팅 체크리스트

- 두 서버의 `virtual_router_id`, `auth_pass`, `interface`, `VIP`가 일치하는지 확인
- 우선순위(priority) 값은 마스터가 더 높도록 설정
- VRRP 패킷이 방화벽, 스위치에서 차단되지 않는지 확인
- Nginx 비정상 시 `pidof nginx` 결과가 0이 아니면 VIP가 백업으로 넘어가지 않음. 헬스체크 스크립트 개선 검토

---

### 관련 문서

- [Nginx](../Nginx%2023fae5fab89d8084ba2cc58d7268d559.md)
- [Nginx HTTPS 설정](Nginx%20HTTPS%20%EC%84%A4%EC%A0%95%20242ae5fab89d804aa47aebd9666cf997.md)