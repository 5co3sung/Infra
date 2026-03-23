# Jenkins 설치

---

## 1. 개요

- 대상: Rocky/RHEL 계열
- 요구사항: 2023년 기준 Java 10 이상 권장. JDK 17 사용 시 JAVA_HOME 등 환경변수 미설정이면 Jenkins가 기동되지 않음

---

## 2. Java 설치 및 환경 설정

### 2.1 설치 가능한 Java 목록 확인

```bash
yum list java*jdk-devel
```

- 2023 기준 10버전 이상 사용 필요
- JDK 17로 설치 시 환경변수 미설정이면 Jenkins 기동 불가

### 2.2 기본 Java 경로 전환(alternatives)

```bash
alternatives --install /usr/bin/java java /usr/local/jdk-17/bin/java 100
alternatives --install /usr/bin/java java <자바설치경로> 100
/usr/sbin/alternatives --config java
```

### 2.3 설치 및 JAVA_HOME 설정

1) Java 설치

```bash
yum install <자바버전>
```

2) 컴파일러 위치 확인

```bash
which javac
# 예) /usr/bin/javac
```

3) 실제 경로 확인(JAVA_HOME 파악)

```bash
readlink -f /usr/bin/javac
# 예) /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.392.b08-3.el9.x86_64/bin/javac
```

4) 환경변수 등록 및 적용

```bash
vi /etc/profile
# 파일 맨 아래에 추가
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.392.b08-3.el9.x86_64/
source /etc/profile
```

---

## 3. Jenkins 저장소 등록 및 설치

### 3.1 레포 등록(옵션 1: config-manager)

```bash
yum config-manager --add-repo [https://pkg.jenkins.io/redhat-stable/jenkins.repo](https://pkg.jenkins.io/redhat-stable/jenkins.repo)
```

### 3.2 레포 등록(옵션 2: repo 파일 직접 다운로드)

```bash
wget -O /etc/yum.repos.d/jenkins.repo [https://pkg.jenkins.io/redhat-stable/jenkins.repo](https://pkg.jenkins.io/redhat-stable/jenkins.repo)
```

### 3.3 GPG 키 등록

```bash
rpm --import [https://pkg.jenkins.io/redhat-stable/jenkins.io.key](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)
```

### 3.4 Jenkins 설치

```bash
yum install epel-release
yum install jenkins
```

---

## 4. Jenkins 기동 및 초기 접속

### 4.1 서비스 기동

```bash
systemctl start jenkins
```

### 4.2 웹 콘솔 접속

- 브라우저에서: http://<jenkins_server_ip>:8080

### 4.3 초기 관리자 비밀번호 확인

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
# 예) a790ced49f334ddabd9809f263a1188f
```

---

## 5. Jenkins 포트 변경

```bash
sudo vi /usr/lib/systemd/system/jenkins.service
# 아래 항목 편집/추가
Environment="JENKINS_PORT=<사용할_포트>"

systemctl daemon-reload
systemctl restart jenkins
```

---

## 6. Jenkins View 생성

- 초기 상태에서 생성하려면: 대시보드 → 새로운 Item 클릭 → Folder 생성 후 구성 진행

---

## 7. Jenkins 파일 작성(파이프라인)

- 파이프라인을 적용할 Git 저장소를 다운로드한 후 Jenkins에서 Pipeline 설정에 연결

---

## 8. 설치 이후 초기 설정 화면 안내

- Jenkins 설치가 완료된 후, Jenkins 서버 IP 또는 공인 IP의 포트포워딩 포트로 접속하면 아래와 같은 초기 화면이 표시됩니다.

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image.png)

- 설치된 서버에서 다음 명령으로 초기 비밀번호를 확인하고, 로그인 화면에 입력합니다.

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword

```

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%201.png)

- 설치 화면에서 우측의 "특정 플러그인만 설치" 옵션으로 진행할 수 있습니다.

기본 설치를 선택하면 불필요한 플러그인이 많이 설치될 수 있습니다.

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%202.png)

---

환경설정 – 환경에 맞게 선택 할 수 있도록 하자 아래는 기본적으로 들어가는 요소인듯

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%203.png)

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%204.png)

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%205.png)

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%206.png)

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%207.png)

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%208.png)

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%209.png)

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%2010.png)

           체크 후 하단의 install 버튼을 누르면 다음과 같이 설치창이 뜨는지 확인

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%2011.png)

설치가 완료되면 관리자 계정 생성 추가를 요구한다
참고로 계정명에 특수기호가 되지 않기 떄문에 확인바람.

![image.png](Jenkins%20%EC%84%A4%EC%B9%98/image%2012.png)

부록

- 주의: JDK 17 사용 시 반드시 JAVA_HOME을 설정해야 Jenkins 서비스가 정상 기동됩니다.
- 참고: alternatives로 자바 버전 전환 후 `java -version`으로 적용 상태 확인

[빌드 이미지를 사설 repository에서 가져오는 방법](Jenkins%20%EC%84%A4%EC%B9%98/%EB%B9%8C%EB%93%9C%20%EC%9D%B4%EB%AF%B8%EC%A7%80%EB%A5%BC%20%EC%82%AC%EC%84%A4%20repository%EC%97%90%EC%84%9C%20%EA%B0%80%EC%A0%B8%EC%98%A4%EB%8A%94%20%EB%B0%A9%EB%B2%95%20324ae5fab89d80ee917bdce7c9fff345.md)

[CDN](Jenkins%20%EC%84%A4%EC%B9%98/CDN%20324ae5fab89d80d88255c8c5f2839358.md)