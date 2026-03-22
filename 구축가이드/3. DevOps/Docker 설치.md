# Docker 설치

---

## 1. 개요

- 대상: Rocky Linux 계열
- 범위: 도커 RPM 수동 설치, docker-compose 준비, 컨테이너 파일 복사, 이미지 커밋·내보내기·불러오기, 실행, 이미지 교체 전략

---

## 2. Docker 설치(RPM 직접 설치)

1) 저장소 접속 및 RPM 다운로드

- 접속: [https://download.docker.com/linux/rhel/8/x86_64/stable/Packages/](https://download.docker.com/linux/rhel/8/x86_64/stable/Packages/로)
- 설치하려는 특정 버전의 RPM 패키지들을 다운로드합니다.

2) 필수 설치 RPM 목록

- [containerd.io](http://containerd.io)-1.7.18-3.1.el8.x86_64.rpm
- docker-ce-cli-27.0.3-1.el8.x86_64.rpm
- docker-compose-plugin-2.28.1-1.el8.x86_64.rpm
- docker-ce-27.0.3-1.el8.x86_64.rpm
- docker-ce-rootless-extras-27.0.3-1.el8.x86_64.rpm

---

## 3. docker-compose 준비

1) 바이너리 다운로드 경로 확인

- 접속: [https://github.com/docker/compose/releases?page=2에](https://github.com/docker/compose/releases?page=2에)
- 설치하고자 하는 docker-compose 버전을 찾아 서버로 이동
- 위의 docker-compose-plugin 버전과 맞춰서 선택
- 파일명 예: `docker-compose-linux-x86_64`

2) 서버에 docker-compose 배치

```bash
mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

3) 버전 확인 예시

```bash
docker-compose -v
# Docker Compose version v2.28.0
```

---

## 4. 컨테이너에서 서버로 파일 복사(docker cp)

- 명령 형식:

```bash
docker cp <컨테이너이름_or_ID>:<컨테이너_내_파일경로> <호스트_저장경로>
```

- 예시:

```bash
docker cp ed8a0e2be979:/data/dump.rdb /root
```

---

## 5. 도커 이미지 이식(커밋 · 저장 · 로드)

주의: 커밋을 하지 않으면 초기 모델로 복사됩니다.

1) 컨테이너를 이미지로 커밋

- 명령 형식:

```bash
docker commit -p <컨테이너ID> <저장할_이미지이름>
```

- 예시:

```bash
docker commit -p 2cbe40c3aaaa save_con
```

2) 이미지 저장(tar 파일로 내보내기)

- 확인: `docker images`로 이미지 존재 확인
- 명령 형식:

```bash
docker save -o <저장할_이름>.tar <이미지이름>
```

- 예시:

```bash
docker save -o image.tar save_con
```

3) 다른 환경으로 이동 후 이미지 로드

- tar 파일을 대상 Docker 서버로 복사한 뒤, 아래 명령 실행
- 명령 형식:

```bash
docker load < <도커이미지파일>.tar
```

- 예시:

```bash
docker load < save.tar
```

4) 이미지 실행

- 명령 형식:

```bash
docker run -d -it <호스트포트>:<컨테이너포트> <이미지이름>
```

- 예시:

```bash
docker run -d -it -p 6380:6379 rredis
```

---

## 6. 배포 시나리오(교체 전략)

- JAR만 전달 받기 → 서버의 JDK 17을 공유해 빌드 → 도커 이미지화 → 이미지 교체
- 이미지 자체를 전달 받기 → 기존 이미지 교체

---

## 부록

- docker-compose-plugin 버전과 docker-compose 바이너리 버전을 일치시키는 것을 권장
- 오프라인 환경에서는 RPM 의존성 체인을 사전에 확보해 둔 후 설치