# Docker 이미지 빌드

방법
- `docker commit` 명령
- Dockerfile

## 명령으로 이미지 생성

### `docker diff`
```
docker diff <CONTAINER>
```

기준 이미지 <-> 컨테이너 차이
- A: Add
- C: Change
- D: Delete

### `docker commit`
```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

CMD 변경
```
docker commit -c "CMD XXX" CONTAINER [REPOSITORY[:TAG]]
```

ExposedPort 변경
```
docker commit -c "EXPOSE PORT/PROTOCOL" CONTAINER [REPOSITORY[:TAG]]
```

> ExposedPort는 실제 작동여부와 상관 X

> CMD /usr/sbin/apache2ctl -D FOREGROUND
> 
> /bin/sh -c /usr/sbin/apache2ctl -D FOREGROUND

### `docker cp`
컨테이너 -> 도커 호스트
```
docker cp CONTAINER:SRC_PATH DEST_PATH
```

도커 호스트 -> 컨테이너
```
docker cp SRC_PATH CONTAINER:DEST_PATH
```

/bin/sh -c /usr/sbin/apache2ctl -DFOREGROUND

### 이미지 레이어

레이어를 가지는 이유? 저장소 및 네트워크 전송 효율성 높임

httpd:latest
```
sha256:9c1b6dd6c1e6be9fdd2b1987783824670d3b0dd7ae8ad6f57dc3cea5739ac71e sha256:1d1a2486e901871ad1257512d588eebb30ae0605d8353abb6635e2d313b2187c sha256:ec02eb7f3cf4dd78bc518d3a8ccf57f57336ceacb9638303891787ff2ec2e96f sha256:67bb571b5bd2b65cbdf2c93c6f9dc8e89414055dd7444df617b23466996f3be7 sha256:e83f42350a11889083536c5af330dcf15fd3624f8e956f4086e1f0cfb07ff246
```

myhttpd:latest
```
sha256:9c1b6dd6c1e6be9fdd2b1987783824670d3b0dd7ae8ad6f57dc3cea5739ac71e sha256:1d1a2486e901871ad1257512d588eebb30ae0605d8353abb6635e2d313b2187c sha256:ec02eb7f3cf4dd78bc518d3a8ccf57f57336ceacb9638303891787ff2ec2e96f sha256:67bb571b5bd2b65cbdf2c93c6f9dc8e89414055dd7444df617b23466996f3be7 sha256:e83f42350a11889083536c5af330dcf15fd3624f8e956f4086e1f0cfb07ff246 sha256:4b8c655a2e61d6f017f3a5cc103fe6bbb6f71bba663085fdae697861dd57695a
```

myhttpd:p8080
```
sha256:9c1b6dd6c1e6be9fdd2b1987783824670d3b0dd7ae8ad6f57dc3cea5739ac71e sha256:1d1a2486e901871ad1257512d588eebb30ae0605d8353abb6635e2d313b2187c sha256:ec02eb7f3cf4dd78bc518d3a8ccf57f57336ceacb9638303891787ff2ec2e96f sha256:67bb571b5bd2b65cbdf2c93c6f9dc8e89414055dd7444df617b23466996f3be7 sha256:e83f42350a11889083536c5af330dcf15fd3624f8e956f4086e1f0cfb07ff246 sha256:4b8c655a2e61d6f017f3a5cc103fe6bbb6f71bba663085fdae697861dd57695a sha256:fffe7ea283ae8867d740e17f619c4db845cb0d75fc655809185e582601786521
```

### `docker history`
이미지 생성 로그 확인
```
docker history <IMAGE>
```

### `docker export`
컨테이너 -> 아카이브
```
docker export <CONTAINER> -o <TARFILE>
```

### `docker import`
아카이브 -> 이미지
```
docker import <TARFILE> <IMAGE>:<TAG>
```

> 1개의 레이어로 가져오기

## Dockerfile 이미지 생성

### FROM
Base Image
```
FROM <image>
FROM <image>[:<tag>]
FROM <image>[@<digest>]
```

### RUN
빌드시 실행할 명령

Shell Form
```
RUN yum install httpd
```
=> `/bin/sh -c yum install httpd`

Exec Form
```
RUN ["yum", "install", "httpd"]
```
=> `exec yum install httpd`

> exec는 명령 X

### CMD
기본 실행 어플리케이션

Shell Form
```
CMD /usr/sbin/httpd -DFOREGROUND
```
=> `/bin/sh -c yum install httpd`

Exec Form(선호)
```
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

### EXPOSE
외부에 노출할 포트 및 프로토콜
```
EXPOSE <PORT>/<PROTOCOL>
```

### ENV
쉘 환경 변수
```
ENV MY_NAME="John Doe"
ENV MY_DOG=Rex\ The\ Dog
ENV MY_CAT=fluffy
```

### ADD/COPY
도커 호스트 -> 컨테이너 파일 복사
```
ADD <SRC> <DEST>
COPY <SRC> <DEST>
```

> ADD는 SRC로 URL 지정 가능

### ENTRYPOINT
CMD의 명령으로 사용

|ENTRYPOINT |CMD |Result |
|-|-|-|
| X | X | 허용X |
| X | abc | abc |
| xyz | X | xyz |
| xyz | abc | xyz abc |

### VOLUME
자동으로 마운트할 볼륨 마운트 포인트 지정
```
VOLUME /myvol
```

### WORKDIR
작업 디렉토리
`RUN`, `CMD`, `ENTRYPOINT`, `ADD`, `COPY`에 영향을 미침
```
WORKDIR /usr/local
```

### 레이어
`RUN`, `ADD`, `COPY` 레이어를 만듦

## 예제
```
mkdir -p ~/image-build/myweb
cd ~/image-build/myweb
```

`Dockerfile`
```
FROM centos:7
RUN yum install -y httpd
ADD index.html /var/www/html/index.html
CMD ["/usr/sbin/httpd", "-DFOREGROUND" ]
EXPOSE 80/tcp
```

`index.html`
```
hello dockerfile
```

```
docker build -t myweb:centos .
```

---
## Linux Timezone 설정
`/etc/localtime` 파일이 가리키고 있는 파일이 시간대

시간대
`/usr/share/zoneinfo/*`

```
sudo ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```

```
sudo timedatectl
```

> ubuntu 20.04/focal 이미지 부터 시간대가 설정되어 있지 않음
> 우회: ENV DEBIAN_FRONTEND=noninteractive

## Dockerfile 명이 다른 경우
`Dockerfile-alpine`
```
docker build -f Dockerfile-alpine -t pyhello:v1 .
```

---

## 이미지 빌드 및 태그, 푸시

```
docker build -t c1t1d0s7/mygo:latest -t c1t1d0s7/mygo:1.0 -t c1t1d0s7/mygo:1 .
```

```
docker push c1t1d0s7/mygo:latest c1t1d0s7/mygo:1.0 c1t1d0s7/mygo:1
```

```
docker build -t c1t1d0s7/mygo:latest -t c1t1d0s7/mygo:1.1 -t c1t1d0s7/mygo:1 .
```

```
docker push c1t1d0s7/mygo:latest c1t1d0s7/mygo:1.1 c1t1d0s7/mygo:1
```

```
docker build -t c1t1d0s7/mygo:latest -t c1t1d0s7/mygo:1.2 -t c1t1d0s7/mygo:1 .
```

```
docker push c1t1d0s7/mygo:latest c1t1d0s7/mygo:1.2 c1t1d0s7/mygo:1
```

```
docker build -t c1t1d0s7/mygo:latest -t c1t1d0s7/mygo:2.0 -t c1t1d0s7/mygo:2 .
```

```
docker push c1t1d0s7/mygo:latest c1t1d0s7/mygo:2.0 c1t1d0s7/mygo:2
```

---

```
docker build -t mygo
```

```
docker tag mygo c1t1d0s7/mygo:2.1
```
