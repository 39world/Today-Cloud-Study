### Ansible 개요

-   Ansible
    -   Ansible 아키택쳐
        -   설정
        -   모듈
        -   인벤토리
    -   Ad-hoc
    -   Playbook
        -   YAML
        -   변수, 조건문, 반복문
        -   포함, 역할 ...
    -   Vault : 암호화
    -   AWX : 웹 인터페이스 (모니터링)
    -   Git, GitHub

### IaC 란?

Infrastructure as Code : 코드형 인프라

장점

-   비용 절감
-   빠른 속도
-   안정성
-   재사용성
-   버전 관리

### 구성 관리 / 배포

구성 관리 : Configuration Management  
패키지 설치, 설정 파일, 파일 복사 ...  
배포 : Provisioning  
리소스 생성, 변경, 삭제 등 관리

구성 관리 도구 : Ansible, Chef, Puppet, SaltStack ..  
배포 : Terraform, Vagrant, AWS CloudFormation

### 가변 인프라 / 불변 인프라

가변 : Mutable  
Ansible  
불변 : Immutable  
Terraform

### 절차적 / 선언전

절차적 : 순서 O  
Ansible  
선언적 : 순서 X  
Terraform, Kubernetes

### 마스터 및 에이전트 유무

마스터, 에이전트 : Chef, Puppet, SaltStack

-   관리형 서버, 프로그램 등 세팅이 필요하다.

### Ansible 설치

ansible 관련 파일을 찾아보면 지금 OS가 CentOS라서 관련 Ansible 레포지토리들이 나온다.

```
sudo yum search ansible
================================================= N/S matched: ansible =================================================
ansible-collection-microsoft-sql.noarch : The Ansible collection for Microsoft SQL Server management
centos-release-ansible-27.noarch : Ansible 2.7 packages from the CentOS ConfigManagement SIG repository
centos-release-ansible-28.noarch : Ansible 2.8 packages from the CentOS ConfigManagement SIG repository
centos-release-ansible-29.noarch : Ansible 2.9 packages from the CentOS ConfigManagement SIG repository
centos-release-ansible26.noarch : Ansible 2.6 packages from the CentOS ConfigManagement SIG repository
scap-security-guide-rule-playbooks.noarch : Ansible playbooks per each rule.
```

Ansible 설치

```
sudo yum install centos-release-ansible-29 -y  
sudo yum install ansible -y
```

설치 확인

```
[vagrant@controller ~]$ ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/vagrant/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr  2 2020, 13:16:51) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
[vagrant@controller ~]$
```

인벤토리

\*\*사용전 ssh 연결 설정이 모두 끝난 상태여야 가능하다.

```
vi inventory.ini
```

```
192.168.100.11  ##현재 Vagrant의 node1,2 IP
192.168.100.12
```

yum 모듈의 5가지 state  
absent, installed, latest, present, removed  
간단하게 absent, removed는 삭제, installed, latest, present는 설치라고 생각하면 된다.

요청을 통해 변경된 점이 있다면 CHANGED, 요청은 성공했지만 변경 내역이 없다면 SUCCESS가 나온다. 

```
 ansible 192.168.100.11 -i inventory.ini -m yum -a "name=httpd state=present" -b
 192.168.100.11 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "installed": [
            "httpd"
        ]
    },
    "msg":  ......
    .
    ..
    
    [vagrant@controller ~]$ ansible 192.168.100.11 -i inventory.ini -m yum -a "name=httpd state=present" -b
192.168.100.11 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "",
    "rc": 0,
    "results": [
        "httpd-2.4.6-97.el7.centos.5.x86_64 providing httpd is already installed"
    ]
}
```

-   ansible : ad-hoc 명령
-   192.168.100.11 : 관리 노드 (인벤토리 파일에 정의되어 있어야 한다)
-   \-i inventory.ini : 인벤토리 파일명
-   m yum : 모듈 이름
-   \-a : 모듈 파라미터
-   \-b : 관리자 권한 취득(become -> sudo)

```
ansible 192.168.100.11 -i inventory.ini -m service -a "name=httpd state=started enabled=yes" -b
```

playbook:  
\*\*\* yaml은 띄어쓰기에 매우 민감하기 때문에 주의할 것.(tab 사용하지 말 것.)  
apache\_install.yaml

```
- hosts:192.168.100.11
  tasks:
  - yum:
      name: httpd
      state: present
  - service:
      name: httpd
      enabled: yes
      state: started
```

실행

```
ansible-playbook -i inventory.ini apache_install.yaml -b
```