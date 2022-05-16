# Docker 네트워크

> 팁! 파일명으로 패키지 찾을 때
> 
> yum provides /usr/bin/ip
> 
> apt install apt-file
> apt-file update
> apt-file search /usr/bin/ip

네트워크 플러그인 종류
- **bridge**: 기본 네트워크
- **host**
- null
- ipvlan, macvlan, overlay

## 브릿지 네트워크
### 호스트 확인
인터페이스 확인
```
ip addr show
```
- docker0: 브릿지
- vethX: 가상 인터페이스

브릿지 확인 명령 설치
```
sudo apt install bridge-utils
```

```
brctl show
```

NAT 테이블 확인
```
sudo iptables -t nat -L -n
```
- MASQUERADE: Source NAT

### 컨테이너 확인
```
sudo apt update
sudo apt install iproute2
```

```
ip addr show
```

```
ip route
```

### 포트 포워딩이 설정된 컨테이너
```
sudo iptables -t nat -L -n
```
- DNAT: Destination NAT

## 호스트 네트워크
호스트의 네트워크를 공유해서 사용
```
docker run -d --network host httpd
```

## Null 네트워크
네트워크가 없는 컨테이너 생성
```
docker run -d --network none httpd
```

## 브릿지 네트워크 생성
```
docker network create --driver bridge --subnet 192.168.200.0/24 --gateway 192.168.200.1 wordpress-network
```

```
docker run --name wp-db -d \
-e MYSQL_ROOT_PASSWORD=P@ssw0rd \
-e MYSQL_DATABASE=wordpress \
-e MYSQL_USER=wpadm \
-e MYSQL_PASSWORD=P@ssw0rd \
--restart always --cpus 0.5 --memory 1000m \
-v wp-db-vol:/var/lib/mysql \
--network wordpress-network \
mysql:5.7
```

```
docker run --name wp-web -d \
--link wp-db:mysql \
-e WORDPRESS_DB_HOST=mysql \
-e WORDPRESS_DB_USER=wpadm \
-e WORDPRESS_DB_PASSWORD=P@ssw0rd \
-e WORDPRESS_DB_NAME=wordpress \
--restart always --cpus 0.5 --memory 500m \
-p 80:80 -v wp-web-app:/var/www/html \
--network wordpress-network \
wordpress:5-apache
```
