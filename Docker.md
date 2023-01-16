## Dockerfile 실습 
    (https://medium.com/code-to-express/https-medium-com-kumarnitish-hosting-static-site-using-nginx-web-server-in-docker-container-167b31df70bb)

    
### Dockerfile로 nginx 설치 후 index.html 페이지를 수정하는 Docker image 만들기
---
1. Dockerfile 작성
    ```
    vi Dockerfile
    ```
    
    ##### [▼ Dockerfile ]
      ```
      FROM ubuntu
      RUN apt-get update
      RUN apt-get install nginx-y
      COPY index.html /var/www/html/
      EXPOSE 80
      CMD ["nginx", "-g", "daemon off;"]
      ```

2. index.html 작성
    ```
    vi index.html
    ```
    
    ##### [▼ index.html ]
      ```
      <html>
      <h1>Dockerfile Test</h1>
      <h2>Hello 2023!</h2>
      <p>May 2023 be an extraordinary year!</p>
      </html>
      ```
      
3. 

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
