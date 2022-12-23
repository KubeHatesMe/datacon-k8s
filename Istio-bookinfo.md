## Istio  
    (https://freedeveloper.tistory.com/417 , https://bcho.tistory.com/1297 , istio 공식 문서: https://istio.io/latest/docs/setup/getting-started/#download)
    Istio 설명 적기
    
### bookinfo 애플리케이션


  ![](https://user-images.githubusercontent.com/15958325/71655801-04538b00-2d7c-11ea-8a1c-2463f6f4e31b.png)  

<div align=center> ▲ bookinfo 구조 </div>



### 실습
---
1. Istio 다운로드 및 설치
    ```
    curl -L https://istio.io/downloadIstio | sh -
    cd istio-<버전명> #존재하는 디렉토리명따라 이동
    export PATH=$PWD/bin:$PATH
    
    istioctl install --set profile=demo -y
    kubectl label namespace default istio-injection=enabled   # 앱 배포할 때 자동으로 envoy sidecar proxy 
                                                              # 추가되도록 namespace label 추가                     
    ```
    
2. BookInfo 예제 배포
    ```
    kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    ```
   service, pod 확인
    ```
    kubectl get services
    kubectl get pods
    ```
    
3. 외부 접속을 위해 istio ingress gateway 배포 및 확인
    ```
    kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
    istioctl analyze
    ```
    
4. ingress IP와 포트 설정 (필요한건가?)
    ```
    kubectl get svc istio-ingressgateway -n istio-system
    ```
    
    load balancer 지원되는 환경일 시, 출력 결과 중 EXTERNAL_IP에 주소가 뜨고, 아래 명령어로 ingress IP와 포트 세팅. 
    ```
    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')

    ```
    지원되지 않을 시, node port를 활용하여 접속.  
    
    아래 명령어 입력
    ```
    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
    export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
    ```  
    
    
5. Gateway URL 설정
    ```
    export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
    ```
    
    IP주소와 포트 번호 재확인
    ```
    echo "$GATEWAY_URL"
    ```

6. 웹페이지 접속
    브라우저 열고 http://$GATEWAY_URL/productpage 로 접속 시도

    
7. addon으로 kiali 설치해서, kiali 대시보드 살펴보기
