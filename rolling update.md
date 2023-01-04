## Rolling Update

- #### 쿠버네티스 배포 전략
  - **Rolling Update**  
    기존 버전들을 죽임과 동시에 새로운 버전을 생성하는 방식이다. 새로 생성되는 파드와 죽는 파드의 수를 조절해 가면서 업데이트를 진행하기 때문에. 서비스 중단이 없다. 새로운 컨테이너 생성 비율은 디플로이먼트를 통해서 제어할 수 있다. 이 방식의 단점은 업데이트 프로세스 동안 두 가지 버전의 컨테이너가 동시에 실행되기 때문에 버전 호환성의 문제가 발생할 수 있다. 
    ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlU01H%2Fbtq3m0j34v6%2FJhIyprNBdHw8KotabNpJX1%2Fimg.png)  
 
 - **Blue/Green**  
    Blue: 이전 버전, Green: 새 버전. 이전 버전과 새로운 버전을 모두 띄우고 새로운 버전이 준비되었다고 판단할때 기존 버전의 연결을 서비스로 부터 제거한다. 이 방식의 장적음 서비스 중단점이 없고, 버전 혼재에 따른 문제가 없다는 것이다. 다만 이전 버전과 새 버전을 동시에 실행하기 때문에 리소스가 두배로 필요하다는 단점이 있다.
    ![](https://user-images.githubusercontent.com/88528515/209255570-5b610bed-0c68-41ac-b5d8-e9ff2977556b.png)  
 
 - **Canary**  
    새로운 버전을 하나씩 올려보면서 해당 버전이 잘 동작한다 싶으면 기존 버전을 내린다. 그런 다음 또 다시 새로운 버전을 올리는 식으로 배포를 진행한다. 
    ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FCLYje%2Fbtq3mxPWRel%2FYkXtjWpFyURsCYNSduQ6r1%2Fimg.png)  
    

### 실습
----
1. **nginx-roll.yaml 파일 생성**  

    ###### ▼ [nginx-roll.yaml]
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 4
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

2. **deployment 배포 & 확인**

    ```
    kubectl apply -f nginx-roll.yaml
    kubectl get deployment
    ```
    
3. **replicaset 확인**

    ```
    kubectl get rs
    ```
    
4. **pod 확인**

    ```
    kubectl get pods
    ```
    
5. **Rolling Update 수행 : 이미지 버전 바꾸기**

    ```
    kubectl set image deployment nginx-deployment nginx=nginx:1.14.2 --record
    ```
    
 6. **Rollout status 확인**
 
    ```
    kubectl rollout status deployment nginx-deployment
    ```
    
    배포 끝난 후 deployment 내용 살펴보면 새로운 버전의 replicaset의 replica 수가 올라간 것을 볼 수 있음
    
    ```
    kubectl describe deployment nginx-deployment
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/rollout-1.14.2.png?raw=true)
 7. **이전 버전으로 Rollback 시키기**
 
    ```
    kubectl rollout history deployment nginx-deployment
    ```
    
    REVISION 확인 후, 원하는 REVISION으로 Rollback (여기선 revision=1로)
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/rollback.png?raw=true)
  
    ```
    # 기존 pod가 삭제되고, 새로 생성되는 과정을 보고 싶으면 터미널을 새로 한 개 더 열어 아래 명령어 실행
    # watch kubectl get pods
    # (만약 minikube 환경이라면 watch minikube kubectl get pods)
    kubectl rollout undo deployment nginx-deployment --to-revision=1 # 기존 터미널 창에서 실행
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/rollback%20%ED%9B%84%20describe%20dp.png?raw=true)
    pod 확인해보면 새 버전은 terminate 되고, 이전 버전 이미지의 pod가 생성되는 걸(Rollback) 확인할 수 있음
    
    ```
    kubectl get pods
    ```
    
