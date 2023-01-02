# 실습 환경 접속 방법
#### AWS 접속 계정
- **계정ID**:
- **사용자 이름**:
- **암호**:


#### VM 접속
- (이름) : (ip주소)
- (이름) : (ip주소)
- (이름) : (ip주소)
- (이름) : (ip주소)
- (이름) : (ip주소)
- (이름) : (ip주소)
- (이름) : (ip주소)

##### Kubernetes 실습 가이드
###### EC2로 클러스터 만들고, namespace로 구분하는 방식으로! 도커 이미지는 미리 github에 올려놓기

1. 기본적인 명령어 사용
  - namespace 구분
  - pod 확인 (예시로 미리 생성해놓기)
    - kubectl get pods, kubectl get pods -o wide, kubectl describe pod (pod name)
    - 각 namespace 별 pod 확인
    ```
    kubectl get pods -n <namespace명>
    ```
    - 모든 namespace에 있는 pod 확인
    ```
    kubectl get pods --all-namespaces
    ```
    - pod 상세 정보 출력
    ```
    kubectl get pods -n <namespace명> -o wide
    ```
    - node 확인
    ```
    kubectl get nodes
    ```
    - pod 삭제
    ```
    kubectl delete pod <pod명> -n <namespace명>
    kubectl get pods -n <namespace명> #삭제된 결과 조회
    ```
    - 기본 namespace 변경
    ```
    kubectl config set-context --current --namespace=<namespace명>
    ```
2. 기본 object들 실습 : pod(생성), replicaset, deployment, services
  - pod 생성하기  


    **방법1. yaml 파일 활용**   
    
    
    (1) yaml 파일 작성  
    ###### [nginx-pod.yaml]
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-nginx
      labels:             # labels 값이
        app: nginx        # service의 selector 값과 일치해야 함 
    spec:
      containers:
      - name: my-nginx
        image: nginx:1.14.0
        ports:
        - containerPort: 80
    ```
    (2) pod 배포
    ```
    kubectl apply -f nginx-pod.yaml
    ```
    **방법2. Command Line Tool 활용**
    ```
    # kubectl run <pod명> --image=<image명> --port=<port번호>
    kubectl run nginx-pod --image=nginx --port=80
    ```
    - 결과 조회
    ```
    kubectl get pods
    ```
  - Service 생성하기  


    (1) yaml 파일 작성  
    
    
    ###### [nginx-svc.yaml]
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: my-nginx-svc
    spec:
      selector:           # service의 대상이 되는 오브젝트의
        app: nginx        # label값과 일치해야 함
      type: NodePort      # service type: Nodeport
      ports:
      - nodePort: 31000   # 외부 접속 시 사용하는 port 번호, 미 설정시 3만번대에서 자동 할당
        port: 8080        # Cluster 내부에서 사용하는 service 객체의 port 번호
        targetPort: 80    # service-> pod 로 전달 시 사용하는 port 번호
        protocol: TCP

    ```
    (2) service 배포
    ```
    kubectl apply -f nginx-svc.yaml
    ```
    - 참고  


      **Service type**  
      
      
      ⊙ ClusterIP  
      
      
      ⊙ NodePort  
      ###### nodePort, port, targetPort 이해
      ![](https://dz2cdn1.dzone.com/storage/temp/14848977-nodeport.jpg)
      
      ⊙ LoadBalancer  
      
      
      ⊙ *ExternalName*
      
  - Replica Set 생성하기
  ```
  apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    name: my-nginx-rs
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
        
   #까지만 해도 되나,,? 아님 밑에
    template:
       metadata:
          name: my-nginx
          labels:             # labels 값이
            app: nginx        # service의 selector 값과 일치해야 함 
        spec:
          containers:
          - name: my-nginx
            image: nginx:1.14.0
            ports:
            - containerPort: 80
   # 또 추가해줘야 하나,,,?
  ```
  - Deployment 생성하기
  ###### cf. Replica Set과 Deployment는 거의 유사한데, Deployment가 Replica Set의 상위 개념으로, Rolling-Update 등 pod 배포 시에 관리를 용이하게 하기 위해 실제 운영에는 Deployment를 더 많이 쓰는 듯함. (어차피 Deployment 배포하면 Replica Set 도 자동 생성되므로)
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:                  #여기서부터,, 써야되는 건지,,? pod.yaml 배포했으면 필요없는건지?
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.14.0
          ports:
          - containerPort: 80
  ```
3. (label and selectors, taints and tolerations)
```
add source code
```
4. Volume

  (1) hostpath로 사용할 경로 생성
  ```
  ex) mkdir 어찌고 저찌고
  ```  
  
  
  (2) Deployment(hostPath Volume 지정)와 Service 배포
  ###### [nginx-dp-vol.yaml]
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment-volume
  spec:
    selector:
      matchLabels:
        app: nginx
    replicas: 1
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
          volumeMounts:
          - name: shared-data
            mountPath: /usr/share/nginx/html
        volumes:
        - name: shared-data
          hostPath:
            path: /home/yoon/Workspace/k8sstudy/shared  # 경로  변경 필요 
            type: Directory

  ---

  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    type: NodePort
    ports:
    - port: 8080
      targetPort: 80
      protocol: TCP
      name: http
    selector:
      app: nginx
  ```  
  
  
  
5. monitoring (Prometheus & Grafana)  

    [Prometheus & Grafana 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/Prometheus%20%26%20Grafana.md)
    
6. rolling update (vs Blue/Green, Canary) (https://phoenixnap.com/kb/kubernetes-rolling-update : v1의 nginx-test.yaml 파일의 이미지 버전 1.14.0으로 수정, v2껄 1.14.2로!  /  https://bcho.tistory.com/1266)  

    [rolling update 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/rolling%20update.md)  
    
7. voting-app 실습 - MSA 개념 설명 (https://wefree.tistory.com/87)

    [voting-app 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/voting-app.md)   


8. [advanced] Istio - bookinfo - Service mesh (https://bcho.tistory.com/1297)

```
add source code
```

