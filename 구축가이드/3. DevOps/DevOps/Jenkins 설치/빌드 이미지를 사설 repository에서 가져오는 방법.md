# 빌드 이미지를 사설 repository에서 가져오는 방법

## 빌드 이미지를 사설 Repository에서 가져오는 방법

빌드 시 사용하는 이미지를 외부(public)에서 직접 pull 하지 않고, 사설 레지스트리에 올려서 사용한다. (외부로 나가는 통신 최소화 목적)

### 절차

1. Jenkins 서버에서 필요한 이미지를 먼저 pull 한다.

```bash
[root@jenkins-svr ~]# docker pull nginx:alpine
alpine: Pulling from library/nginx
589002ba0eae: Already exists
bca5d04786e1: Already exists
3e2c181db1b0: Already exists
6b7b6c7061b7: Already exists
399d0898a94e: Already exists
955a8478f9ac: Already exists
6d397a54a185: Already exists
5e7756927bef: Already exists
Digest: sha256:1d13701a5f9f3fb01aaa88cef2344d65b6b5bf6b7d9fa4cf0dca557a8d7702ba
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine
```

이미지가 정상적으로 받아졌는지 확인한다.

```bash
[root@jenkins-svr ~]# docker images | grep nginx
nginx  alpine  b76de378d572  2 weeks ago  62.1MB
```

1. 사설 레지스트리 주소로 태그를 변경한다.

```bash
docker tag nginx:alpine krukogd9.private-ncr.gov-ntruss.com/nginx:alpine
```

1. 사설 레지스트리에 로그인한다.

```bash
docker login <사설레지스트리주소>
```

1. 로그인 성공 후, 사설 레지스트리에 이미지를 push 한다.

```bash
[root@jenkins-svr ~]# docker push krukogd9.private-ncr.gov-ntruss.com/nginx:alpine
The push refers to repository [krukogd9.private-ncr.gov-ntruss.com/nginx]
da3ac26fdf0f: Mounted from nfh-bo-ui
7b2903554e63: Mounted from nfh-bo-ui
6c0e59fd138a: Mounted from nfh-bo-ui
aa0fc249df10: Mounted from nfh-bo-ui
f35bcec50d8c: Mounted from nfh-bo-ui
660f9a93104f: Mounted from nfh-bo-ui
53998a5033c3: Mounted from nfh-bo-ui
989e799e6349: Mounted from nfh-bo-ui
alpine: digest: sha256:0b506371247cdd8973b4d196ae3b50bae6d64f173df2df2865fa03fbd4867040 size: 1989
[root@jenkins-svr ~]#
```