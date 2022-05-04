## Container 핵심 기술

-   Cgroup: Control Group(리소스 양)
-   Namespace: Isolation
    -   IPC NS: IPC
    -   PID NS: Process
    -   Network NS: Network
    -   UID NS: User/Group
    -   Mount NS: Mount Point
    -   UTS NS: Hostname
-   Layered Filesystem

Docker = Docker Engine

Docker CE: Community Edition  
Docker EE: Enterprise Edition

-   Docker 버전 관리의 변화

0.X -> 1.X(1.13.X) -> 17.04 -> 20.10

## Docker Engine 설치

> [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

```
sudo apt update
```

```
sudo apt install ca-certificates curl gnupg lsb-release
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt update
```

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

-   docker-ce: Docker Engine
-   docker-ce-cli: docker command
-   containerd.io: Container Runtime Interface
-   docker-compose-plugin: Docker Compose

```
sudo usermod -aG docker vagrant
```

## 컨테이너 이미지

(registry/)repository/name:tag

## Lifecycle

create -> start -> (pause) -> (unpause) -> (kill) -> stop -> rm  
run ---------->

> application이 종료 컨테이너도 종료(stop)

-   \-i: STDIN 유지
-   \-t: Terminal 할당
-   \-d: Detach

> \-it 옵션은 Shell을 실행하는 이미지에서 사용: centos, ubuntu ...  
> \-d 옵션 application이 계속적으로 실행되어햐 할 때: httpd ...

# Docker 관리

## 최신 docker 명령 구조

```
docker container <sub-command>
```

```
docker image <sub-command>
```

```
docker network <sub-command>
```

```
docker volume <sub-command>
```

## 이미지

로컬 이미지 목록 확인

```
docker images
```

Docker Hub 이미지 검색

```
docker search <TERM>
```

이미지 풀링

```
docker pull <IMAGE>:<TAG>
docker pull <IMAGE>@<HASH>
```

이미지 삭제

```
docker rmi <IMAGE>
```

이미지 상세정보 확인

```
docker inspect <IMAGE>
```

`ContainerConfig` vs `Config`

-   `ContainerConfig`: 이미지 최초 생성할 때 사용했던 설정
-   `Config`: 가장 최근에 이미지 생성시 사용했던 설정
-   `Config`
    -   `Env`
    -   `Cmd`
    -   `ExposedPorts`
    -   `WorkingDir`
    -   `Volume`
    -   `Entrypoint`
    -   `Volumes`

```
docker inspect centos:7 --format '{{ .Config.Cmd }}'
```

이미지 저장/아카이브

```
docker save <IMAGE> -o <FILE>
```

이미지 가져오기

```
docker load -i <FILE>
```

## 컨테이너

create -> start -> (pause) -> (unpause) -> (kill) -> (restart) -> stop -> rm  
run ---------->

컨테이너 목록

현재 실행중인 컨테이너 목록

```
docker ps
```

모든 컨테이너 목록

```
docker ps -a
```

컨테이너 실행

```
docker run <IMAGE>
docker run --name <IMAGE>
```

> 동일한 이름의 컨테이너 생성 X

-   옵션 없음: Docker Daemon --stdout/stderr-> Docker Clinet
    -   `--name` 옵션 X
-   `-it`: Attach 모드(stdin/stdout/stderr 연결) -> Foreground
    -   `-i`: stdin 유지
    -   `-t`: Terminal 할당
    -   `ctrl-p-q`
        -   `docker attach` 명령으로 연결 가능
-   `-d`: Detach 모드(stdin/stdout/stderr 연결 해제) -> Background 실행
-   `-itd`

> 하나의 컨테이너에는 하나의 Application만 실행 원칙

재시작 정책

> [https://docs.docker.com/config/containers/start-containers-automatically/](https://docs.docker.com/config/containers/start-containers-automatically/)

```
docker run --restart <no|always|on-failure|unless-stopped> <IMAGE>
```

이미지 풀 정책

```
docker run --pull <missing|always|never> <IMAGE>
```

컨테이너의 프로세스 목록 확인

```
docker top <CONTAINER>
```

> 실행중인 컨테이너만 확인

컨테이너에서 (추가)애플리케이션 실행

```
docker exec <CONTAINER> <COMMAND>
```

```
docker exec -it a8 bash
```

```
docker exec a8 cat /etc/httpd/conf/httpd.conf
```

컨테이너 리소스 사용량 확인

```
docker stats
docker stats --no-stream
```

컨테이너 (cpu/memory) 리소스 제한

```
docker run --cpus 0.1 --memory 100m ubuntu sha256sum /dev/zero
```

컨테이너 리소스 제한 변경

```
docker update --cpus 0.2 da
docker update --memory 200m da
```

컨테이너 로그(stdout/stderr) 확인

```
/var/lib/docker/container/<ID>/<ID>-json.log
```

```
docker logs <CONTAINER>
```

> 컨테이너를 삭제하면 로그도 삭제됨

환경변수

```
docker run -e A=100 ubuntu
docker run -d -e MYSQL_ROOT_PASSWORD=P@ssw0rd -e MYSQL_DATABASE=wordpress mysql:5.7
```

> 일부 이미지는 실행시 환경 변수가 필요함

컨테이너 정보 확인

```
docker inspect <CONTAINER>
```

컨테이너 IP 확인

```
docker inspect 16a -f '{{ .NetworkSettings.IPAddress }}'
```

컨테이너 Discovery

```
docker run --name mysqldb -d -e MYSQL_ROOT_PASSWORD=P@ssw0rd mysql:5.7 
```

```
docker run -it --link mysqldb ubuntu bash

> cat /etc/hosts
>> 172.17.X.X mysqldb
```

```
docker run -it --link mysqldb:xyz ubuntu bash

> cat /etc/hosts
>> 172.17.X.X mysqldb xyz
```

```
docker run -it --link mysqldb:xyz mysql:5.7 mysql -h xyz -u root -p
```

컨테이너의 포트를 호스트의 포트로 포워딩 (포트 포워딩)

```
docker run -p <HOST>:<CONTAINER> <IMAGE>
```

```
docker run -d -p 80:80 httpd
```

### Docker를 사용해 워드프레스 설치하기

```
docker run --name wp-db -d -e MYSQL_ROOT_PASSWORD=P@ssw0rd -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wpadm -e MYSQL_PASSWORD=P@ssw0rd --restart always --cpus 0.5 --memory 1000m mysql:5.7
```

```
docker run --name wp-web -d --link wp-db:mysql -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=wpadm -e WORDPRESS_DB_PASSWORD=P@ssw0rd -e WORDPRESS_DB_NAME=wordpress --restart always --cpus 0.5 --memory 500m -p 80:80 wordpress:5-apache
```