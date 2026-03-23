# 1.2 PostgreSQL 이중화 (Archive-backup)

<aside>
🧭

PostgreSQL 물리 복제 + 아카이브(Archive) 기반 이중화 구성 절차입니다. 아래 내용은 기존 문구를 생략 없이 보존하면서 보기 좋게 정리한 것입니다.

</aside>

---

## 1. Master 서버 준비

### 1.1 Replication 전용 계정 및 슬롯 생성

```bash
su - postgres
psql
```

```sql
-- replication 복제 사용자 생성
CREATE ROLE repli WITH REPLICATION PASSWORD 'repli' LOGIN;
-- 전체 사용자 조회
SELECT usename FROM pg_user;
-- 물리 복제 슬롯 생성
SELECT * FROM pg_create_physical_replication_slot('standby');
-- 복제 슬롯 정보 조회
SELECT slot_name, restart_lsn FROM pg_replication_slots;
```

참고: psql에서 `\du`로 사용자 권한을 조회할 수 있습니다.

### 1.2 postgresql.conf 설정 (주석 해제 및 값 설정)

```
listen_addresses = '*'              # 모든 주소 수신
wal_level = replica                 # 대기 서버 읽기 전용 작업 가능 설정
max_wal_senders = 10                # WAL 전송 가능한 최대 서버 수
max_replication_slots = 10          # 복제 슬롯 최대 개수
hot_standby = on                    # 대기 서버 읽기 전용 작업 가능
archive_mode = on                   # 아카이빙 모드 설정
archive_command = 'cp %p /var/lib/pgsql/15/data/%f'
```

예시(설명 포함):

```
archive_command = 'cp %p /path/to/archive/%f'   # 백업 디렉토리 지정 가능
# /path/to/archive = 해당 파일 복사 디렉토리
```

### 1.3 pg_hba.conf 설정 (마스터, 슬레이브 IP 등록)

맨 아래에 다음을 추가합니다.

```
# replication privilege.
local   replication     all                                   peer
host    replication     all           127.0.0.1/32            scram-sha-256
host    replication     all           ::1/128                  scram-sha-256
host    replication     repli         172.16.0.8/32           trust    # 마스터 IP
host    replication     repli         172.16.0.18/32          trust    # 슬레이브 IP
```

### 1.4 서비스 재시작

```bash
systemctl restart postgresql-15
```

---

## 2. Slave 서버 준비

### 2.1 Replication 사용자 및 슬롯 생성

마스터와 동일한 이름으로 생성합니다.

```bash
su - postgres
psql
```

```sql
-- replication 복제 사용자 생성
CREATE ROLE repli WITH REPLICATION PASSWORD 'repli' LOGIN;
-- 전체 사용자 조회
SELECT usename FROM pg_user;
-- 물리적 복제 슬롯 생성
SELECT * FROM pg_create_physical_replication_slot('standby');
-- 복제 슬롯 정보 조회
SELECT slot_name, restart_lsn FROM pg_replication_slots;
```

### 2.2 postgresql.conf 설정 (슬레이브 측)

```
primary_conninfo = 'host=마스터서버IP주소 port=포트번호 user=replication사용자명 password=비밀번호'
primary_slot_name = '생성한 슬롯이름'
hot_standby = on
```

저장 후 서비스 재시작:

```bash
systemctl restart postgresql-15
```

---

## 3. 역할 상태 확인

```bash
su - postgres
psql
```

- 마스터 서버 상태 확인

```sql
SELECT pg_is_in_recovery();  -- f 라고 나오면 마스터
```

- 슬레이브 서버 상태 확인

```sql
SELECT pg_is_in_recovery();  -- t 라고 나오면 슬레이브
```

---

## 4. Base Backup로 동기화(슬레이브)

슬레이브에서 기존 데이터 제거 후 `pg_basebackup` 수행:

```bash
# 기존 데이터 디렉토리 삭제 (주의)
rm -rf /var/lib/pgsql/15/data

# 베이스백업 수행 (예시 포함)
su - postgres
pg_basebackup -h [마스터ip] -D [postgresql_data경로] -U [복제사용자] -v -P -X stream -S standby -R
# 예시
# pg_basebackup -h 192.168.56.105 -D /var/lib/pgsql/15/data -U repli -v -P -X stream -S standby -R
```

작업 완료 후 서비스 기동:

```bash
systemctl start postgresql-15   # 에러 없이 실행되면 성공
```

---

## 5. 동기화 및 복제 상태 확인 (마스터)

```sql
SELECT * FROM pg_stat_replication;   -- replication 상태 확인
```

> 참고: replication 상태에서 슬레이브 서버는 DDL(create, alter), DML(insert, select) 적용 제한이 있을 수 있습니다.
> 

---

## 부록: 요약 포인트

- Master에서 repli 계정 및 물리 복제 슬롯 생성 → 설정 반영(postgresql.conf, pg_hba.conf) → 서비스 재시작
- Slave에서 동일 계정 및 슬롯 생성 → primary_conninfo, primary_slot_name, hot_standby 설정 → 재시작
- 슬레이브 데이터 디렉토리 초기화 후 `pg_basebackup`로 동기화 → 서비스 기동
- `pg_is_in_recovery()`와 `pg_stat_replication`으로 역할과 상태 점검