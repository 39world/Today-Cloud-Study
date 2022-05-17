## Label & LabelSelector
>https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/labels/
>https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/common-labels/

## Label
pod들에 라벨을 붙여 설명을 달아주는 것.

레이블 확인
```
kubectl get pods --show-labels

kubectl get pods --show-labels -n dev
NAME    READY   STATUS    RESTARTS   AGE   LABELS
myweb   1/1     Running   0          44m   run=myweb

```

레이블 추가
```
kubectl label pods myweb APP=apache -n dev
pod/myweb labeled

kubectl get pods --show-labels -n dev     
NAME    READY   STATUS    RESTARTS   AGE   LABELS
myweb   1/1     Running   0          46m   APP=apache,run=myweb
```

레이블 수정
```
kubectl label pods myweb APP=apache2 -n dev --overwrite
pod/myweb labeled

kubectl get pods --show-labels -n dev                  
NAME    READY   STATUS    RESTARTS   AGE   LABELS
myweb   1/1     Running   0          47m   APP=apache2,run=myweb
```

yaml 파일로 관리하는 경우 metadata에 추가해준다.
```yaml
apiVersion: v1  
kind: Pod    
metadata:            
  name: myweb-label
  labels:
    APP: apache
    ENV: staging
spec:      
  containers:
    - name: myweb
      image: httpd
      ports:
        - containerPort: 80
          protocol: TCP
```

## LabelSelector
- 검색
- 리소스 간 연결

### 검색 방법
#### 일치성 (equality base)
- =
- ==
```
kubectl get pods -l APP=nginx
kubectl get pods -l APP==nginx
```
- !=
```
kubectl get pods -l 'APP!=nginx'
```

#### 집합성 (set base)
- in
```
kubectl get pods -l 'ENV in (staging)'
kubectl get pods -l 'APP in (nginx, apache)'
```
- notin
```
kubectl get pods -l 'APP notin (apache)'
```
- exist : 키만 매칭한다.
```
kubectl get pods -l 'APP'
```
- doesnotexists : 키를 제외하고 매칭한다.
```
kubectl get pods -l '!APP'
```

### Annotations
레이블과 비슷하지만 어노테이션은 비식별 데이터이다. 도구 및 라이브러리와 같은 클라이언트에서 이 메타데이터를 검색할 수 있다.
key:Value 쌍으로 구성된다.
어노테이션의 예시
- 빌드,릴리스 또는 타임 스탬프, 릴리스 ID, git 브랜치, PR 번호, 이미지 해시 등
- 로깅, 모니터링, 분석 등에 대한 포인터
- 디버깅 정보
- 책임자의 전화번호 또는 호출기의 번호 
    
어노테이션 확인
```
kubectl get pods myweb -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:   # calico에서 달아둔 어노테이션을 확인할 수 있다.
    cni.projectcalico.org/containerID: 42d99e11f4e272785c16881e48526f381743f3e7a2991b0e41534abb13fad00e
    cni.projectcalico.org/podIP: 10.233.92.4/32
    cni.projectcalico.org/podIPs: 10.233.92.4/32
  creationTimestamp: "2022-05-17T05:33:58Z"
  labels:
    APP: apache2
```
  
어노테이션 추가 방법  
```
kubectl annotate pods myweb created-by=Kang

kubectl get pods myweb -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: 42d99e11f4e272785c16881e48526f381743f3e7a2991b0e41534abb13fad00e
    cni.projectcalico.org/podIP: 10.233.92.4/32
    cni.projectcalico.org/podIPs: 10.233.92.4/32
    created-by: Kang
  creationTimestamp: "2022-05-17T05:33:58Z"
  labels:
    APP: apache2
```

어노테이션 수정
```
kubectl annotate pods myweb created-by=Kang --overwrite
```

어노테이션 제거
```
kubectl annotate pods myweb created-by-
```

yaml 파일을 사용하는 방법
```yaml
apiVersion: v1  
kind: Pod 
metadata:            
  name: myweb-label
  labels:
    APP: apache
    ENV: staging
  annotations:
    Created-by: Kang
spec:      
  containers:
    - name: myweb
      image: httpd
      ports:
        - containerPort: 80
          protocol: TCP
```