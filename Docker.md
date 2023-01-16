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
      RUN apt-get install nginx -y
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
      
3. image 빌드
    ```
    docker build -t my-nginx .   # docker build -t <이미지 이름> .
    ```
    [build-image]
    
    
4. 생성된 image 확인
    ```
    docker images
    ```
    [docker-images]

5. 컨테이너 생성  docker run -d --name my-nginx-container -p 8080:80 my-nginx   #docker run -d --name <컨테이너 이름> -p <포트연결> <이미지 이름>
[docker-run]
6. 컨테이너 조회 docker ps
[docker-ps]
7. 웹페이지 접속
localhost:8080
[dockerfile-nginx-web]
8. 컨테이너 접속 docker exec my-nginx-container bash  #docker exec <컨테이너 이름 or 컨테이너 id> bash
[docker-exec]
9. index.html 수정
[index.html 조회]
cd var/www/html
cat index.html

[index.html 수정]
#vi 설치
apt-get update
apt-get install vim

#html파일 수정
vi index.html

<html>
<h1>Dockerfile Test Ver.2</h1>
<h2>Hello 2023!</h2>
  <p>May 2023 be an extraordinary year!</p>
  <p>Wishing you lots of love and laughter today<p>
</html>

10. 현재 container 상태를 image로 만들기
docker commit my-nginx-container my-nginx:2.0
[docker commit]

11. docker image 조회
docker images
[docker-images-ver2]

12. docker hub에 올리기
#docker hub 로그인
docker login
[docker-login]

# docker tag & push
docker tag my-nginx:2.0 kubehatesme/my-nginx:2.0
docker push kubehatesme/my-nginx:2.0
[docker-tag-push]

13. docker hub 확인
이미지가 잘 올라갔나 확인
[docker-hub-my-nginx-pushed]

14. 올라간 이미지 pull해서 localhost:8080하면 잘 표출되는지 보기!
    기존꺼랑 충돌날 수 있으니까 기존 컨테이너 삭제하고 진행하기



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
