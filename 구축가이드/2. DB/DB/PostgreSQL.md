# PostgreSQL

<aside>
🧭

PostgreSQL 설치 및 초기 설정 가이드입니다. 원문 내용을 유지하면서 가독성과 구조를 개선했습니다. RPM 설치와 소스(tar.gz) 설치 흐름을 모두 포함합니다.

</aside>

---

## 1) 개요

- 대상: Rocky Linux 8 계열 기준
- 범위: 설치(RPM, 소스) → 사용자/그룹 및 환경 변수 → 디렉터리 구성 → 컴파일/설치 → 설정(postgresql.conf, pg_hba.conf) → 기동 및 접속 → 부록(패키지 목록, 에러 대응, 링크)
- 출처 표기: 클라우드운영팀 신상원 작성

---

## 2) PostgreSQL 설치 (RPM)

1. 저장소 및 모듈 정리

```bash
yum install [https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm`](https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm`)
sudo dnf -qy module disable postgresql
```

1. 패키지 설치

```bash
yum -y install postgresql15-server postgresql15-contrib
```

1. 초기화 및 서비스 기동

```bash
cd /usr/pgsql-15/bin/
postgresql-15-setup initdb
vi /var/lib/pgsql/15/data/postgresql.conf   # listen_addresses = '*'
systemctl start postgresql-15               # systemctl로 기동/중지 가능
```

1. 접속

```bash
su - postgres
psql
```

---

## 3) PostgreSQL 설치 (소스 tar.gz)

사내 표준 또는 네트워크 제약 환경에서 소스 설치가 필요한 경우 아래를 참고합니다.

### 3.1 연관 패키지 설치

인터넷 미 연결 환경에서는 downloadonly 옵션으로 미리 받아서 옮겨 설치합니다.

```bash
yum install -y glibc make gcc gcc-c++ readline readline-devel
# 설치 예: readline-devel x86_64 7.0-10.el8
#          ncurses-c++-libs x86_64 6.1-9.20180224.el8_8.1
#          ncurses-devel x86_64 6.1-9.20180224.el8_8.1

yum -y install zlib zlib-devel autoconf openssl openssl-devel uuid gettext gettext-devel
# 설치 예: uuid x86_64 1.6.2-43.el8

yum -y install libuuid libuuid-devel
# 설치 예: libuuid-devel x86_64 2.32.1-42.el8_8
```

라이브러리 경로 확인 및 환경 변수 반영:

```bash
find / -name [libuuid.so](http://libuuid.so) 2>/dev/null
# 예: /usr/lib64/[libuuid.so](http://libuuid.so)
export LD_LIBRARY_PATH=/usr/lib64/[libuuid.so](http://libuuid.so)
```

### 3.2 사용자/그룹 생성 및 환경 변수

```bash
groupadd postgres
useradd -g postgres -d /postgres postgres
su - postgres

# ~/.bash_profile
export PG_HOME=/postgres/pgsql
export PATH=$PG_HOME/bin:$PATH
export PGLIB=$PG_HOME/lib
export PGDATA=$PG_HOME/data
export MANPATH=$MANPATH:$PG_HOME/man
export LD_LIBRARY_PATH=/postgres/pgsql/lib
```

### 3.3 디렉터리 생성

```bash
# root
mkdir /pg_data /pg_log
chown postgres:postgres /pg_data /pg_log

# postgres
mkdir -p /postgres/pgsql /postgres/pgsql/data /postgres/pgsql/man
```

### 3.4 컴파일 및 설치

```bash
# tar.gz를 /postgres 하위로 옮긴 후 진행
tar -xf postgresql-15.1.tar.gz
cd postgresql-15.1
./configure --prefix=/postgres/pgsql --enable-depend --enable-nls=utf-8 --with-openssl
make world
make install-docs
make install-world

cd $PG_HOME/bin
./initdb -E utf-8 -D /postgres/pgsql/data   # character-set을 utf-8로 설정
```

---

## 4) 설정 변경

### 4.1 postgresql.conf (예시)

```
# Connection Settings
listen_addresses = '*'       # 모든 IP 수신
port = 5432
max_connections = 100

# Logging
log_destination    = 'stderr'        # csvlog/jsonlog 사용 시 logging_collector 필요
logging_collector  = on
log_directory      = '/pg_log'
log_filename       = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age   = 30d
log_rotation_size  = 10MB
```

### 4.2 pg_hba.conf (예시)

보안 요구사항에 맞게 최소 허용 원칙으로 조정하세요.

```
# local
local   all   all                                   trust
# IPv4
host    all   all                                   127.0.0.1/32       trust
host    all   all                                   0.0.0.0/0          trust
host    all   all                                   all                 trust
```

---

## 5) 기동과 접속

```bash
pg_ctl start -D /postgres/pgsql/data   # -D는 데이터 디렉터리 지정
psql -d postgres                       # 기본 DB는 postgres, -d는 DB 지정
```

---

## 6) 부록

### 6.1 RPM 패키지 링크 및 목록

- 다운로드 경로: [https://download.postgresql.org/pub/repos/yum/15/redhat/rhel-7-x86_64/](https://download.postgresql.org/pub/repos/yum/15/redhat/rhel-7-x86_64/)
    
    ※ 설치 OS, 버전에 따라 경로 상이
    
- (Rocky 8 기준) 필요 패키지 예시
    - postgresql15-server-15.8-1PGDG.rhel8.x86_64
    - postgresql15-libs-15.8-1PGDG.rhel8.x86_64
    - postgresql15-contrib-15.8-1PGDG.rhel8.x86_64
    - postgresql15-15.8-1PGDG.rhel8.x86_64

### 6.2 에러 대응

- psql 실행 에러: [`libpq.so](http://libpq.so).5: cannot open shared object file`
    - 조치: `yum install -y [libpq.so](http://libpq.so).5` (Rocky 8에서 확인)

### 6.3 릴리스 노트

- 0.1v 최초작성

---

## 7) 관련 문서

- 
    
    [1.1 PG-AUTO-FAILOVER](PostgreSQL/1%201%20PG-AUTO-FAILOVER%2026fae5fab89d808495e7d689c3d01952.md)
    
- 
    
    [1.2 PostgreSQL 이중화 (Archive-backup)](PostgreSQL/1%202%20PostgreSQL%20%EC%9D%B4%EC%A4%91%ED%99%94%20(Archive-backup)%2026fae5fab89d80e082abe1b5cd9b6ad8.md)