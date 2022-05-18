# Kubernetes Objects
> https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/kubernetes-objects/

Object는 리소스를 뜻한다. 오브젝트를 만드는 방법에는 `명령 방식`과 `yaml 방식`이 있는데  보통 `yaml 방식`을  주로 사용한다.

kubectl -> 클라이언트 도구
어떤 명령을 내리면 API를 통해 API Server에 요청이 간다.
### 오브젝트 명세(Spec)와 상태(Status)
명세(Spec)을 우리가 오브젝트에 설정해주고 상태(Status)는 현재 상태를 k8s이 알려준다.  

- 오브젝트 파일 구성

```yaml
apiVersion: apps/v1  #필수 지원하는 오브젝트의 버전
kind: Deployment     #필수 Object의 종류
metadata:            #필수 Data를 설명하는 정보 (이름, 네임스페이스, 레이블, 어노테이션)
  name: nginx-deployment
spec:                #필수(거의 대부분 사용) 오브젝트에 대한 선언
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```
리소스 정보 확인
```
kubectl api-resources

NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints

```
```
kubectl get <오브젝트 종류>
kubectl get nodes
kubectl get services
kubectl get pods
```
### 오브젝트의 버전
- Stable : 안정화된 버전 ex)vX  v1, v2..
- Beta : 충분한 오류 검증. 기능 변경이 있을 수 있음. v1betaX, v2betaX
- Alpha :  현재 개발중인 API. v1alphaX, v2alphaX

개발 순서는 Dev -> Alpha -> Beta -> Stable

- 해당하는 오브젝트의 정보를 자세하게 볼 수 있다 
KIND, VERSION, DESCRIPTION, FIELD 등
Status 정보는 쿠버네티스가 제공하는 정보로 Read Only가 붙어있다.
계속해서 레이어를 낮춰가며 정보를 자세하게 볼 수 있다.

```
kubectl explain <오브젝트 종류>
kubectl explain pods
kubectl explain nodes

kubectl explain pods.metadata
kubectl explain pods.spec.containers.ports
```

### 오브젝트 관리
- 명령형 커맨드 : kubectl 명령으로만 구성
```
kubectl create
kubectl run
kubectl expose
```
- 명령형 오브젝트 구성 : yaml 파일을 순서대로 하나씩 실행
```
kubectl create -f a.yaml
kubectl replace -f a.yaml
kubectl delete -f a.yaml
kubectl expose -f a.yaml
```
- 선언형 오브젝트 구성 : yaml 파일의 모음을 한번에 실행
```
kubectl create -f resources/
kubectl run -f resources/
kubectl expose -f resources/
```

```
kubectl run myweb --image httpd
pod/myweb created

kubectl get pods
NAME    READY   STATUS              RESTARTS   AGE
myweb   0/1     ContainerCreating   0          10s

kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
myweb   1/1     Running   0          18s   10.233.96.1   node2   <none>           <none>

kubectl get pods -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
.
.
.
```


#### Object의 종류
- Label/LabelSelector

- Workload
	- Pod
	- Controller
		- ReplicationController
		- ReplicaSets
		- DaemonSets
		- Jobs
		- CronJobs
		- Deployments
		- StatefulSets
		- HorizontalPodAutoscaler

- Network
	- Service
	- Endpoints
	- Ingress

- Storage
	- PersistentVolume
	- PersistentVolumeClaim
	- ConfigMap
	- Secret

- Authentication
	- ServiceAccount
	- RBAC
		- Role
		- ClusterRole
		- RoleBinding
		- ClusterRoleBinding

- Resource Isolation
	- Namespaces

- Resource Limits
	- Limits
	- Requests
	- ResourceQuota
	- LimitRange

- Scheduling
	- NodeName
	- NodeSelector
	- Affinity
		- Node Affinity
		- Pod Affinity
		- Pod Anti Affinity
	- Taints/Tolerations
	- Drain/Cordon
