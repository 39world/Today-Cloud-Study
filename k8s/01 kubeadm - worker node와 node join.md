## Worker Node 추가

Worker Node를 새롭게 만들어 Control Plane과 조인하는 방법에 대해 서술한다.

### Docker 설치

먼저 도커를 설치해준다.

```
sudo apt-get update
```

```
sudo apt-get install 
    ca-certificates 
    curl 
    gnupg 
    lsb-release
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
echo 
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu 
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt-get update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Cgroup Driver 설정

```
docker info | grep 'Cgroup Driver'

 Cgroup Driver: cgroupfs
```

`/etc/docker/daemon.json`

```
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

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

```
sudo usermod -aG docker vagrant
```

재접속

### kubeadm, kubelet, kubectl 설치

```
sudo apt-get install -y apt-transport-https ca-certificates curl
```

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```
sudo apt-get update
```

```
sudo apt-get install kubeadm=1.22.8-00 kubelet=1.22.8-00 kubectl=1.22.8-00 -y
```

### K8s Cluster Join

연결은 다음과 같은 형식으로 진행된다.

```
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

여기서 필요한 토큰 정보와 hash 값은 Control Plane에서 확인해야한다.  
Control plane에서 토큰을 확인한다.

토큰 목록 확인하기

```
kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
k6ojsd.ias2fsttqwbo8j52   11h         2022-05-16T16:22:05Z   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
```

토큰 생성하기

```
kubeadm token create
```

토큰의 해시값 확인하기

```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
ssl dgst>    openssl dgst -sha256 -hex | sed 's/^.* //'
e6ff97485e0619b073d900051847b46cbe8d68f312d5f5e92bdc52c6ef660754
```

이제 해당하는 값을 가지고 join을 요청한다.

```
sudo kubeadm join --token 69bz9a.jd5a3qmzlhb66iua 192.168.100.100:6443 
--discovery-token-ca-cert-hash sha256:73f2901d915c6fe5a5a37a6c1c4c1aa73e36da2c3619e36c16d2387a856bc840
```

결과 확인

```
kubectl get nodes
NAME     STATUS   ROLES                  AGE     VERSION
docker   Ready    control-plane,master   1h     v1.22.8
worker   Ready    <none>                 1m30s   v1.22.8
```