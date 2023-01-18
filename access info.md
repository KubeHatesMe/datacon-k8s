# 실습 환경 접속 방법
#### AWS 접속 계정
- **계정ID**:
- **사용자 이름**:
- **암호**:

Minikube를 쓸까..
에러나면 delete 했다가 다시 만들면 돼서 편할 거 같긴 한데..

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
      ```
      minikube start --mount --mount-string="/home/(Username)/vol-mount:/home/docker/vol-mount"
      # "(source Directory - ex. 로컬 디렉토리):(target Directory - minikube 내 디렉토리)
      ```
      또한, deployment yaml 파일의 hostPath를 마운트한 minikube directory (/home/docker/vol-mount)로 바꿔주어야 함.

    (1) hostpath로 사용할 경로 생성
        Worker Node1에 접속하여 셸을 열고 아래 명령어를 실행하여 디렉터리 생성
    ```
    sudo mkdir vol-mount
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
              path: /home/hy/vol-mount  #/home/(Username)/vol-mount
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

