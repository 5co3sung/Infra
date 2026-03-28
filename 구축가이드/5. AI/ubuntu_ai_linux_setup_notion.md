# Ubuntu / AI·Linux 환경 설정 문서 정리본

GPU가 필요한 이유 : 산술연산을 병렬로 처리하기 위해 사용
RTX3090에서 90은 CPU안에 있는 보조산술연산의 개수가 많다는 것을 의미

## 1. Ubuntu 기본 설정

### 1-1. 패키지 목록 업데이트

```bash
apt-get update
```

### 1-2. make 설치

```bash
sudo apt-get install gcc make
```

## 2. SSH 설치 및 설정

### 2-1. SSH 설치

```bash
sudo apt-get purge openssh-server
sudo apt-get install openssh-server
```

### 2-2. SSH 설정 파일 수정

```bash
vi /etc/ssh/sshd_config
```

#### 수정 내용

```conf
Port 22
AddressFamily any
ListenAddress 0.0.0.0
PermitRootLogin no
PasswordAuthentication yes
```

### 2-3. SSH 재시작

```bash
service ssh restart
```

## 3. 사용자 생성 및 계정 설정

### 3-1. 사용자 및 그룹 생성

```bash
useradd alosm
passwd alosm
groupadd work
usermod -G work alosm
```

### 3-2. 홈 디렉터리 생성 및 권한 설정

```bash
mkdir /home/alosm
chown -R alosm:work /home/alosm
usermod -s /bin/bash alosm
```

### 3-3. 계정 확인

```bash
cat /etc/passwd
```

## 4. sudo 권한 추가

### 4-1. sudoers 파일 수정 권한 부여

```bash
chmod +w /etc/sudoers
```

### 4-2. sudoers 설정

```bash
vi /etc/sudoers
```

#### 추가 내용

```conf
alosm	ALL=(ALL) NOPASSWD: ALL
```

## 5. 성능 설정

### 5-1. limits.conf 수정

```bash
vi /etc/security/limits.conf
```

#### 추가 내용

```conf
* soft     nproc          65535
* hard     nproc          65535
* soft     nofile         65535
* hard     nofile         65535
```

### 5-2. 참고 변경 항목

```bash
#ulimit -n 65535    open files 변경
#ulimit -u 79100    max user processes 변경
```

## 6. alosm 계정으로 접속 후 설정

## 7. 한글 설정

### 7-1. vim 관련 파일 수정

```bash
vi ~/.virc
vi ~/.vimrc
```

#### 설정 내용

```vim
set encoding=utf-8
set fileencodings=utf-8,cp949
```

## 8. 환경 설정

### 8-1. bashrc 수정

```bash
vi ~/.bashrc
```

#### 설정 내용

```bash
export LANG=ko_KR.UTF-8
export SUPPORTED=ko_KR.UTF-8:ko_KR:ko
export SYSFONT=lat0-sun16

alias python='python3'
export PYTHONPATH=/usr/bin/python3

export PS1='[$USER@HOSTNAME:$PWD] \[\033[1;37m\]'
```

### 8-2. bash_profile 수정

```bash
vi ~/.bash_profile
```

#### 설정 내용

```bash
source ~/.bashrc
```

## 9. JDK 설치

```bash
sudo apt install openjdk-17-jre-headless
```

## 10. Python 설치

```bash
sudo apt-get install python3-pip
```

#### 참고

```bash
#sudo apt remove pip
```

## 11. Python 패키지 설치

### 11-1. 기본 패키지

```bash
pip install gluonnlp pandas tqdm
pip install mxnet
pip install sentencepiece==0.1.91
pip install transformers==4.8.2
```

### 11-2. 시간 오래 걸리는 패키지

```bash
pip install torch
pip install git+https://git@github.com/SKTBrain/KoBERT.git@master
```

### 11-3. 추가 패키지

```bash
pip install mxnet-mkl==1.6.0 numpy==1.23.1
pip install scikit-learn
pip install jupyter
```

## 12. 빠짐없이 체크할 수 있는 작업 체크리스트

### 시스템 기본

- [ ] `apt-get update`
- [ ] `gcc`, `make` 설치

### SSH

- [ ] `openssh-server` 삭제
- [ ] `openssh-server` 재설치
- [ ] `/etc/ssh/sshd_config` 수정
- [ ] `service ssh restart`

### 사용자/권한

- [ ] `alosm` 사용자 생성
- [ ] 비밀번호 설정
- [ ] `work` 그룹 생성
- [ ] `alosm`을 `work` 그룹에 추가
- [ ] `/home/alosm` 생성
- [ ] 소유권 변경
- [ ] bash shell 설정
- [ ] `/etc/passwd` 확인

### sudo

- [ ] `/etc/sudoers` 쓰기 권한 부여
- [ ] `alosm ALL=(ALL) NOPASSWD: ALL` 추가

### 성능

- [ ] `/etc/security/limits.conf` 수정
- [ ] `nproc` 설정
- [ ] `nofile` 설정
- [ ] `ulimit -n`, `ulimit -u` 검토

### alosm 계정 접속 후

- [ ] `~/.virc` 수정
- [ ] `~/.vimrc` 수정
- [ ] `~/.bashrc` 수정
- [ ] `~/.bash_profile` 수정

### 개발환경

- [ ] OpenJDK 설치
- [ ] `python3-pip` 설치
- [ ] Python 패키지 설치
- [ ] `jupyter` 설치

