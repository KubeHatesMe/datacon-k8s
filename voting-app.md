#### Voting-App 실습 가이드
----

**1. Pod yaml (voting-app, result-app, redis, postgres, worker) 작성**

▼ [voting-app-pod.yaml]
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

<
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
  type: LoadBalancer
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
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
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


  
  
  
