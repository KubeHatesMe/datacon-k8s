#### Voting-App 실습 가이드
----

1. Pod yaml (voting-app, result-app, redis, db, worker) 작성
  - voting-app-pod.yaml
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
  - result-app-pod.yaml
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
  - redis-pod.yaml
  - postgres-pod.yaml
  - worker-app-pod.yaml
