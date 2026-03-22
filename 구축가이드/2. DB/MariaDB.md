# MariaDB

### 개요

이 문서는 MariaDB 설치부터 초기 보안 설정, 포트 변경, 문자셋 표준화, 기본 점검까지를 절차형으로 정리합니다.

---

### 1) 저장소 등록 및 설치

1. MariaDB 공식 사이트에서 OS와 버전에 맞는 리포지토리 설정을 복사합니다.
    - 사이트: [MariaDB Downloads](https://mariadb.org/download/?t=repo-config&d=Red%20Hat%20Enterprise%20Linux%208&v=10.5&r_m=blendbyte)
2. 리포지토리 파일 생성

```bash
vi /etc/yum.repos.d/MariaDB.repo
# 사이트에서 복사한 repo 내용을 붙여넣기
```

1. 설치 및 기동, 부팅 등록

```bash
yum install -y mariadb-server
systemctl enable mariadb
systemctl start mariadb
```

---

### 2) 초기 보안 설정(mysql_secure_installation)

아래 스크립트로 루트 비밀번호, 익명 사용자, 원격 root 접속, test DB 등을 정리합니다.

```bash
mariadb_secure_installation
```

권장 응답 가이드

- Switch to unix_socket authentication: y
- Change the root password?: y  → 새 비밀번호 입력
- Remove anonymous users?: y  → 익명의 유저 삭제
- Disallow root login remotely?: y  → root계정이 원격으로 접속하는것을 비활성화 여부
- Remove test database and access to it?: y  → 테스트 데이터베이스 삭제
- Reload privilege tables now?: y  → 변경사항 즉시 적용 여부

설정 후 접속 확인

```bash
mysql -u root -p
```

---

### 3) 포트 변경(선택)

1. 설정 파일 편집

```bash
vi /etc/my.cnf.d/server.cnf
```

1. [mysqld] 섹션에 포트 지정 후 저장

```
[mysqld]
port = 3306   # 필요 시 원하는 포트로 변경
```

1. 재시작

```bash
systemctl restart mariadb
```

---

### 4) 문자셋 표준화(UTF-8)

서버와 클라이언트 모두 UTF-8을 명시합니다.

1. 서버 설정

```bash
vi /etc/my.cnf.d/server.cnf
```

```
[mysqld]
skip-character-set-client-handshake
character-set-server = utf8
collation-server     = utf8_general_ci
init-connect         = SET NAMES utf8
```

1. 클라이언트 설정

```bash
vi /etc/my.cnf.d/client.cnf
```

```
[client]
default-character-set = utf8
```

1. 적용 및 확인

```bash
systemctl restart mariadb
mysql -u root -p -e "status" | egrep 'Server characterset|Db *characterset|Client characterset|Conn. *characterset'
```

---

### 5) 상태·로그·보관 정책 점검

1. 기본 상태 확인

```bash
mysql -u root -p -e "status"
```

1. 제네랄 로그 비활성화 권장

```sql
SHOW VARIABLES LIKE 'general%';
-- general_log 가 ON이면
SET GLOBAL general_log = OFF;
```

1. 바이너리 로그 보관일 설정(운영 정책에 맞게)

```sql
SHOW VARIABLES LIKE 'expire%';
-- 0이면 보관 없음 → 예: 3일 보관으로 설정
SET GLOBAL expire_logs_days = 3;
```

---

### 6) 운영 팁

- 방화벽 포트 오픈 확인: 3306
- SELinux 사용 시 포트 변경했다면 semanage로 포트 컨텍스트 추가 검토
- 루트 원격 접속은 금지하고, 운영 계정은 최소 권한으로 발급
- 설정 변경 시 항상 `systemctl restart mariadb` 전 `mysqld --verbose --help`로 파라미터 유효성 확인

---

### 다음 단계

- 
    
    [1-1 DB 내 스키마, 테이블에 적용 된 캐릭터셋 변경](MariaDB/1-1%20DB%20%EB%82%B4%20%EC%8A%A4%ED%82%A4%EB%A7%88,%20%ED%85%8C%EC%9D%B4%EB%B8%94%EC%97%90%20%EC%A0%81%EC%9A%A9%20%EB%90%9C%20%EC%BA%90%EB%A6%AD%ED%84%B0%EC%85%8B%20%EB%B3%80%EA%B2%BD%20260ae5fab89d808bae46c4b030e0b625.md)
    
- 
    
    [1-2 MariaDB 이중화 (1)](MariaDB/1-2%20MariaDB%20%EC%9D%B4%EC%A4%91%ED%99%94%20(1)%20284ae5fab89d8099aa78faea7206a373.md)
    
    [1-2 MariaDB 이중화](MariaDB/1-2%20MariaDB%20%EC%9D%B4%EC%A4%91%ED%99%94%20260ae5fab89d809f860bc1baf8284ef1.md)