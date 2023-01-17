## Dockerfile 실습 
    (https://medium.com/code-to-express/https-medium-com-kumarnitish-hosting-static-site-using-nginx-web-server-in-docker-container-167b31df70bb)

    
### Dockerfile로 nginx 설치 후 index.html 페이지를 수정하는 Docker image 만들기
---
**1. Dockerfile 작성**
```
vi Dockerfile  
```    
    
    
##### [▼ Dockerfile ]
  ```
  FROM ubuntu
  RUN apt-get update
  RUN apt-get install nginx -y
  COPY index.html /var/www/html/
  EXPOSE 80
  CMD ["nginx", "-g", "daemon off;"]
  ```

**2. index.html 작성**
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
      
**3. image 빌드**
```
docker build -t my-nginx .   # docker build -t <이미지 이름> .
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/build-image.png?raw=true)
    
    
**4. 생성된 image 확인**
```
docker images
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-images.png?raw=true)
    

**5. 컨테이너 생성** 
```
docker run -d --name my-nginx-container -p 8080:80 my-nginx   #docker run -d --name <컨테이너 이름> -p <포트연결> <이미지 이름>
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-run.png?raw=true)

    
**6. 컨테이너 조회**
```
docker ps
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-ps.png?raw=true)
    
**7. 웹페이지 접속**
브라우저를 열어 localhost:8080 으로 접속
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/dockerfile-nginx-web.png?raw=true)
    

---    

### 동작하는 컨테이너에 접속하여 index.html 페이지 수정하고, 이를 Docker image로 만들기
---  

**1. 컨테이너 접속**
```
docker exec my-nginx-container bash  #docker exec <컨테이너 이름 or 컨테이너 id> bash
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-exec.png?raw=true)
    
**2. index.html 수정**

(1) index.html 조회
```
cd var/www/html
cat index.html
```


(2) index.html 수정  
```
#vi 설치
apt-get update
apt-get install vim

#html파일 수정
vi index.html
```

[▼ index.html ]
```
<html>
<h1>Dockerfile Test Ver.2</h1>
<h2>Hello 2023!</h2>
  <p>May 2023 be an extraordinary year!</p>
  <p>Wishing you lots of love and laughter today<p>
</html>
```

**3. 현재 container 상태를 image로 만들기**
```
docker commit my-nginx-container my-nginx:2.0
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-commit.png?raw=true)


**4. docker image 조회**
```
docker images
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-images-ver2.png?raw=true)
---  



### Docker image를 Docker hub에 올리기(push)
---
**1. docker hub에 올리기**  


(1) docker hub 로그인
```
docker login
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-login.png?raw=true)


(2) docker tag & push
```
docker tag my-nginx:2.0 kubehatesme/my-nginx:2.0
docker push kubehatesme/my-nginx:2.0
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-tag-push.png?raw=true)
        

**2. docker hub 확인**
이미지가 잘 올라갔나 확인
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-hub-my-nginx-pushed.png?raw=true)
---  



### Docker hub에 올린 image를 받아와(pull) index.html 수정사항 반영되었는지 확인하기
---
**0. 사전 준비**

(1) my-nginx:2.0 이미지를 생성하면서 이미 해당 이미지를 갖고 있으므로, kubehatesme/my-nginx:2.0 이미지 삭제
```
docker rmi kubehatesme/my-nginx:2.0  # 이미지 삭제
docker images   # 이미지 삭제 조회
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-rmi.png?raw=true)
    
**1. Docker hub에 올린 이미지 받아오기**
```
docker pull kubehatesme/my-nginx:2.0
docker images    #이미지 조회
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-pull-images.png?raw=true)
    
**2. 이미지 사용하여 컨테이너 실행**  
```
docker run -d --name my-nginx-container2 -p 8000:80 kubehatesme/my-nginx:2.0
# 8080포트는 기존 컨테이너가 사용 중이므로 다른 포트인 8000번 포트 사용
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-run-ver2.png?raw=true)
   
    
**3. 브라우저 열어 localhost:8000에 접속**
변경사항이 잘 반영되었는지 
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/docker-web2.png?raw=true)
    


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
