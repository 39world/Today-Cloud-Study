# SSH

-   /etc/ssh/
-   ~/.ssh/
-   ~/.ssh/known\_hosts
-   ~/.ssh/authorized\_keys
-   ~/.ssh/config

A(Client) ---SSH---> B(Server)

## 패스워드 기반의 인증

1.  A는 B의 공개키
    -   `/etc/ssh/ssh_host_<Algorithm>.pub`
    -   `/etc/ssh/ssh_host_<Algorithm>`
        -   **RSA**
        -   DSA
        -   ECDSA
2.  (B 시스템에 최초 접속시)  
    A의 시스템의 사용자에게 B의 공개키(지문) 맞는지 확인?
    -   YES
3.  A의 `~/.ssh/known_hosts` 파일에 B의 공개키 등록
    -   B의 IP/Domain
    -   B의 공개키
4.  ID/PWD 묻는다!(인증)

## 키 기반의 인증

1.  A에서 (인증용)키 쌍을 생성  
    `ssh-keygen`  
    `~/.ssh/id_rsa`: 개인키  
    `~/.ssh/id_rsa.pub`: 공개키
2.  B에 A의 공개키 등록
    -   B 시스템의 `~/.ssh/authorized_keys` : 클라이언트의 공개키 등록
    -   EC2(클라우드 인스턴스): A에서 지정한 A의 공개키 등록
    -   BM, VM: `ssh-copy-id` 명령으로 등록
        -   B에 패스워드 인증 방법이 화성화 되어 있어야 함
3.  (B 시스템에 최초 접속시)  
    A의 시스템의 사용자에게 B의 공개키(지문) 맞는지 확인?
    -   YES
4.  A의 `~/.ssh/known_hosts` 파일에 B의 공개키 등록
    -   B의 IP/Domain
    -   B의 공개키
5.  A의 개인키로 인증

기본 로그인 사용자

-   Amazon Linux: ec2-user
-   Ubuntu: ubuntu
-   Debian: debian
-   Centos: centos
-   RHEL: cloud-user
-   vagrant: vagrant
-   ...

### 서버의 SSH 공개키 지문 확인

```
ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
```

### 서버의 SSH 공개키 미리 확인

```
ssh-keyscan 192.168.100.11
```

```
ssh-keyscan -t <rsa|ecdsa> 192.168.100.11
```

지문 확인

```
ssh-keyscan -t ecdsa 192.168.100.11 | ssh-keygen -l -f -
```

미리 서버의 공개키 등록

```
ssh-keyscan -t ecdsa 192.168.100.11 >> ~/.ssh/known_hosts
```

\=> 연결 불가. 서버의 기본 설정은 키 인증만 가능하고 암호 인증은 불가능하게 설정된다.  
하지만 키를 등록하기 위해서는 암호 인증이 가능해야 한다.  
따라서 잠깐 서버의 설정 파일에서 패스워드 인증을 가능하게끔 설정해준다.

-   `/etc/ssh/ssh_config`: 클라이언트 설정 파일
-   `/etc/ssh/sshd_config`: 서버의 설정 파일  
    여기서 관리자 권한이 없으면 파일 내용이 보이지 않는다. sudo vi /etc/ssh/ssh\_config 를 입력해주자.

`/etc/ssh/sshd_config`

```
PasswordAuthentication no # 패스워드 인증
GSSAPIAuthentication yes # 키 인증
```

## 키 기반 인증 구성

먼저 클라이언트에서 ssh-keygen을 사용해 키를 만들어준다.  
Client

```
[vagrant@controller ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:dHexyEcoqP+fYwK9yxU3hvThaOksfBgE4RVxMZFQ0VA vagrant@controller
The key's randomart image is:
+---[RSA 2048]----+
|        oo==OXE  |
|       ..o.oo+.o |
|       .o o.= =  |
|      .. o o O . |
|       .S.. * *  |
|        o..* + . |
|         o+.=    |
|         .++o.   |
|          o=o.   |
+----[SHA256]-----+
[vagrant@controller ~]
```

실습에서는 passphrase를 그냥 엔터로 넘어갔지만, 실제 서버에서는 절대 공백으로 하지 않고 입력해줘야한다!  
이후에 클라이언트에서 서버로 키를 복사해준다.

```
[vagrant@controller ~]$ ssh-copy-id 192.168.100.11
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@192.168.100.11's password:
Permission denied, please try again.
vagrant@192.168.100.11's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.100.11'"
and check to make sure that only the key(s) you wanted were added.

[vagrant@controller ~]$
```

서버에서 새롭게 추가된 키를 확인할 수 있다.

```
[vagrant@node2 ~]$ cat ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDL/VF3yVHaSFtkmWgTeQ4pzLkWYvx1X4sKr2VKm7gI+1Y5RdZkYPJ61IE1WDO6WvebyClobTzy6rfS5u5NSX/bgNqktthbOl+4capR9tVE3hxpp5dp9IVtCQqvGT5WUqa0IgTEBuYcvgoyHwynui7RFMpsQcjjpK8hNfTiis55oYMePN3U/CmUczVq83+9ko9Pe7dzhCpVTFrpS4BGIr47gYI0uxLQEghJOhzm/Jx43eWSngIPiBHSdQQ7WvtehJLs8CwLj0pY2HB1COoQvBDzMd5CoLHXBl+eP3/MJT1Bdn12T9fex8ZQiqHc7aOxoPvISfT0PNXjhP/BUaDS9PjZ vagrant
##서버의 키 밖에 없던 파일에 클라이언트 공개키가 복사됐다.
[vagrant@node2 ~]$ cat ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDL/VF3yVHaSFtkmWgTeQ4pzLkWYvx1X4sKr2VKm7gI+1Y5RdZkYPJ61IE1WDO6WvebyClobTzy6rfS5u5NSX/bgNqktthbOl+4capR9tVE3hxpp5dp9IVtCQqvGT5WUqa0IgTEBuYcvgoyHwynui7RFMpsQcjjpK8hNfTiis55oYMePN3U/CmUczVq83+9ko9Pe7dzhCpVTFrpS4BGIr47gYI0uxLQEghJOhzm/Jx43eWSngIPiBHSdQQ7WvtehJLs8CwLj0pY2HB1COoQvBDzMd5CoLHXBl+eP3/MJT1Bdn12T9fex8ZQiqHc7aOxoPvISfT0PNXjhP/BUaDS9PjZ vagrant
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCjlmPjTSpp3HR+0Z9O8QAUx6kHMu2s3ULRBgx94BBn8cpvjlxV4Q1fy4bcZula3TBIVlJaETwZYHfdSHgP6W7hoFQLoHODfgPTeSY8n8LtICH9wsVMtGp1PgoY1iX0LOU1M9tBMadtfYzJsTwXwLG8LL1rExu4V6TcytPS950dREuKG8unhynmUIgOcBH6IYmH1CK6D6kOtn9zS59TPq1Xe7Vfuhlft289f8R6H+lVJFpIuX/Xrh6XtOPWOrrI9JC4rJWSiqMlwwbNrfi2gD49p4bFNiftGWKSCPSWSdI+Di4SNq60XmmMjMXpEMIs127QXJgyq4I/jgJhSAwbN09H vagrant@controller
```

```
#이제 연결 요청을 해도 패스워드를 물어보지 않는다.
[vagrant@controller ~]$ ssh 192.168.100.11
Last failed login: Tue Apr 12 07:59:54 UTC 2022 from 192.168.100.10 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Tue Apr 12 07:49:36 2022 from 192.168.100.10
[vagrant@node1 ~]$
```

이때 현재 유저의 이름을 그대로 가져다가 연결 요청을 하기 때문에 만약 내가 root 유저로 요청한다면 서버의 루트 유저의 패스워드가 필요하다. vagrant 유저로 접속하고 싶은데 현재 루트 유저 상태라면 vagrant를 붙여줘야한다.

```
[root@controller ~]# ssh 192.168.100.11
root@192.168.100.11's password:
[root@controller ~]# ssh vagrant@192.168.100.11
vagrant@192.168.100.11's password:
```

```
ssh-keyscan -t ecdsa 192.168.100.11 >> ~/.ssh/known_hosts
ssh-keyscan -t ecdsa 192.168.100.12 >> ~/.ssh/known_hosts
```

```
ssh-copy-id vagrant@192.168.100.11
ssh-copy-id vagrant@192.168.100.12
```

### Windows --> Vagrant SSH 접근

```
vagrant ssh <VM_NAME>
```

```
ssh -i .\.vagrant\machines\controller\virtualbox\private_key 192.168.100.10
```

```
ssh -i .\.vagrant\machines\node1\virtualbox\private_key vagrant@192.168.100.11
```

```
 ssh -i .\.vagrant\machines\node2\virtualbox\private_key vagrant@192.168.100.12
```

### SSH 클라이언트 설정 파일

`~/.ssh/config`

```
Host controller
    HostName 192.168.100.10
    User vagrant
    IdentityFile C:\Users\Playdata\vagrant\ansible\.vagrant\machines\controller\virtualbox\private_key

Host node1
    HostName 192.168.100.11
    User vagrant
    IdentityFile C:\Users\Playdata\vagrant\ansible\.vagrant\machines\node1\virtualbox\private_key

Host node2
    HostName 192.168.100.12
    User vagrant
    IdentityFile C:\Users\Playdata\vagrant\ansible\.vagrant\machines\node2\virtualbox\private_key
```