## Workload-Controller


### Replication Controller 
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb-rc
spec:
  replicas: 3
  selector:   # 항상 라벨 중 하나를 포함해야한다.
    app: web
# Pod Configure
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: myweb
          image: httpd
          ports:
            - containerPort: 8080
              protocol: TCP
```
yaml 파일을 사용해 생성
```
kubectl create -f myweb-rc.yaml
replicationcontroller/myweb-rc created
```
생성 결과 확인
```
kubectl get pods               
NAME             READY   STATUS              RESTARTS   AGE
myweb-rc-hrfb7   0/1     ContainerCreating   0          10s
myweb-rc-kpfgs   1/1     Running             0          10s
myweb-rc-v9kfh   0/1     ContainerCreating   0          10s
kubectl get rc  
NAME       DESIRED   CURRENT   READY   AGE
myweb-rc   3         3         1       14s
```
### RC 스케일 변경
명령형 커맨드
```
kubectl scale rc rcName --replicas=원하는 숫자

kubectl scale rc myweb-rc --replicas=5
```

yaml 파일을 변경하고 적용할 때 그대로 create 명령어를 사용하면 에러가 발생한다.
그럴땐 replace를 사용해야한다.
```
kubectl replace -f myweb-rc.yaml
```
만약 파일을 사용하지 않고 RC를 만들었거나, 파일을 수정하지않고 RC를 수정하려면 kubectl patch를 사용하면 된다.
patch  명령어는 모든 정보를 다시 제공하는 것이 아니라 변경하려는 특정 부분만 정보를 제공하면 된다. 이때 Json 형식에 따른다.
```
kubectl patch rc myweb-rc -p '{"spec": {"replicas":3}}'
```
만약 json 형식의 파일을 만들어서 패치를 진행하려면 json 형태로 파일을 만들고 적용해준다.
```
{
	"spec":
		{"replicas": 3}
}
```
```
kubectl patch rc myweb-rc --patch-file replicas.json
```
이런 방식은 yaml 파일을 수정하는 것보다 굉장히 불편하고 어렵다. 
따라서 edit 명령어를 사용하는게 더 직관적이고 간단하다.
edit 명령어는 파일을 지정한다 하더라도 파일 자체를 수정하는 것이 아니라 RC의 설정을 수정하는 것이다.
```
kubectl edit -f myweb-rc.yaml
```
```yaml
# reopened with the relevant failures.
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: ReplicationController
metadata:
  creationTimestamp: "2022-05-18T05:28:45Z"
  generation: 1
  labels:
    app: web
  name: myweb-rc
  namespace: default
  resourceVersion: "168410"
  uid: eb8a9cef-fdbd-461e-8464-85b54a313254
spec:
  replicas: 3
  selector:
    app: web
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
.
.
.
```

### 선언형 오브젝트 구성
```
kubectl apply -rc myweb-rc.yaml
```

## ReplicaSets

ReplicationController -> ReplicaSets
Replication Controller에 Set 기반 Selector를 추가해서 나온게 ReplicaSets이다.
matchLabels 외에도 matchExpressions로 키:벨류 값과 operator를 지정하여 Seletor를 사용할 수 있다.

```yaml
apiVersion: apps/v1
kind: ReplicaSets
metadata:
  name: myweb-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
      env: dev
  template:
    metadata:
      labels:
        app: web
        env: dev
    spec:
      containers:
        - name: myweb
          image: ghcr.io/c1t1d0s7/go-myweb
          ports:
            - containerPort: 8080
              protocol: TCP
              
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myweb-rs-set
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: In
        values: 
          - web
      - key: env
        operator: Exists
  template:
    metadata:
      labels:
        app: web
        env: dev
    spec:
      containers:
        - name: myweb
          image: ghcr.io/c1t1d0s7/go-myweb
          ports:
            - containerPort: 8080
              protocol: TCP
              
```