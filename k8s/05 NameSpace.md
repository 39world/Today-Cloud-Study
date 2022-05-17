## 네임스페이스
리소스를 분리하는 역할
- 서비스 마다 분리
- 사용자 마다 분리
- 환경: 개발, 스테이징, 프로덕션 등

> 서비스. DNS 이름이 분리되는 용도이다.
> RBAC: 권한을 NS에 설정

>https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/namespaces/

```
kubectl get namespaces
kubectl get ns
kubectl get pods --all-namespaces
kubectl get pods -n kube-system  

```

- kube-system : Kubernetes의 핵심 컴포넌트
- kube-public : 모든 사용자가 읽기 권한을 가지고 있음
- kube-node-lease : 노드의 HeartBeat 체크를 위한 Lease 리소스가 존재
- default


yaml 파일을 이용해 Namespace 생성 가능
dev-ns.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

yaml 파일을 이용해 생성
```
kubectl create -f dev-ns.yaml
namespace/dev created
```
Namespace 확인
```
kubectl get ns               
NAME              STATUS   AGE
default           Active   23h
dev               Active   4s
kube-node-lease   Active   23h
kube-public       Active   23h
kube-system       Active   23h
```

Namespace를 지정하지않으면 default 값에 만들어지며, 항상 네임스페이스를 지정하지 않으면 명령어들은 default 값만 보여주기때문에, 혼동하지 않도록 주의해야한다.
httpd 이미지를 dev 네임스페이스에 생성
```
kubectl run myweb --image httpd -n dev             
pod/myweb created
```
 확인해보면 네임스페이스를 명령어에 지정해줘야 조금전에 생성된 파드가 보인다
```
kubectl get pods    #default 값                  
NAME    READY   STATUS    RESTARTS   AGE
myweb   1/1     Running   0          22m

kubectl get pods -n dev         
NAME    READY   STATUS    RESTARTS   AGE
myweb   1/1     Running   0          76s
```
 
myweb.yaml 파일에 Namespace를 지정해주면 파일을 이용해 생성시 자동으로 네임스페이스가 지정된다.
```
apiVersion: v1  
kind: Pod     #kubectl api-resources
metadata:            
  name: myweb
  Namespace: dev
spec:      
  containers:
    - name: myweb
      image: httpd
      ports:
        - containerPort: 80
          protocol: TCP
```

```
kubectl create -f myweb.yaml
kubectl get pods               
NAME    READY   STATUS    RESTARTS   AGE
myweb   1/1     Running   0          30s

```