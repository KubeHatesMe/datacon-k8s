#### Voting-App 실습 가이드
----

1. Pod yaml (voting-app, result-app, redis, db, worker) 작성

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
  
