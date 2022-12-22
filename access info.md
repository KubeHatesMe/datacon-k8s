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
    - 방법1.  yaml 파일 활용  


    (1) yaml 파일 작성
    [nginx-pod.yaml]
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
    ```
    (2) pod 배포
    ```
    kubectl apply -f nginx-pod.yaml
    ```
    - 방법2.  Command Line Tool 활용
    ```
    # kubectl run <pod명> --image=<image명> --port=<port번호>
    kubectl run nginx-pod --image=nginx --port=80
    ```
    - 결과 조회
    ```
    kubectl get pods
    ```
  - Service 생성하기
    - (1) yaml 파일 작성
      [nginx-svc.yaml]
      ```
      add source code
      ```
    - (2) service 배포
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
6. rolling update (vs Blue/Green, Canary)
```
add source code
```
7. voting-app 실습 (https://wefree.tistory.com/87)

    [voting-app 실습 가이드](https://github.com/KubeHatesMe/datacon-k8s/blob/master/voting-app.md)   


8. [advanced] Istio - bookinfo 

```
add source code
```

