## 클라우드 미니 프로젝트
- AWS 클라우드를 이용한 고가용성 Wordpress 서비스 배포

## 목차
1. 프로젝트 개요
	1-1)  목표
	1-2) 진행 기간
	1-3) 인프라 구축 환경
	1-4) 아키텍쳐 구성
2. 구성 과정
	2-1) VPC 생성
	2-2) 보안 그룹 생성
	2-3) EC2 인스턴스 생성 및 배스쳔 호스트 연결
	2-4) DB 생성
	2-5) WordPress 구현
	2-6) Auto Scaling 구현
	2-7) Load Balancer를 통한 접속 확인
	2-8) Cloud Front를 이용한 정적 호스팅
	


## 1. 프로젝트 개요
### 1-1) 목표
- VPC, 보안 그룹, 베스천 호스트를 활용해 보안을 강화하고 Auto Scaling과 ELB를 통해 부하 분산이 가능한 WordPress를 AWS 서버 위에 구현하는 것이 목표입니다.

### 1-2) 진행 기간
- 2022.04.07(목) ~ 2022.04.09(토)

### 1-3) 인프라 구축 환경
- Linux
- WordPress
- Apache
- PHP
- MariaDB
- AWS EC2
- AWS Security group
- AWS VPC
- AWS RDS
- AWS Auto Scaling
- AWS Cloud Front
- AWS S3

### 1-4) 아키텍쳐 구성
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSSV9c%2FbtryRt85yeQ%2FANsezH3TVkxgMWlBZWuzUK%2Fimg.png)

## 2. 구성 과정
### 2-1) VPC 생성
- 모든 시작에 앞서 우리가 사용할 공간을 VPC로 만들어줍니다.
- VPC 생성 탭으로 들어가서 VPC와 서브넷을 생성해줍니다.
- VPC의 이름 태그를 입력하고, 가용성과 DB를 위해 가용영역은 2개로 지정합니다.
- 이때 프리 티어 사용을 위해 프리 티어가 지원되는 a와 c를 선택합니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqntrM%2FbtryLH8ng3A%2FsNvu1JgjDLpPi7L0ruR6NK%2Fimg.png)

### 2-2) 보안 그룹 생성
- 웹 서버와 DB, 배스쳔 호스트 등에 사용할 보안 그룹을 만들어줘야 합니다.  
- 공통적으로 이전에 생성한 VPC를 선택합니다.
- 먼저 관리자들이 접속할 배스쳔 호스트 생성을 위해 배스천 호스트에 접속할 수 있도록 내 IP의 SSH 접속을 허용해준 보안 그룹을 생성합니다.  

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb9tZ0O%2FbtryOljyNEg%2FzokZrk7ps44cqxGDm5ANo0%2Fimg.png)
- 워드프레스를 설치할 웹 서버의 보안 그룹을 생성해준다. 워드 프레스는 로드 밸런서 혹은 배스천 호스트를 통해 접속할 것이기에 배스천 호스트 보안 그룹에 속한 대상을 SSH 접속 허용하도록 보안 그룹을 등록합니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FuMQDO%2FbtryMmKvl9R%2FHZUk3xqNMFWkDMLgkZBmz1%2Fimg.png)
- 마지막으로 데이터 베이스에서 사용할 보안 그룹을 생성해준다. 배스쳔 호스트에서 관리를 위해 DB에 접속하거나 웹 서버에서 접속이 필요하니 이전에 생성한 두 보안 그룹을 MYSQL 접속 허용으로 추가합니다.
- ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FrD4pi%2FbtryNjl9d9C%2FgTvV60s9udfYYDTMySycqk%2Fimg.png)

### 2-3) EC2 인스턴스 생성 및 배스쳔 호스트 연결
- 보안 그룹을 모두 만들었으니 이에 해당하는 인스턴스를 만들 차례입니다. 일단 하나의 인스턴스에 워드 프레스를 만들어주고 이미지를 찍어 오토 스케일링 그룹으로 묶습니다.
- 먼저 배스쳔 호스트로 사용할 EC2를 생성해줍니다. 이는 외부에서 접속이 가능한 퍼블릭 서브넷으로 설정해주고, 퍼블릭 IP를 할당하도록 지정합니다.
- 보안 그룹은 조금전 생성해준 배스쳔 호스트용 보안 그룹을 등록합니다.
![](https://blog.kakaocdn.net/dn/cgcRs0/btryNa3MSSD/I5vo3WetEzC53DKOz09grK/img.png)
- 이제 Wordpress를 설치할 웹 서버 인스턴스를 만듭니다. 외부에서 접속이 불가능한 프라이빗 서브넷으로 설정하고, 퍼블릭 IP를 할당하지 않습니다.
- 보안 그룹은 조금 전 생성한 웹 서버용 보안 그룹을 등록합니다. 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcUygzr%2FbtryPri1UgV%2FVq1ZYvlto01nNhcoulQQU0%2Fimg.png)
- 점프 호스트를 이용해 배스쳔 호스트에서 웹 서버로 접속이 가능한지 확인합니다.
```
PS C:\Users\rkdtk> ssh -J ec2-user@3.36.116.80 ec2-user@10.0.131.242
The authenticity of host '3.36.116.80 (3.36.116.80)' can't be established.
ECDSA key fingerprint is SHA256:ougHceiFjRlSyku31Ktwftx/yzU9cshrq0eRLt+5eiI.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Warning: Permanently added '3.36.116.80' (ECDSA) to the list of known hosts.
The authenticity of host '10.0.131.242 (<no hostip for proxy command>)' can't be established.
ECDSA key fingerprint is SHA256:5G07f5JJwEQbvTSiNCZH1yQB0PKD2x3OgY2IjEwPYvg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.131.242' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
14 package(s) needed for security, out of 17 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-10-0-131-242 ~]$
```
  
### 2-4) DB 생성
- 이제 워드프레스에서 사용할 DB를 AWS RDS를 통해 만들 차례입니다.
- 먼저 DB에게 서브넷을 할당해주기 위해서 서브넷 그룹을 생성해줍니다.
- DB의 가용성을 높여주기 위해서 2개의 가용 영역이 필요하기 때문에 VPC의 서브넷 탭에서 새로운 프라이빗 서브넷을 먼저 2개 생성해줍니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbqwK7g%2FbtryO5IwJoG%2FsQqNnh9eRRwquRLJvO877k%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoIVrE%2FbtryP8LjAAn%2FWE9H9DdkBSy0RwDxxwTbpK%2Fimg.png)
- 해당 서브넷을 묶어서 AWS RDS 메뉴에 있는 서브넷 그룹을 생성해줍니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FrlEKl%2FbtryOMJbkLw%2F7F8YcMZN8wBqHKABacR3Sk%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fchqbq1%2FbtryRStU0HH%2FQH6VDLowc7gPivHubEn9L0%2Fimg.png)
- 이후 AWS RDS로 들어가서 데이터베이스 생성을 해줍니다.
- 본 실습은 MySQL로 진행했습니다. 데이터베이스의 이름과 어드민, 암호 등을 설정하고 DB가 속하는 VPC를 설정하면 서브넷 그룹에서 조금 전에 만든 서브넷 그룹을 선택할 수 있습니다.
- 프라이빗 서브넷에서 작동할 DB이기에 퍼블릭 IP를 할당하지 않고 처음에 만든 DB용 보안 그룹을 선택해줍니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc9fCi2%2FbtryPTt4JI0%2FKxOYRzgpIWNk34peZHJkL0%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb8VuxT%2FbtryPqMs4qq%2FsolKireJbU6bZlCxnKsVv0%2Fimg.png)
- 이후 초기 데이터 베이스 설정을 작성하고 생성해줍니다.  

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbsAOyT%2FbtryRsWw880%2F4D5tLkQryecKn0iBNCPj7K%2Fimg.png)

- 이제 EC2와의 연결을 확인해봅니다.
- EC2에서 DB와의 연결을 확인하기 위해서 Web 서버 인스턴스에 접속해 MySQL 을 설치해줍니다.
```
[ec2-user@ip-10-0-131-242 ~]$ sudo yum install mysqldb
```

- 만들어진 DB의 상세 페이지에 들어가서 엔드 포인트를 확인합니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbIox9e%2FbtryRQpkrGu%2FAqGEsRME29MmpHjcaJtDI0%2Fimg.png)

- 이후 웹 서버 인스턴스에서 명령어를 통해 DB 접속을 확인해봅니다.
- `mysql -u admin -p wordpress -h 엔드포인트`

```
[ec2-user@ip-10-0-131-242 ~]$ mysql -u admin -p wordpress -h wordpress-db.co4sy4ji3quq.ap-northeast-2.rds.amazonaws.com
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 25
Server version: 10.6.7-MariaDB-log managed by https://aws.amazon.com/rds/

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [wordpress]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| innodb             |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
6 rows in set (0.00 sec)
```

- 이제 생성된 DB에 wordpress에서 사용할 사용자를 만들고 사용자에게 게시판 사용 권한을 부여합니다.
```
MySQL [wordpress]> CREATE USER adminuser@'%' IDENTIFIED BY 'qwer1234';  //사용자 생성
Query OK, 0 rows affected (0.01 sec)

MySQL [wordpress]> GRANT ALL PRIVILEGES ON wordpress.* TO adminuser@'%';  // 게시판 사용 권한 부여
Query OK, 0 rows affected (0.01 sec)

MySQL [wordpress]> FLUSH PRIVILEGES; //권한 적용
Query OK, 0 rows affected (0.01 sec)

MySQL [wordpress]>
```
   
### 2-5) Word Press 구현
- Word Press 구현하기 위해서는 Data base와 함께 PHP와 Apache가 필요합니다.
- 웹 서버 EC2에 접속해서 PHP와 Apache를 설치해줍니다.
```
sudo yum -y install httpd  //Apache 설치
sudo service httpd start // httpd 실행
sudo systemctl enable httpd.service// 재부팅되어도 자동으로 실행되도록 설정
sudo systemctl status httpd // httpd 상태 확인
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-04-08 13:24:12 UTC; 24s ago
     Docs: man:httpd.service(8)
 Main PID: 1147 (httpd)
   Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─1147 /usr/sbin/httpd -DFOREGROUND
           ├─1148 /usr/sbin/httpd -DFOREGROUND
           ├─1149 /usr/sbin/httpd -DFOREGROUND
           ├─1150 /usr/sbin/httpd -DFOREGROUND
           ├─1151 /usr/sbin/httpd -DFOREGROUND
           └─1152 /usr/sbin/httpd -DFOREGROUND
		   
 sudo amazon-linux-extras install -y php7.4 //PHP 설치
```
- 이제 wget 명령어를 통해 wordpress 압출 파일을 다운받아 인스턴스에 설치해줘야 합니다.
```
sudo wget https://wordpress.org/latest.tar.gz  //wordpress 파일 다운로드
sudo tar -xvzf latest.tar.gz -C /var/www/html //-C 명령어를 통해 압출 해제 폴더를 지정해주고 압축을 해제해준다.
sudo rm -f latest.tar.gz //이제 압축파일은 필요 없으니 삭제 처리
sudo chown -R apache:apache /var/www/html/wordpress //압축해제한 파일들의 소유권을 apache로 변경해준다.

```

- 사실 이 과정들은 인스턴스 생성 후 wordpress를 생성하기 위해서는 무조건 거치는 작업이기 때문에 AWS EC2의 인스턴스 생성 과정에서 유저 스크립트에 입력해 간단하게 처리할 수도 있습니다.
- 만약 이를 스크립트화 한다면 다음과 같습니다.  (위 과정을 거치지않고 바로 설정)
```
#!/bin/sh
yum -y install httpd php mysql
systemctl enable httpd.service
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz -C /var/www/html
rm -f latest.tar.gz
chown -R apache:apache /var/www/html/wordpress
systemctl restart httpd.service
```


- 이제 설치한 wordpress 파일에서 우리가 만든 RDS와 연결이 되도록 설정을 만져줘야합니다.
- 설치된 wordpress 폴더에 들어가서 wp-config-sample 파일을 복사해줍니다.
```
cd /var/www/html/wordpress/
sudo cp wp-config-sample.php wp-config.php //파일 복사
sudo vi wp-config.php //파일 수정
```
- 이제 파일 내부에서 이전에 생성한 RDS 데이터 베이스와 연결이 되도록 설정을 변경해줍니다.

```
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );  //DB 이름

/** Database username */
define( 'DB_USER', 'admin' );  //DB 사용자 이름

/** Database password */
define( 'DB_PASSWORD', 'qwer1234' );   // DB 사용자 패스워드

/** Database hostname */
define( 'DB_HOST', 'wordpress-database.co4sy4ji3quq.ap-northeast-2.rds.amazonaws.com' );    // RDS 데이터 베이스의 패스워드

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );
```
- 이제 apache의 설정파일 /etc/httpd/conf/httpd.conf 에서 DocumentRoot를 wordpress 폴더임 /var/www/html/wordpress로 변경하고, DirectoryIndex를 index.php로 변경합니다.
```
sudo vim /etc/httpd/conf/httpd.conf

119 DocumentRoot "/var/www/html/wordpress"
120
121 #
122 # Relax access to content within /var/www.
123 #
124 <Directory "/var/www/wordpress">
125     AllowOverride None
126     # Allow open access:
127     Require all granted
128 </Directory>
129
130 # Further relax access to the default document root:
131 <Directory "/var/www/html/wordpress">

163 <IfModule dir_module>
164     DirectoryIndex index.php
165 </IfModule>

sudo systemctl restart httpd //wordpress 설치 후 httpd 재시작해주기
```

### 2-6) Auto Scaling 구현
- 이제 Wordpress 설치가 완료된 인스턴스를 이용해 오토 스케일링에 사용할 이미지를 만들어줍니다.
- 먼저 wordpress가 설치된 프라이빗 인스턴스를 이용해 이미지를 생성하고, 인스턴스를 이용해 템플릿 생성하기 탭으로 들어갑니다.
- 이때 이미지를 생성한 후인스턴스는 종료합니다. 이미지를 생성했기 때문에 언제든 다시 만들 수 있습니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FFiDiN%2FbtryPCyY6l1%2FQUGBbK9vzXinFT2x4vNc10%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbLPjWv%2FbtryPbO5ZuV%2FVDldhKApfdFAOwoaeGzK9k%2Fimg.png)
- 템플릿의 이름과 설명, Auto Scaling 지침을 설정합니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbrbVZD%2FbtryOTBbGFj%2FTlKY3atmn4xr6hUcr3EyU1%2Fimg.png)
- 이제 이미지에서 조금전에 Wordpress를 설치해둔 이미지를 선택합니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwAEgQ%2FbtryPA2c3XI%2Ff6AKd4eCVhu2vKbTzaJIOK%2Fimg.png)
- 인스턴스 유형과 키 페어를 선택해주면 이제 네트워크 설정을 진행합니다.
![](https://blog.kakaocdn.net/dn/FrIZ3/btryQqkvI7L/ANJFaKmsDdDz6Qd0f9MlHk/img.png)
- 여기서 서브넷은 시작 템플릿에 포함하지 않고 기존 웹 서버용 보안 그룹을 설정하고 기초 템플릿 생성 요청하면 설정이 끝입니다. 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F1gYzZ%2FbtrySOLDw0t%2FGxRyrSmnKQsiTBgFUk3cC1%2Fimg.png)

- 이제 Auto Scaling 탭으로 이동해 오토 스케일링 그룹을 생성합니다.
- 그룹 이름을 입력하고 조금전 만든 시작 템플릿을 선택합니다.
![](https://blog.kakaocdn.net/dn/0Kd4c/btryQ0MOsXw/tkz57zkvTXpMzF9YXpQTTk/img.png)
- 우리가 사용할 가용영역은 프리티어가 가능한 a와 c 두 가지입니다.
- 두 가용 영역을 선택하고 해당하는 프라이빗 서브넷을 선택합니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcrwxLx%2FbtryQB0NBnx%2F1eT8J5n8rZK8bk1l8OSh51%2Fimg.png)
- 이제 다음 페이지로 넘어와서 로드 밸런싱에서 새로운 로드 밸런서에 연결 옵션을 선택합니다.
- 웹서버 이기 때문에 HTTP, HTTPS를 사용하는 Application Load Balancer로 설정하고 인터넷에서 로드 밸런서를 통해 워드프레스에 접속할 예정이므로 Internet-facing을 선택합니다.
- 서브넷은 외부에서 접속해야 하기때문에 퍼블릭 서브넷을 선택합니다. (프라이빗 선택시 로드밸런서로 http 접속이 안된다.)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbqJYl9%2FbtrySNlEI2l%2FDAk0U46SaGFObGIDlejTIk%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F6DMdx%2FbtryPSBYEDS%2Fr7CT04dLMs9EjnQKo9K2g0%2Fimg.png)
- 원하는 용량 (Desire)는 보통 최소 용량과 최대 용량의 중간값으로 지정합니다.
- Scale in이되면 최소 용량으로, out이 되면 최대 용량으로 늘어납니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDD6e8%2FbtryPSWmlwK%2FC8xxFvVlQlkbSPlrNvlSeK%2Fimg.png)
- 여기서 조정 정책이란 동적으로 조정을 할 것인가 묻는 것으로 선택하지 않으면 자동으로 이뤄지지않고 수동으로 스케일을 변경해줘야 합니다.
- 동적 조정을 선택하여 자동으로 스케일이 조정되도록 설정합니다.  

- 평균 CPU 사용률을 대상 값으로 지정하여 CPU의 사용률이 50%를 넘어가면 스케일을 조정하도록 설정해줍니다. (보통 50~75정도로 설정한다.)  

- 인스턴스 요구 사항은 설정된 수치에 도달했을 때 어느정도 시간을 기다렸다 스케일을 조정할 것인가에 대한 설정합니다. (지연 설정)
- 만약 0으로 설정하면 수치가 도달하는 순간 스케일을 키우게 되지만 만약 설정 수치 근처에서 위아래로 변동이 잦을 경우 스케일을 계속해서 조정하는 상황이 되어 성능이 오히려 저하됩니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcd2F7A%2FbtryPS240Zk%2FZ2xrKazk4ohkyal7vuINU0%2Fimg.png)
- Auto Scaling 그룹 생성 완료.  
- Auto scaling 설정을 통해 원하는 용량으로 설정했던 두 개의 인스턴스와 로스 밸런서, 대상 그룹이 새롭게 생성된 것을 확인할 수 있습니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbbnEvT%2FbtryPqscsZy%2FmYMjNX8O0JRCP35i4xJJ71%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fl8yaM%2FbtryQ0lIs3c%2FosRQJu9NI95hzjs0Dxv0DK%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbdqsKF%2FbtryPSBYWCT%2F7ExCBmsjgRSkzce5V94amk%2Fimg.png)
- 이제 외부에서 Load Balancer에 접속이 가능하게끔 로드밸런서의 http 접속을 허용해줘야합니다.
- http의 모든 접속을 모두 허용하는 보안 그룹을 생성하고, 로드 밸런서 탭의 보안에서 보안 그룹을 편집합니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbACI6x%2FbtryQ1SupRQ%2FrUNVFUpUsHBB03sep3QLKK%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Frh9OB%2FbtrySPcIDUB%2FmptABje9O2GSYnW27bYjTK%2Fimg.png)
- 이제 로드밸런서에서 EC2로 접속이 되도록 EC2의 보안그룹의 인바운드 규칙에  로드밸런서 보안 그룹을 추가합니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FccZX8E%2FbtryP85BM3a%2FPgqnqRC4BrNxIVY1i16FB0%2Fimg.png)


### 2-7) Load Balancer를 통한 접속 확인
- 이제 생성된 로드 밸런서로 http 접속을 시도해 워드 프레스의 정상 작동을 확인합니다.
- 로드밸런서의 DNS 주소를 확인하고 크롬에서 접속 요청을 보냅니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FPoywq%2FbtryQqdIyI2%2FB1m8j9zkRcdP1W3PjKyIgK%2Fimg.png)
- 정상적으로 접속되는 것을 확인할 수 있다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbF3U6p%2FbtryO4bRi3i%2FmslzAhyF08xO21AUiLLWm0%2Fimg.png)
- 워드프레스의 기초 설정을 진행하면 워드프레스 설치가 완료됩니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcLKNgE%2FbtryP2YsVjP%2FE54orUt0ns2VskFKCKTnY0%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbTioAJ%2FbtryQTfGKDh%2F4w8NfKk0Z2kObdfQma6KoK%2Fimg.png)
- 로그인이 성공해서 관리자 페이지로 연결된 모습.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FccdBJX%2FbtryOTutIOl%2FbBKIfIBpHUtsUNFgepJjB1%2Fimg.png)
### 2-8) Cloud Front를 이용한 정적 호스팅
- 이제 Cloud Front를 이용해서 정적 호스팅을 진행합니다.
- AWS의 Cloud Front 탭으로 들어가서 배포 생성을 눌러줍니다
- 원본 도메인으로 로드밸런서를 선택해주고 프로토콜은 뷰어 일치를 선택합니다. 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcnB35i%2FbtryQDde7Qk%2F5FahqspoeY4GfooCEJJrm0%2Fimg.png)
- 클라우트 프론트에 배포가 된 것을 확인할 수 있습니다. 이제 배포된 도메인으로 인터넷 접속을 진행하면 워드 프레스로 접속이 가능합니다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlXzOg%2FbtryQDRSI05%2F1v3GSds0hCoYyHo2gi8FIK%2Fimg.png)
- Cloud Front 이용 전 로딩 시간
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEGY5M%2FbtryQC6uK5F%2FSdaZYn2wt8apcIy3PvuLDk%2Fimg.png)
- Cloud Front 이용 후 로딩 시간
- 584ms에서 443ms로 개선된 로딩 시간을 확인할 수 있다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fczajrv%2FbtryRQJHItL%2FUzx6xuACLKkg5a1JJk2Zm1%2Fimg.png)
