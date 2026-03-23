# MongoDB

## 1) 설치 (YUM/DNF)

### 1-1. MongoDB Repository 설정

- 아래 파일을 생성/편집합니다.

```bash
vi /etc/yum.repos.d/mongodb-org-5.0.repo
```

- 파일 내용 예시 (아래는 7.0 repo 예시)

```
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/8/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://pgp.mongodb.com/server-7.0.asc
```

- 참고
    - 파일명이 `mongodb-org-5.0.repo`여도, 내부 repo 섹션/URL이 7.0이면 7.0 기준으로 내려받습니다.
    - 특정 버전으로 설치하려면 repo의 `baseurl`, `gpgkey`, 섹션명 등을 원하는 버전에 맞게 수정해야 합니다.

### 1-2. 패키지 설치

- 예시 (5.0.25 설치)

```bash
dnf install -y \
  mongodb-org-5.0.25 \
  mongodb-org-database-5.0.25 \
  mongodb-org-database-tools-extra-5.0.25 \
  mongodb-org-mongos-5.0.25 \
  mongodb-org-server-5.0.25 \
  mongodb-org-shell-5.0.25 \
  mongodb-org-tools-5.0.25
```

### 1-3. mongod 설정파일 확인/수정

- 설정파일 편집

```bash
vi /etc/mongod.conf
```

- 설정 내용 예시 (주요 항목)

```yaml
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log   # 로그 저장 경로

# Where and how to store data.
storage:
  dbPath: /var/lib/mongo             # 데이터 저장 경로
  journal:
    enabled: true

# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo
```

### 1-4. 서비스 기동

```bash
systemctl start mongod
```

- 단일(Single) 구성은 위 단계까지 진행하면 기본 기동 완료

### 1-5. 버전 업그레이드 주의

- 5.0 → 6.0 → 7.0 처럼 메이저 버전은 순차 업그레이드를 권장
- 또는 (상황에 따라) 이전 버전의 데이터 디렉토리(`/var/lib/mongo` 등)를 정리해야 하는 케이스가 있음

---

## 2) 접속

```bash
mongosh
```

- 접속 시 출력 예시로 아래 경고가 보일 수 있음
    - Access control 미설정 (인증/권한 설정 없음)
    - transparent hugepage 설정 관련 권고

---

## 3) 계정(User) 생성

- `admin` DB에서 사용자 생성 예시

```jsx
db.createUser({
  user: "master",
  pwd: "@zxcv()01@#",
  roles: [{ role: "root", db: "admin" }]
})
```

- 형식

```jsx
db.createUser({
  user: "사용할계정명",
  pwd: "패스워드",
  roles: [{ role: "부여할Role", db: "적용DB명" }]
})
```

---

## 4) 계정 패스워드 변경

1) 관리자 DB로 전환

```jsx
use admin
```

2) 패스워드 변경

```jsx
db.changeUserPassword("lemon", "DEVlemoncare26!")
```

---

## 5) Role(권한) 정리

### 최상위 (슈퍼 관리자급)

- `root`
    - 모든 권한 포함 (사용자/롤 관리, 모든 DB R/W, 클러스터 관리, 서버 제어 등)
    - 운영 환경에서는 최소 인원만 부여 권장

### 매우 높음 (클러스터 전체 제어)

- `clusterAdmin`
    - 클러스터 설정 전권 (ReplicaSet/Sharding/Balancer/Index 관리 등)
- `hostManager`
    - 서버 레벨 작업 (shutdown, fsync, 로그 회전 등) 가능
- `clusterManager`
    - ReplicaSet/Shard 관리, balancer, chunk 이동 등

### 높음 (모든 DB 접근)

- `readWriteAnyDatabase`
    - 모든 DB read/write (사용자/클러스터 설정은 불가)
- `dbAdminAnyDatabase`
    - 모든 DB index/collection/stats 관리, schema 변경 가능 (데이터 직접 수정은 아님)
- `readAnyDatabase`
    - 모든 DB read 가능

### 중간 (계정/권한 관리)

- `userAdminAnyDatabase`
    - 모든 DB 사용자/롤 관리
- `userAdmin`
    - 특정 DB 사용자 관리 (admin DB에서는 영향 큼)

### 일반 운영/업무용

- `dbOwner`
    - 해당 DB 내 모든 권한 (read/write + dbAdmin + userAdmin)
- `dbAdmin`
    - index/collection 관리, 통계 조회
- `readWrite`
    - 해당 DB read/write (가장 일반적인 애플리케이션 계정)
- `read`
    - 조회 전용

### 특수 목적 (운영/백업용)

- `backup`
    - 전체 DB 백업 가능
- `restore`
    - 데이터 복구 가능
- `__queryableBackup`
    - 내부용 (Atlas/백업 엔진)

### 시스템 내부 (부여 금지)

- `__system`
    - MongoDB 내부용 (사용자에게 부여 금지)

---

## 6) DB 생성(사용) 방법

1) 사용할 DB로 전환

```jsx
use lemon
```

2) 테스트 데이터 입력 (컬렉션 자동 생성)

```jsx
db.test.insertOne({ name: "test" })
```

3) DB 목록 확인

```jsx
show dbs
```