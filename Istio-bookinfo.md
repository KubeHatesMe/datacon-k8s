## Istio  
    (https://freedeveloper.tistory.com/417 , https://bcho.tistory.com/1297 , istio 공식 문서: https://istio.io/latest/docs/setup/getting-started/#download)
    Istio 설명 적기
    
### bookinfo 애플리케이션


  ![](https://user-images.githubusercontent.com/15958325/71655801-04538b00-2d7c-11ea-8a1c-2463f6f4e31b.png)  

<div align=center> ▲ bookinfo 구조 </div>



### 실습
---
(helm은 이미 설치되어있다는 가정 하에 진행)
1. Istio 다운로드 및 설치
    ```
    curl -L https://istio.io/downloadIstio | sh -
    cd istio-1.16.1
    export PATH=$PWD/bin:$PATH
    
    istioctl install --set profile=demo -y
    
    kubectl label namespace default istio-injection=enabled   # 앱 배포할 때 자동으로 envoy sidecar proxy 
                                                              # 추가되도록 namespace label 추가                     
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/istio-install.png?raw=true)
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/istio-inject.png?raw=true)

2. BookInfo 예제 배포
    ```
    kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    ```
   service, pod 확인
    ```
    kubectl get services
    kubectl get pods
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/apply-bookinfo.png?raw=true)
    
   kubectl get pods 결과, 모든 pod의 READY 항목이 2/2가 될 때까지 기다린다. (시간이 좀 걸림. 인내심 요망)
   ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/pods-done.png?raw=true)
   
   * 만약, minikube 환경에서 pod가 그림과 같이 ImagePullBackOff 상태일 시, 아래 명령어를 통해 pull되지 못한 이미지를 pull한다.
     ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/imgpullbackoff.png?raw=true)
     ```
     minikube ssh docker pull (이미지 이름 - ex. docker.io/istio/examples-bookinfo-reviews-v1:1.17.0 )
     ```
     ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/pull-cmd.png?raw=true)
   
   잘 배포되었는지, 아래 명령어를 통해 확인
    ```
    kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/simple.png?raw=true)
    
3. 외부 접속을 위해 istio ingress gateway 배포 및 확인
    ```
    kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
    istioctl analyze
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/analyze.png?raw=true)
    
4. ingress IP와 포트 설정
    ```
    kubectl get svc istio-ingressgateway -n istio-system
    ```
    
    [선택 1] load balancer 지원되는 환경일 시, 출력 결과 중 EXTERNAL_IP에 주소가 뜨고, 아래 명령어로 ingress IP와 포트 세팅. 
    ```
    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')

    ```
    [선택 2] 지원되지 않을 시, node port를 활용하여 접속.  
    
    아래 명령어 입력
    ```
    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
    export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
    ```  
    
    [선택 3] minikube 환경인 경우, 아래 명령어 입력
    ```
    minikube tunnel >/dev/null 2>&1 &   # 백그라운드에서 minikube tunnel 실행
    
    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
    ```
    
    ```
    echo "$INGRESS_HOST"
    echo "$INGRESS_PORT"
    echo "$SECURE_INGRESS_PORT"
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/tunnel-bg.png?raw=true)
    
    
5. Gateway URL 설정
    ```
    export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
    ```
    
    IP주소와 포트 번호 재확인
    ```
    echo "$GATEWAY_URL"
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/bookinfo-url.png?raw=true)

6. 웹페이지 접속
    브라우저 열고 http://$GATEWAY_URL/productpage 로 접속 시도
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/bookinfo-webpage.png?raw=true)

    
7. addon으로 kiali 설치해서, kiali 대시보드 살펴보기
    ```
    #kubectl apply -f samples/addons
    istioctl dashboard kiali
    
    <http://localhost:20001:kiali>
    ```
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/kiali-cmd.png?raw=true)
    
    브라우저에서 kiali dashboard 열어 왼쪽에 Graph 메뉴 클릭 후, book info의 트래픽 구조, 트래픽 흐름 등 확인
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/kiali.png?raw=true)
    ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/kiali2.png?raw=true)
