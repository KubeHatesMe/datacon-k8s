## Ingress 실습
---
1. Ingress controller 설치하기 (minikube 환경)
```
minikube addons enable ingress   # nginx-ingress-controller 설치
kubectl get namespaces    # nginx-ingress-controller가 설치된 namespace 확인
kubectl config set-context --current --namespace=ingress-nginx   # 기본 namespace를 ingress-nginx로 변경
```
[ingress-addon-change-ns]


2. 첫 번째 app 배포하기
```
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0  #deployment 만든 후
kubectl expose deployment web --type=NodePort --port=8080   #expose 시켜줌
```
[ingress-app1]

3. 두 번째 app 배포하기
```
kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0  #deployment 만든 후
kubectl expose deployment web2 --type=NodePort --port=8080   #expose 시켜줌
```
[ingress-app2]

4. ingress yaml 파일 작성
#####[ ▼ example-ingress.yaml ]
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
          - path: /v2
            pathType: Prefix
            backend:
              service:
                name: web2
                port:
                  number: 8080
```

5. ingress 배포 후 조회
```
kubectl apply -f example-ingress.yaml
kubectl get ingress
```
[ingress-get-ingress]

6. host값 변경
```
sudo vi /etc/hosts
## /etc/hosts 파일 열리면 가장 밑에 아랫줄 추가
<ingress의 ADDRESS값> hello-word.info
```
[ingress-hosts]

7. curl 명령어를 통해 ingress 결과 조회
```
curl hello-world.info  # verson:1.0.0
curl hello-world.info/v2   # version:2.0.0
```
[ingress-result]
