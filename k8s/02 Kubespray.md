# Kubespray

이번엔 kubeadm이 아닌 Kubespray를 설치해본다.  
kubeadm은 수동으로 모듈을 하나씩 설치하지만 kubespray는 Ansible 기반의 배포툴로 매우 간단하다.

> [https://kubernetes.io/ko/docs/setup/production-environment/tools/kubespray/](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubespray/)  
> [https://kubespray.io/#/](https://kubespray.io/#/)  
> [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)

## 구성 목표

Control Plane 1  
Work Node 3(1 Control Plan + 2 Woker Node)

CPU: 2, Memory 3GB

## 설치 순서

-   VM 구축
-   Inventory 서버에 모든 서버의 SSH 키 복사
-   kubespray 다운로드
-   Ansible 등 요구 패키지 설치
-   인벤토리 설정
-   변수 설정
-   Ansible-playbook 실행
-   확인

### VM 구성

Ansible을 사용하기 위해 ssh 키를 배포해야하는데, 이를 위해 패스워드 인증이 가능하도록 변경하는 내용과 설치 속도를 빠르게하기 위해 저장소 변경 내용이 포함되어있다.

`~/vagrant/k8s`

```
Vagrant.configure("2") do |config|
    # Define VM
    config.vm.define "k8s-node1" do |ubuntu|
        ubuntu.vm.box = "ubuntu/focal64"
        ubuntu.vm.hostname = "k8s-node1"
        ubuntu.vm.network "private_network", ip: "192.168.100.100"
        ubuntu.vm.provider "virtualbox" do |vb|
            vb.name = "k8s-node1"
            vb.cpus = 2
            vb.memory = 3000
        end
    end
    config.vm.define "k8s-node2" do |ubuntu|
        ubuntu.vm.box = "ubuntu/focal64"
        ubuntu.vm.hostname = "k8s-node2"
        ubuntu.vm.network "private_network", ip: "192.168.100.101"
        ubuntu.vm.provider "virtualbox" do |vb|
            vb.name = "k8s-node2"
            vb.cpus = 2
            vb.memory = 3000
        end
    end
    config.vm.define "k8s-node3" do |ubuntu|
        ubuntu.vm.box = "ubuntu/focal64"
        ubuntu.vm.hostname = "k8s-node3"
        ubuntu.vm.network "private_network", ip: "192.168.100.102"
        ubuntu.vm.provider "virtualbox" do |vb|
            vb.name = "k8s-node3"
            vb.cpus = 2
            vb.memory = 3000
        end
    end

    config.vm.provision "shell", inline: <<-SHELL
      sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
      sed -i 's/archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
      sed -i 's/security.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
      systemctl restart ssh
    SHELL
end
```

### 1\. SSH 키 생성 및 복사

```
ssh-keygen
```

```
ssh-copy-id vagrant@192.168.100.100
ssh-copy-id vagrant@192.168.100.101
ssh-copy-id vagrant@192.168.100.102
```

### 2\. kubespray 소스 다운로드

```
cd ~
```

```
git clone -b v2.18.1 https://github.com/kubernetes-sigs/kubespray.git
```

```
cd kubespray
```

### 3\. ansible, netaddr, jinja 등 패키지 설치

다운받은 kubespray 폴더에 들어가보면 `requirement.txt`로 요구되는 패키지들을 정리해놓았다. pip 명령어를 통해 이를 한 번에 설치해준다.

```
sudo apt update
sudo apt install python3-pip -y
```

```
sudo pip3 install -r requirments.txt
```

### 4\. 인벤토리 구성

```
cp -rpf inventory/sample/ inventory/mycluster
```

`inventory/mycluster/inventory.ini`

```
[all]
node1 ansible_host=192.168.100.100 ip=192.168.100.100
node2 ansible_host=192.168.100.101 ip=192.168.100.101
node3 ansible_host=192.168.100.102 ip=192.168.100.102

[kube_control_plane]
node1

[etcd]
node1

[kube_node]
node1
node2
node3

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

### 5\. 변수 설정

`inventory/mycluster/group_vars`

### 6\. 플레이북 실행

```
ansible all -m ping -i inventory/mycluster/inventory.ini
```

```
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b 
```

### 7\. 결과 확인

api server와 통신하기 위해 인증 정보를 가져와준다.

```
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown vagrant:vagrant ~/.kube/config
```

결과 확인

```
kubectl get nodes
NAME    STATUS   ROLES                  AGE   VERSION
node1   Ready    control-plane,master   50m   v1.22.8
node2   Ready    <none>                 49m   v1.22.8
node3   Ready    <none>                 49m   v1.22.8
```

```
kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS    RESTARTS      AGE
kube-system   calico-kube-controllers-5788f6558-7c5lt   1/1     Running   2 (15m ago)   48m
kube-system   calico-node-6nlhb                         1/1     Running   1 (14m ago)   48m
kube-system   calico-node-jndtj                         1/1     Running   1 (15m ago)   48m
kube-system   calico-node-rq4tm                         1/1     Running   1 (15m ago)   48m
kube-system   coredns-8474476ff8-sblxx                  1/1     Running   1 (14m ago)   47m
kube-system   coredns-8474476ff8-smwqh                  1/1     Running   1 (15m ago)   47m
kube-system   dns-autoscaler-5ffdc7f89d-2pqrg           1/1     Running   1 (15m ago)   47m
kube-system   kube-apiserver-node1                      1/1     Running   2 (15m ago)   50m
kube-system   kube-controller-manager-node1             1/1     Running   3 (15m ago)   50m
kube-system   kube-proxy-26jrg                          1/1     Running   0             4m34s
kube-system   kube-proxy-5m5jg                          1/1     Running   0             4m34s
kube-system   kube-proxy-wbnw5                          1/1     Running   0             4m34s
kube-system   kube-scheduler-node1                      1/1     Running   3 (15m ago)   50m
kube-system   nginx-proxy-node2                         1/1     Running   1 (15m ago)   49m
kube-system   nginx-proxy-node3                         1/1     Running   1 (14m ago)   49m
kube-system   nodelocaldns-8jlgg                        1/1     Running   2 (25s ago)   47m
kube-system   nodelocaldns-pv42f                        1/1     Running   1 (14m ago)   47m
kube-system   nodelocaldns-xw49g                        1/1     Running   1 (15m ago)   47m
```