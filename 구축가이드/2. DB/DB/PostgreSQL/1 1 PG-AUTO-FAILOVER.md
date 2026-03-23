# 1.1 PG-AUTO-FAILOVER

<설치매뉴얼-PG_AUTO_FAILOVER>

클라우드운영팀 신상원 작성

1. **pg_auto_failover 개요**

## 0. pg_auto_failover 개요

- **pg_auto_failover 패키지 다운로드:**
    
    [https://github.com/hapostgres/pg_auto_failover/releases](https://github.com/hapostgres/pg_auto_failover/releases)
    
- 다운로드한 패키지를 각 서버별로 이동 후 적용.
- 일반 구성: **Monitor 1EA, Primary 1EA, Standby 1EA**
- 세 서버 모두 PostgreSQL이 설치되어 있어야 함.
- 구성 순서: **Monitor → Primary → Standby**

![image01.png](1%201%20PG-AUTO-FAILOVER/image01.png)

1. pg_auto_failover 설치

1. **패키지 다운로드 및 이동**
    
    ```bash
    https://github.com/hapostgres/pg_auto_failover/releases
    # 서버별로 파일 이동 후 /postgres 로 이동
    
    ```
    
2. **압축 해제**
    
    ```bash
    [postgres@localhost pgaf]$ unzip pg_auto_failover-main.zip
    $ cd pg_auto_failover-main/
    [postgres@localhost pg_auto_failover-main]$ make
    [postgres@localhost pg_auto_failover-main]$ make install
    
    ```
    
3. **PostgreSQL 설정 변경**
    
    ```bash
    vi postgresql.conf
    shared_preload_libraries = 'pgautofailover'
    
    ```
    
4. **PostgreSQL 재시작**
    
    ```bash
    $pg_ctl restart -D /postgres/pgsql/data
    
    ```
    
    > rpm/yum으로 설치한 경우 systemctl restart postgresql 명령으로 가능
    > 
5. **확장(Extension) 추가**
    
    ```bash
    $psql
    postgres=# create extension btree_gist;
    postgres=# create extension pgautofailover;
    
    ```
    
6. **주의:**
    
    중지하지 않고 구성 시 오류 발생 가능.
    

---

## 2. pg_auto_failover 구성

### 2.1 Monitor 구성

```bash
[postgres@localhost data]$ pg_autoctl create monitor \
--pgctl /postgres/pgsql/bin/pg_ctl \
--pgdata /postgres/pgsql/data \
--pgport 5432 \
--auth trust \
--ssl-self-signed

$pg_autoctl run --pgdata /postgres/pgsql/data &

```

> --pgctl: pg_ctl 경로--pgdata: 데이터 디렉토리 경로--pgport: 포트 번호--auth: 인증 방식--ssl: self-signed 또는 인증서 경로 지정
> 

```bash
[postgres@localhost ~]$ pg_autoctl show uri

```

![image03.png](1%201%20PG-AUTO-FAILOVER/image03.png)

---

**2.1 pg_auto_failover 구성 – primary 구성**

postgres@localhost data]$ `pg_autoctl create postgres \`

- `-pgctl /postgres/pgsql/bin/pg_ctl \`
- `-pgdata /postgres/pgsql/data \`
- `-hostname xxx.xxx.xxx.117 \`
- `-pgport 5432 \`
- `-auth trust \`
- `-ssl-self-signed \`
- `-monitor`

`postgres://autoctl_node@monitor-ip:5432/pg_auto_failover?sslmode=require \`

- `-name "Active“`

---

### 2.2 Primary 구성

```bash
[postgres@localhost data]$ pg_autoctl create postgres \
--pgctl /postgres/pgsql/bin/pg_ctl \
--pgdata /postgres/pgsql/data \
--hostname xxx.xxx.xxx.117 \
--pgport 5432 \
--auth trust \
--ssl-self-signed \
--monitor postgres://autoctl_node@monitor-ip:5432/pg_auto_failover?sslmode=require \
--name "Active"

$pg_autoctl run --pgdata /postgres/pgsql/data &

```

> 설명:
> 
> - `-monitor` 옵션은 Monitor 서버에서 `pg_autoctl show uri`로 확인한 URI 사용
> - URI 내 `localhost.localdomain`은 Monitor의 IP로 변경해야 함
> - 구성 후 Monitor 서버에서 다음 명령으로 연결 상태 확인
>     
>     ```bash
>     $ pg_autoctl show state
>     ```
>     

![image04.png](1%201%20PG-AUTO-FAILOVER/image04.png)

### 2.3 Standby 구성

```bash
[postgres@localhost pgsql]$ cd /postgres/pgsql/
[postgres@localhost pgsql]$ mv data data_old

[postgres@localhost pgsql]$ pg_autoctl create postgres \
--pgctl /postgres/pgsql/bin/pg_ctl \
--hostname xxx.xxx.xxx.118 \
--pgport 5432 \
--auth trust \
--ssl-self-signed \
--monitor postgres://autoctl_node@monitor-node-IP:5432/pg_auto_failover?sslmode=require \
--name "Standby"

$pg_autoctl run --pgdata /postgres/pgsql/data

```

> 기존 데이터 디렉토리는 백업 또는 삭제Standby는 Primary 데이터 복제 후 사용하므로 --pgdata 옵션 생략 가능구성 완료 후 Monitor 노드에서 pg_autoctl show state로 상태 확인
> 

![image05.png](1%201%20PG-AUTO-FAILOVER/image05.png)

<standby를 구성 시에 데이터 복제중인 것을 확인 할 수 있다.>

▶에러없이 잘 실행되면 아래 명령어로 기동

$`pg_autoctl run --pgdata /postgres/pgsql/data`

---

모니터node에서 pg_autoctl show state명령어로 연결 됬는지 확인

<스탠바이 구성 후 모니터 node에서 조회 했을 때 상태 변화>

![image07.png](1%201%20PG-AUTO-FAILOVER/image07.png)

1. **Switch-over 테스트 방법**
2. Monitor 노드에서 아래 명령어로 Primary와 standby가 바뀌는지 확인

$ pg_autoctl perform switchover

---

<진행 과정>

![image08.png](1%201%20PG-AUTO-FAILOVER/image08.png)

<실행결과>

![image10.png](1%201%20PG-AUTO-FAILOVER/image10.png)

▶pg_autoctl perform switchover 명령어를 실행 후

10:55:56 4138 ERROR Failed to receive monitor's notifications

10:55:56 4138 ERROR Failed to wait until a new primary has been notified

에러가 발생하는데 perform switchover 간 시간이 60초가 넘어서 발생, 모니터 node의 로그를 확인하여 standby가 primary로 전환이 된 것을 확인 후 다시

pg_autoctl show state명령어로 조회하면 상태가 바뀐 것을 확인이 가능하다.

1. **failover 테스트**
2. primary 노드의 서버를 셧다운 한다.

![image11.png](1%201%20PG-AUTO-FAILOVER/image11.png)

1. 시간이 지난 후 standby가 primary를 계승하는지 확인한다.
2. 기존 standby 서버에서 db접속 후 select pg_is_in_recovery(); 조회했을떄 f 나오는지 확인

![image12.png](1%201%20PG-AUTO-FAILOVER/image12.png)

1. create user test; 명령어로 유저 생성도 되는지 확인

![image13.png](1%201%20PG-AUTO-FAILOVER/image13.png)

1. primary서버 재기동 및 pg_auto_failover 재기동

$`pg_autoctl run —pgdata /postgres/pgsql/data`

---

11:48:37 2041 INFO pg_rewind: connected to server

11:48:37 2041 INFO pg_rewind: servers diverged at WAL location 0/3015CF0 on timeline 3

11:48:37 2041 INFO pg_rewind: rewinding from last common checkpoint at 0/3015C40 on timeline 3

11:48:37 2041 INFO pg_rewind: reading source file list

11:48:37 2041 INFO pg_rewind: reading target file list

11:48:37 2041 INFO pg_rewind: reading WAL in target

11:48:37 2041 INFO pg_rewind: need to copy 83 MB (total source directory size is 103 MB)

11:48:37 2041 INFO

0/85703 kB (0%) copied

11:48:37 2041 INFO 85703/85703 kB (100%) copied

11:48:37 2041 INFO pg_rewind:

11:48:37 2041 INFO

11:48:37 2041 INFO creating backup label and updating control file

11:48:37 2041 INFO pg_rewind:

11:48:37 2041 INFO

11:48:37 2041 INFO syncing target data directory

11:49:43 2041 INFO pg_rewind:

11:49:43 2041 INFO

11:49:43 2041 INFO Done!

11:49:43 2041 INFO Creating the standby signal file at "/postgres/pgsql/data/standby.signal", and replication setup at "/postgres/pgsql/data/postgresql-auto-failover-standby.conf"

11:49:43 2041 INFO Contents of "/postgres/pgsql/data/postgresql-auto-failover-standby.conf" have changed, overwriting

11:49:44 2054 INFO

/postgres/pgsql/bin/postgres -D /postgres/pgsql/data -p 5432 -h *

---

▶재기동 시 기존 primary 노드가 master로 승격된 standby노드에서 데이터를 복제중인 것을 확인이 가능하다.

1. 아래 명령어를 monitor노드에서 실행하여 primary, secondary가 바뀐 상태로 되있는지 확인

$`pg_autoctl show state`

---

(서버 종료전 과 다른 역할을 해야함 primary는 standby standby는 primary)

![image14.png](1%201%20PG-AUTO-FAILOVER/image14.png)

▶기존 primary노드에서 복제가 끝난 후 secondary로 동작하는 것을 확인 가능하다.

7.monitor 노드에서 아래 명령어로 복구 진행

`$pg_autoctl perform failover`

![image15.png](1%201%20PG-AUTO-FAILOVER/image15.png)

---

1. **진행 간 오류해결 사항 및 추가내용 정리**

▶기존에 create psotgres 설정을 지우는 방법

- pg_auto_failover 설치 디렉토리에서

make uninstall, make clean all 순서대로 컴파일구성 삭제

후에 OS유저 홈 디렉토리 (문서 기준 /postgres) 아래의 숨겨진 파일 탐색

▶/postgres/.config/pg_autoctl/postgres/pgsql/data --> pg_autoctl.cfg (pg_autoctl 설정파일, 해당 파일이 있다면 삭제)

▶/postgres/.local/share/pg_autoctl/postgres/pgsql/data --> pg_autoctl.state (postmaster.pid 파일과 유사한 것, pg_autoctl 상태 체크에 사용됨) (해당 파일이 남아있다면, pg_auto_failover를 재구성할 때 오류가 발생할 수 있음)

▶기존에 생성된 primary standby의 설정을 지우려면

$pg_autoctl drop node --pgdata $PGDATA --destroy

---

명령어로 standby -> primary 순서대로 삭제

▶postgres 프로세스조회 및 죽이는 방법

[postgres@localhost ~]$ pgrep postgres

---

7839

7841

7842

7843

7846

7847

7848

7849

7850

[postgres@localhost ~]$ pkill –kill postgres

---

[postgres@localhost ~]$ pgrep postgres

---

[postgres@localhost ~]$

---

릴리즈노트

0.1v 최초작성