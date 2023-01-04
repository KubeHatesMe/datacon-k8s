#### Voting-App 실습 가이드
----

**1. Pod yaml (voting-app, result-app, redis, postgres, worker) 작성**

▼ [voting-app-pod.yaml]
  #### Voting-App 실습 가이드
----

**1. Pod yaml (voting-app, result-app, redis, postgres, worker) 작성**

▼ [voting-app-pod.yaml]  

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: voting-app-pod
    labels:
      name: voting-app-pod
      app: demo-voting-app
  spec:
    containers:
    - name: voting-app
      image: dockersamples/examplevotingapp_vote
      ports:
           - containerPort: 80
  ```
  
▼ [result-app-pod.yaml]
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: result-app-pod
    labels:
      name: result-app-pod
      app: demo-voting-app
  spec:
    containers:
    - name: result-app
      image: dockersamples/examplevotingapp_result
      ports:
       - containerPort: 80
  ```
  
▼ [redis-pod.yaml]
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: redis-pod
    labels:
      name: redis-pod
      app: demo-voting-app
  spec:
    containers:
    - name: redis
      image: redis:latest
      ports:
       - containerPort: 6379
  ```
  
▼ [postgres-pod.yaml]
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: postgres-pod
    labels:
      name: postgres-pod
      app: demo-voting-app
  spec:
    containers:
    - name: postgres
      image: postgres:9.4
      env:
      - name: POSTGRES_USER
        value: "postgres"
      - name:  POSTGRES_PASSWORD
        value: "postgres"
      - name: POSTGRES_HOST_AUTH_METHOD
        value: trust
      ports:
       - containerPort: 5432
  ```
  
▼ [worker-app-pod.yaml]
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: worker-app-pod
    labels:
      name: worker-app-pod
      app: demo-voting-app
  spec:
    containers:
    - name: worker-app
      image: dockersamples/examplevotingapp_worker  
  ```     


**2. Service yaml (voting-app, result-app, redis, postgres) yaml 작성**
  
  
▼ [voting-app-service.yaml]
```
apiVersion: v1
kind: Service
metadata:
  name: voting-service
  labels:
    name: voting-service
    app: demo-voting-app
spec:
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30004
  type: NodePort
  selector:
     name: voting-app-pod
     app: demo-voting-app
```
  
▼ [result-app-service.yaml]
```
apiVersion: v1
kind: Service
metadata:
  name: result-service
  labels:
    name: result-service
    app: demo-result-app
spec:
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30005
  type: NodePort
  selector:
     name: result-app-pod
     app: demo-voting-app
```
  
▼ [redis-service.yaml]
```
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    name: redis-service
    app: demo-voting-app
spec:
  ports:
  - port: 6379
    name: redis-something
    targetPort: 6379
  selector:
     name: redis-pod
     app: demo-voting-app
```

▼ [postgres-service.yaml]
```
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    name: db-service
    app: demo-voting-app
spec:
  ports:
  - port: 5432
    targetPort: 5432
  selector:
     name: postgres-pod
     app: demo-voting-app
```

**3. 배포한 service의 접속 포트 확인**
```
kubectl get svc
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/voting-app-svc.png?raw=true)

**4. 웹 브라우저 앱 실행**
   - (노드 interal-ip):(접속 포트) 로 접속
        ex. result 페이지 : 192.168.49.2:30005
        ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/result1.png?raw=true)
        
   - voting 페이지 접속 후, vote 실행 (사진에선 cat에 vote함)
        ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/result2-catvote.png?raw=true)
        
   - 다시 result 페이지 결과 확인
        ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/result3.png?raw=true)

**3. [Advanced] 안정성? 가용성?을 위해 deployment 배포**
https://countrymouse.tistory.com/entry/k8s-11
  
