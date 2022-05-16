# K8s 설치

쿠버네티스 설치에는 다양한 방법이 존재한다. 여기서는 kubeadm 설치 방법을 서술한다.

-   **Kubeadm**
-   Kubespray (Kubeadm + Ansible)
-   Kops
-   Docker Desktop - Kubernetes
-   minikube

## Kubeadm

1.22.8 -> 1.22.8로 진행 후 업그레이드를 해본다

### kubeadm, kubectl, kubelet 도구 설치

apt 패키지 업데이트 및 apt 레포지토리에 필요한 패키지 설치

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

구글 클라우드의 서명키를 가져온다.

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

쿠버네티스 api 레포를 추가해준다.

```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```
sudo apt-get update
```

쿠버네티스를 설치해준다.

```
sudo apt-get install kubeadm=1.22.8-00 kubelet=1.22.8-00 kubectl=1.22.8-00 -y
```

kubeamd, kubelet, kubectl의 버전을 고정하도로 설정한다. 이 설정을 하지 않으면 항상 최신 버전을 설치하게 되면서 오류가 발생할 수 있다.

```
sudo apt-mark hold kubelet kubeadm kubectl
```

### cgroup driver 오류

docker info를 살펴보면 Cgourp Driver가 cgroupfs로 설정되어있다. 도커는 cgroupfs 방식을 선호하고 쿠버네티스는 systemd 방식을 선호한다.  
쿠버네티스를 v1.22로 올라가면서 기존에는 경고 수준으로 처리하던 cgroup diver의 default 값을 systemd로 설정하도록 알려주고 있다.  
따라서 이 설정을 변경해준다

-   cgroupfs 와 systemd  
    systemd와 cgroupfs는 모두 cgroup을 관리하기위해 동작하지만 접근법에서 차이가 있다.  
    cgourpfs는 cgroup 디렉토리 하위에서 리소스를 직접 매핑하는 구조를 가진다.  
    반면 systemd는 프로세스가 사용하는 자원을 관리하기 위해 slice > scope > service 단위의 계층 구조를 만들어 각 단위에 자원을 할당한다.  
    이런 계층 구조는 systemd-cgls 명령어가 표현해주는 구조 혹은 /sys/fs/cgorup/systemd 하위의 파일 시스팀 계층 구조를 통해 확인할 수 있따.

Cgroup Driver 설정을 보는 방법

```
docker info | grep 'Cgroup Driver'

 Cgroup Driver: cgroupfs
```

설정을 변경하기위해 `/etc/docker/daemon.json` 파일을 생성해준다.

```
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

이후 도커를 재시작해준다.

```
sudo systemctl restart docker
```

```
docker info | grep 'Cgroup Driver'

 Cgroup Driver: systemd
```

```
sudo systemctl daemon-reload && sudo systemctl restart kubelet
```

### k8s 클러스터 생성

만약 진행하다 `kubeadm init` 실패 시 reset을 해준다.

```
sudo kubeadm reset
```

컨트롤 플레인의 엔드포인트(VM의 주소)와 네트워크 대역 apiserver의 주소를 설정해주고 init을 해준다.

```
sudo kubeadm init --control-plane-endpoint 192.168.100.100 --pod-network-cidr 172.16.0.0/16 --apiserver-advertise-address 192.168.100.100
```

출력 결과를 통해 설정된 결과를 볼 수 있다.

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

`get nodes` 명령어를 입력하면 현재 연결된 노드를 확인할 수 있다.

```
kubectl get nodes

NAME     STATUS     ROLES                  AGE   VERSION
docker   **NotReady**   control-plane,master   14m   v1.22.8
```

### Calico Network Add-on

현재 상태를 보면 상태가 NotReady인 것을 확인할 수 있다. Calico를 이용해 Network 설정을 통해 kubeadm init 명령에서 넘어갔던 CNI 설정을 완료해준다.

```
kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
```

```
curl https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml -O
```

`custom-resources.yaml` 파일을 수정하여 kubeadm init 명령에서 사용한 cidr과 동일하게 설정해준다.

```
...
      cidr: 172.16.0.0/16
...
```

calio 설치

```
kubectl create -f custom-resources.yaml
```

### 클러스터 상태 확인

```
kubectl get pods -A   

NAMESPACE          NAME                                       ...
calico-apiserver   calico-apiserver-c9565f67b-2p29k           ...
calico-apiserver   calico-apiserver-c9565f67b-slthl           ...
calico-system      calico-kube-controllers-5d74cd74bc-sg7dn   ...
calico-system      calico-node-tgxks                          ...
calico-system      calico-typha-7447fdc844-txrdb              ...
kube-system        coredns-78fcd69978-4ztkq                   ...
kube-system        coredns-78fcd69978-jpwxx                   ...
kube-system        etcd-docker                                ...
kube-system        kube-apiserver-docker                      ...
kube-system        kube-controller-manager-docker             ...
kube-system        kube-proxy-5st98                           ...
kube-system        kube-scheduler-docker                      ...
tigera-operator    tigera-operator-7cf4df8fc7-kx87z           ...
```

```
kubectl get nodes

NAME     STATUS   ROLES                  AGE   VERSION
docker   **Ready**    control-plane,master   30m   v1.22.8
```

control plane과 node를 하나에 구성했을 경우 컨테이너가 실행하지 못하도록 격리된 상태를 풀어줘야한다.

```
kubectl taint node docker node-role.kubernetes.io/master-
```

### 간단한 예제

docker를 실습할 때 만들어둔 myweb 서비스를 배포해보는 과정

먼저 이미지를 받아온다.

-   replicaset : 컨테이너를 복사하는 것
    
    ```
    kubectl create deployment myweb --image=ghcr.io/c1t1d0s7/go-myweb
    ```
    

```
kubectl get deployments,replicasets,pods

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myweb   1/1     1            1           4m40s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/myweb-97dbf5749   1         1         1       4m40s

NAME                        READY   STATUS    RESTARTS   AGE
pod/myweb-97dbf5749-8tq2l   1/1     **Running**   0          4m40s
```

항상 이미지를 받고 pod가 Running 상태가 되어있는지 확인하고 진행한다.  
이제 pod를 외부로 노출시켜준다. 여기서 myweb-svc는 로드밸런서 역할을 한다.

-   deployment : seplicaset을 복제하는 것
    
    ```
    kubectl expose deployment myweb --port=80 --protocol=TCP --target-port=8080 --name myweb-svc --type=NodePort
    ```
    

```
kubectl get services

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        40m
myweb-svc    NodePort    10.96.114.201   <none>        80:**31891**/TCP   5s
```

배포된 서비스를 확인한다

```
curl 192.168.100.100:**31891**

Hello World!
myweb-97dbf5749-8tq2l
```

만약 pod의 개수를 늘리고 싶다면 scale을 조정하면 된다.

```
kubectl scale deployment myweb --replicas=3
```

늘어난 pod 확인

```
kubectl get pods

myweb-97dbf5749-8tq2l   1/1     Running       0          12m
myweb-97dbf5749-9bm8l   1/1     Running       0          3m13s
myweb-97dbf5749-n29m2   1/1     Running       0          3m13s
```

```
curl 192.168.100.100:**31891**
```

삭제 방법

```
kubectl delete service myweb-svc
kubectl delete deployment myweb
```

쿠버네티스에는 어렵게 설치하는 방법도 있다.  
쿠버네티스의 동작 원리를 알아가며 하나씩 진행하는 방식으로 도움이 된다.  
`https://github.com/kelseyhightower/kubernetes-the-hard-way`