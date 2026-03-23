# 1-2 MariaDB 이중화

---

## 1. 개요

- 대상: MariaDB Master–Master 비동기 복제 환경
- 목적: 연결 실패 및 복제 이상 발생 시 신속한 원인 파악과 조치, 표준 구축 절차 제공
- 범위: 서버 준비 → 설치 및 설정 → 복제 구성 → 점검 및 모니터링 → 장애 대응 및 복구 → 운영 체크리스트

---

## 2. 구성도 및 흐름

- Master에서 생성된 바이너리 로그를 Slave가 읽어 적용하는 구조
- 주요 포트: 3306(TCP)

![image.png](1-2%20MariaDB%20%EC%9D%B4%EC%A4%91%ED%99%94/image.png)

---

## 3. 사전 준비 사항

- OS 및 패키지 최신화, 시간 동기화(chrony 또는 ntpd) 활성화
- 방화벽 및 보안그룹에서 3306 양방향 허용 여부 확인
- 호스트네임·DNS 또는 /etc/hosts 정합성 확인
- 동일 Major 버전, 유사 Minor 버전 권장. 문자셋과 sql_mode 등 기본 설정 표준화

### 3.1 점검 명령 예시

```bash
# 포트 개방 확인 (서버 간)
nc -vz <상대_IP> 3306 || telnet <상대_IP> 3306

# 시간 동기화 상태
chronyc tracking || timedatectl status
```

---

## 4. MariaDB 설치 및 공통 설정

my.cnf의 핵심 표준 예시입니다. 환경 표준에 맞게 조정하세요.

```
[mysqld]
# 네트워크
port = 3306
# bind-address는 기본적으로 0.0.0.0 또는 NIC IP를 사용하고, 방화벽/보안그룹으로 접근 제어를 권장
# bind-address = 0.0.0.0

# 문자셋 표준
character-set-server = utf8
collation-server     = utf8_general_ci
skip-character-set-client-handshake
init-connect = 'SET NAMES utf8'

# 로그 및 복제 관련
server-id = 1              # Master는 고유, Slave는 서로 다른 값 부여
log_bin   = mysql-bin      # Master에서 필수
binlog_format = ROW        # 일반적으로 ROW 권장
sync_binlog = 1            # 장애 내구성 강화(성능/스토리지 고려하여 조정)
expire_logs_days = 7

# 트랜잭션/엔진
default_storage_engine = InnoDB
innodb_flush_log_at_trx_commit = 1
```

- Master는 server-id 예: 1, Slave는 2,3...과 같이 유일 값 사용
- bind-address는 함부로 변경하지 말고, 접근 제어는 OS 방화벽 또는 보안그룹으로 관리

적용 후 재기동 및 변수 확인

```bash
systemctl restart mariadb
mysql -uroot -p -e "SHOW VARIABLES LIKE 'c%';"
```

---

## 5. 복제 계정 및 권한 설정(Master)

```sql
-- 복제 전용 계정 생성(강력한 비밀번호 사용)
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%' IDENTIFIED BY '비밀번호';
FLUSH PRIVILEGES;

-- 현재 바이너리 로그 위치 확인(파일/포지션)
SHOW MASTER STATUS;  -- File, Position 확인
```

주의: 운영 환경에서는 '%' 대신 Slave의 고정 IP로 제한하는 것을 권장합니다.

---

## 6. 초기 데이터 동기화(선택지)

초기 동기화는 데이터 정합성 확보를 위해 필수입니다. 환경에 맞는 한 가지를 선택하세요.

### 6.1 mysqldump 방식

```bash
mysqldump -uroot -p <복제 할 데이터베이스 명> --single-transaction --routines --events --triggers \
  > /tmp/full_dump.sql

# Slave로 전송 후 적용
mysql -uroot -p < /tmp/full_dump.sql
```

![image.png](1-2%20MariaDB%20%EC%9D%B4%EC%A4%91%ED%99%94/image%201.png)

---

7. Slave 복제 설정

Slave의 my.cnf 예시

```
[mysqld]
server-id = 2
relay_log = relay-bin
read_only = 1       # 운영 환경에서 권장 (SUPER 계정 제외)
```

Master 접속 정보 설정 후 복제 시작

```sql
CHANGE MASTER TO
  MASTER_HOST='[master.example.com](http://master.example.com)',
  MASTER_USER='repl',
  MASTER_PASSWORD='비밀번호',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='mysql-bin.000123',
  MASTER_LOG_POS=456789,
  GET_MASTER_PUBLIC_KEY=1;  -- 10.4+ 클라이언트 인증 이슈 대응

START SLAVE;

-- 상태 확인
SHOW SLAVE STATUS\G;
```

중요 필드

- Slave_IO_Running: Yes, Slave_SQL_Running: Yes가 정상
- Seconds_Behind_Master: 지연 지표. 0 또는 낮게 유지
- Last_IO_Error/Last_SQL_Error: 장애 원인 파악에 핵심

---

## 8. 연결 실패 및 복제 장애 점검 절차(상세)

### 8.1 네트워크·방화벽

- 3306 포트 통신 확인, 중간 장비(L4, FW, 보안그룹) 규칙 확인
- 패킷 드롭이 의심되면 tcpdump로 SYN/ACK 흐름 확인

```bash
tcpdump -i eth0 port 3306 -nn
```

### 8.2 원격 접속 테스트

```bash
mysql -h <Master_IP_or_DNS> -u repl -p -e 'SELECT 1;'
```

- DNS 이슈 시 /etc/hosts로 직접 매핑하여 재시험

### 8.3 계정·권한·플러그인

```sql
SHOW GRANTS FOR 'repl'@'%';
SELECT user,host,plugin FROM mysql.user WHERE user='repl';
```

- 권한 부족 시 GRANT 재수행, 인증 플러그인 호환 확인(caching_sha2_password 등)

### 8.4 CHANGE MASTER 파라미터 재검토

- 호스트, 포트, 로그 파일/포지션 값 정확성 재확인
- Master 측 binlog purge로 파일이 사라진 경우, 재덤프 또는 GTID 사용 고려

### 8.5 슬레이브 스레드 상태

```sql
SHOW SLAVE STATUS\G;
-- 예) IO_Errno 1045(권한), 2003(접속 불가), 1236(binlog 불일치)
```

---

## 9. Failover 및 복구

- 계획된 점검: 애플리케이션 쓰기 트래픽 정지 → Slave 최신화 확인 → 승격
- 비계획 장애: Master 불가 시, 최신 Slave 승격 후 다른 Slave는 새 Master로 재구성

승격 예시(간단)

```sql
STOP SLAVE; 
RESET SLAVE ALL; 
SET GLOBAL read_only=0;  -- SUPER 권한 계정으로
-- VIP/프록시 전환 또는 DNS 업데이트 수행
```

구 Master 복구 후 재참여

- 재설치 또는 데이터 동기화 후 새 Master를 기준으로 CHANGE MASTER 구성

---

## 10. 모니터링 포인트

- Seconds_Behind_Master 추이, IO/SQL 스레드 상태, Last_*_Error
- 디스크 용량(binlog, relay log), IOPS, 네트워크 지연

---

## 12. 자주 발생하는 에러와 대응

- 1045 Access denied: 계정/비밀번호/호스트 권한 재확인
- 2003 Can't connect: 방화벽·보안그룹·DNS·포트 점검
- 1236 Could not find first log file: Master에서 binlog 만료 또는 삭제. 재덤프 후 재구성
- Duplicate key on slave: 충돌 레코드 정리 또는 재동기화

---

## 13. 운영 체크리스트

- [ ]  방화벽·보안그룹 3306 허용 및 네트워크 경로 확인
- [ ]  시간 동기화 정상
- [ ]  my.cnf 표준 적용 및 재기동 후 변수 확인
- [ ]  복제 계정 생성 및 최소 권한·접근 제한 적용
- [ ]  초기 동기화 수행 및 검증
- [ ]  CHANGE MASTER 구성, IO/SQL 스레드 정상 실행
- [ ]  모니터링 및 알림 연계
- [ ]  Failover 절차 문서화 및 정기 리허설

<aside>
✅

위 절차를 순서대로 수행하면 MariaDB 이중화 환경의 구축과 장애 대응을 일관되게 진행할 수 있습니다.

</aside>

[https://www.notion.so](https://www.notion.so)