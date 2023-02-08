# 실습 환경 접속 방법 #

  (1) Putty 열어 Hostname(or Ip address)에 ```centos@210.104.76.192``` / Port에 ```30001``` 입력
   ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/putty_ip.JPG?raw=true)  
   
  (2) 좌측 메뉴바에서 Connection 아래 SSH > Auth 클릭 후, ```Browse..``` 버튼을 눌러 key 파일 가져오기
   ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/putty_key.JPG?raw=true)
  
  

# Kubernetes 실습 가이드
  * **Cluster 구성**
     - Master Node 1대, Worker Node 2대
     - KT Cloud Server 사용  
     
  * **Namespace 할당**
      |성함|namespace|
      |------|:---:|
      |강인균 팀장님|ik|
      |박경규 부장님|kk|
      |이승우 부장님|sw|
      |서동희 차장님|dh|
      |이영주 차장님|yj|
        
        
## 1. 기본 명령어 실습
  - 조회하기 : ```kubectl get <조회할 objeect>```
    ```
    kubectl get namespace   # namespace 조회
    kubectl get nodes   # 클러스터를 구성하는 node들 조회
    kubectl get nodes -o wide   # 클러스터를 구성하는 node 정보를 조금 더 자세히 조회
    kubectl get pods -A   # 모든 namespace에 있는 pod들 조회
    kubectl get pods -n <namespace명>   # 특정 namespace에 있는 pod들 조회
    kubectl describe pod nginx-default -n default   # default namespace에 있는 nginx-default pod에 대한 정보를 자세히 조회
    kubectl get service   # 현재 namespace에 있는 service를 조회
    kubectl delete pod <pod명> -n <namespace명>   #  특정 namespace에 있는 이름이 <pod명>인 pod를 삭제
    ```
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
    kubectl config view --minify | grep namespace    #현재 속해있는 namespace 확인
    ```
## 2. 기본 object들 실습 : pod(생성), replicaset, deployment, services
  - pod 생성하기  


    **방법1. yaml 파일 활용**   
    
    
    (1) yaml 파일 작성  
    ###### [nginx-pod.yaml]
    ```
    apiVersion: v1
    kind: #here          #Object종류
    metadata:
      name: #here        #Pod의 이름
      labels:            #labels 값이
        app: #here       #Service의 selector 값과 일치해야 함
    spec:
      containers:
      - name: #here     #container의 이름
        image: #here    #사용할 image
        ports:
        - containerPort: 80

    ```
    
    ###### [nginx-pod-ans.yaml]
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
    
    (3) 브라우저 접속 및 확인
    ```
    kubectl get nodes -o wide # 출력 결과 중 node의 <internal-IP> 값 확인
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/svc-nodeip.JPG?raw=true)  
    
    
    브라우저 열어 (internal-ip 주소):(nodeport값) 으로 접속하여 출력 화면 확인
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/nginx-web.JPG?raw=true)

    
    - 참고  


      **Service type**  
      
      
      ⊙ ClusterIP  
      
      
      ⊙ NodePort  
      ###### nodePort, port, targetPort 이해
      ![](https://dz2cdn1.dzone.com/storage/temp/14848977-nodeport.jpg)
      
      ⊙ LoadBalancer  
      
      
      ⊙ *ExternalName*
      
  - Replica Set 생성하기
  ###### [nginx-rs.yaml]
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
  ```
  - Deployment 생성하기
  ###### cf. Replica Set과 Deployment는 거의 유사한데, Deployment가 Replica Set의 상위 개념으로, Rolling-Update 등 pod 배포 시에 관리를 용이하게 하기 위해 실제 운영에는 Deployment를 더 많이 쓰는 듯함. (어차피 Deployment 배포하면 Replica Set 도 자동 생성되므로) >> 기존 위에서 생성한 rs 놔둔 채로 아래 deployment 배포하면, 기존 rs는 desired가 0으로 되어 기존 pod들도 없어지고, 아래 deployment에 의한 pod들만 생겨남
  ###### [nginx-dp.yaml]
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
    template:                  
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

3. Volume

    * if, minikube로 실행한 경우, 기존 minikube 클러스터는 삭제하고 (minikube delete) 마운트 명령어를 써서 minikube를 시작해야함.
      로컬에 마운트할 디렉토리 (여기선 /home/hy/vol-mount) 만들고 
      ```
      minikube start --mount --mount-string="/home/(Username)/vol-mount:/home/docker/vol-mount"
      # "(source Directory - ex. 로컬 디렉토리):(target Directory - minikube 내 디렉토리)
      ```
      또한, deployment yaml 파일의 hostPath를 마운트한 minikube directory (/home/docker/vol-mount)로 바꿔주어야 함.

    (1) hostpath로 사용할 경로 생성
        Worker Node1에 접속하여 셸을 열고 아래 명령어를 실행하여 디렉터리 생성
    ```
    sudo mkdir vol-mount
    #minikube인 경우 위 마운트 명령어로 이미 minikube 내에 디렉토리가 생성됨
    ```   
    
    (2) 만든 경로에 index.html 파일 생성 및 확인
    ```
    sudo sh -c "echo 'Hello from Kubernetes storage' > vol-mount/index.html"
    cat vol-mount/index.html #결과: Hello from Kubernetes stroage
    ```
    
    
    (3) Deployment(hostPath Volume 지정)와 Service 배포
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
              path: /home/docker/vol-mount 
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
    
    (4) Deployment, Service 배포
    ```
    kubectl apply -f nginx-dp-vol.yaml
    ```  
    
    
    (5) 브라우저에 접속 **(노드 ip):(접속 포트)**
    ```
    kubectl get svc nginx #출력 결과 중 PORT(S)에서 외부 접속 포트 확인
    ```
    
    (6) 출력 결과가 작성한 index.html 내용을 잘 출력하는지 확인
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/volume.png?raw=true)
  
  
4. monitoring (Prometheus & Grafana)  

    [Prometheus & Grafana 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/Prometheus%20%26%20Grafana.md)
    
5. rolling update (vs Blue/Green, Canary) (https://phoenixnap.com/kb/kubernetes-rolling-update : v1의 nginx-test.yaml 파일의 이미지 버전 1.14.0으로 수정, v2껄 1.14.2로!  /  https://bcho.tistory.com/1266)  

    [rolling update 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/rolling%20update.md)  
    
6. voting-app 실습 - MSA 개념 설명 (https://wefree.tistory.com/87)

    [voting-app 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/voting-app.md)   

7. Ingress  


    [ingress 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/ingress.md)   

8. [advanced] Istio - bookinfo - Service mesh (https://bcho.tistory.com/1297)

    [Istio - bookinfo 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/Istio-bookinfo.md)

