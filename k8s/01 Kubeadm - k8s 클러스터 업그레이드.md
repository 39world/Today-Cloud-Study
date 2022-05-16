# k8s 클러스터 업그레이드

### 업그레이드 순서

쿠버네티스의 버전차이(skew) 정책에 따라 업그레이드 순서를 지키지 않으면 작동에 오류가 생길 수 있다.  
버전 차이 정책을 자세히 살펴보고 업그레이드하고자 하는 쿠버네티스의 구성 요소와 지원하는 버전 차이를 확인 후 진행하도록 한다.

보통 다음과 같은 순서로 진행된다.

1.  kube-apiserver
2.  kube-controller-manager, kube-cloud-controller-manage, kube-scheduler
3.  kubelet(Control Plane -> Worker Node)
4.  kube-proxy(Control Plane -> Worker Node)

Control Plane(api -> cm, ccm, sched -> let,proxy) --> Work Node(let, proxy)

## kubeadm 업그레이드 상세 방법

1.  Control Plane의 kubeadm 업그레이드
2.  Control Plane의 kubeadm으로 api, cm, sched 업그레이드
3.  Control Plane의 kubelet, kubectl 업그레이드
4.  Work Node의 kubeadm 업그레이드
5.  Work Node의 kubeadm으로 업그레이드
6.  Work Node의 kubelet, kubectl 업그레이드

-   현재 CNI 업그레이드는 진행하지 않았지만 필요시 kubeadm upgrade apply 이후 CNI 업그레이드를 진행해준다.

### Control Plane

Control Plane 작업이 Wordker Node 작업보다 우선 순위에 온다.

최신 버전으로 교체되는 것을 막기위해 hold해놓은 설정을 풀어준다.

```
sudo apt-mark unhold kubeadm
```

```
sudo apt update
```

```
sudo apt upgrade kubeadm=1.22.9-00 -y
```

설치된 버전을 확인해본다.

```
kubeadm version
```

다시 버전을 잠금해준다.

```
sudo apt-mark hold kubeadm
```

```
sudo kubeadm upgrade plan
```

```
sudo kubeadm upgrade apply v1.22.9
```

kubelet, kubectl도 똑같이 진행해준다.

```
sudo apt-mark unhold kubelet kubectl
```

```
sudo apt upgrade kubectl=1.22.9-00 kubelet=1.22.9-00 -y
```

```
sudo apt-mark hold kubelet kubectl
```

버전 확인

```
kubelet --version
kubectl version
```

-   drain 작업  
    본래 drain 작업을 진행해야 하지만 현재 기본 구성 이외에 추가적인 pod를 구성하지 않았기때문에 생략가능

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

-   uncordon 작업  
    drain 작업이 없으므로 생략 가능

```
systemctl status kubelet
```

### Worker Node

```
sudo apt-mark unhold kubeadm
```

```
sudo apt update
```

```
sudo apt upgrade kubeadm=1.22.9-00 -y
```

```
kubeadm version
```

```
sudo apt-mark hold kubeadm
```


```
sudo kubeadm upgrade node
```

> drain 작업

```
sudo apt-mark unhold kubelet kubectl
```

```
sudo apt upgrade kubectl=1.22.9-00 kubelet=1.22.9-00 -y
```

```
sudo apt-mark hold kubelet kubectl
```

```
kubelet --version
kubectl version
```

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

> uncordon 작업

#### 패키지 저장소 변경

업그레이드나 kubespay등 다운로드 받을 파일이 많아 속도가 너무 느리다면 패키지 저장소를 변경해 시간을 단축할 수 있다.  
Ubuntu 패키지 저장소 변경

```
sed -i 's/security.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
sed -i 's/archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
sudo apt update
```

> [https://kubernetes.io/ko/releases/version-skew-policy/](https://kubernetes.io/ko/releases/version-skew-policy/)