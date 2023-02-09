# Ingress

## Ingress 실습
- 실습 내용
  - 두 개의 웹 앱 배포 (dacon-main, dacon-album)
  - http://dataconsulting.com:<nodeport>로 접속 시, dacon-main으로 연결
  - http://dataconsulting.com:<nodeport>/dacon-album으로 접속 시, dacon-album으로 연결
  
  (0) ingress controller 설치 완료 (Nginx ingress Controller 사용함)
      ingress-nginx-controller service의 NodePort를 31000번으로 설정 (사유: 앱에서 31000번 포트 사용하게끔 만들어놔서)
  
  (1) dacon-main 앱을 위한 pod, service 생성
  ```
  kubectl run dacon-main --port=80 --image=kubehatesme/dacon-main:0209 -n <namespace명>    #pod 생성
  kubectl expose pod dacon-main --port=80 --type=NodePort      #service 생성
  ```
  
  (2) dacon-album 앱을 위한 pod, service 생성
  ```
  kubectl run dacon-album --port=80 --image=kubehatesme/dacon-web:0209 -n <namespace명>    #pod 생성
  kubectl expose pod dacon-album --port=80 --type=NodePort     #service 생성
  ```
  
  (3) Ingress yaml 파일 작성
  
  **[▼ dacon-ingress.yaml]**
  
  ```
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: dacon-ingress
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
      kubernetes.io/ingress.class: "nginx"
  spec:
    rules:
      - host: dataconsulting.com
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: dacon-main
                  port:
                    number: 80
            - path: /album
              pathType: Prefix
              backend:
                service:
                  name: dacon-album
                  port:
                    number: 80
  ```
  
  (4) Ingress 배포
  ```
  kubectl apply -f dacon-ingress.yaml -n <namespace명>
  ```
  
  (5) Ingress 조회
  ```
  kubectl get ingress -o wide   #결과 중 ADDRESS값 확인
  ```
  
  (6) host 등록 - (완료)
  ```
  vi /etc/hosts
  # 가장 아랫줄에 <ingress의 ADDRESS> dataconsulting.com 추가
  ```
  
  (7) 접속
    - vnc viewer 사용해서 firefox로 http://dataconsulting:31000과 http://dataconsulting:31000/album에 접속
    - curl 명령어를 통해 동일한 주소로 접속
    - 성공적인 접속 결과  
  
      **[dacon-main]**  
      ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/dacon-main.PNG?raw=true)  
  
      **[dacon-album]**  
      ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/dacon-album.PNG?raw=true)
