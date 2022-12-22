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
      ports:
      - port: 80
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
      
      
      ⊙ LoadBalancer  
      
      
      ⊙ *ExternalName*
      
  - Replica Set 생성하기
  - Deployment 생성하기
  ###### cf. Replica Set과 Deployment는 거의 유사한데, Deployment가 Replica Set의 상위 개념으로, Rolling-Update 등 pod 배포 시에 관리를 용이하게 하기 위해 실제 운영에는 Deployment를 더 많이 쓰는 듯함. (어차피 Deployment 배포하면 Replica Set 도 자동 생성되므로)
```
add source code
```
3. (label and selectors, taints and tolerations)
```
add source code
```
4. ingress
```
add source code
```
5. monitoring
```
add source code
```
6. rolling update (vs Blue/Green, Canary) (https://phoenixnap.com/kb/kubernetes-rolling-update : v1의 nginx-test.yaml 파일의 이미지 버전 1.14.0으로 수정, v2껄 1.14.2로!  /  https://bcho.tistory.com/1266)
```
add source code
```
7. voting-app 실습 - MSA 개념 설명 (https://wefree.tistory.com/87)

    [voting-app 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/voting-app.md)   


8. [advanced] Istio - bookinfo - Service mesh 였나? 개념 

```
add source code
```

