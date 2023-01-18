## Ingress 실습
---
1. Ingress controller 설치하기 (minikube 환경)
```
minikube addons enable ingress   # nginx-ingress-controller 설치
kubectl get namespaces    # nginx-ingress-controller가 설치된 namespace 확인
kubectl config set-context --current --namespace=ingress-nginx   # 기본 namespace를 ingress-nginx로 변경
```
[ingress-addon-change-ns]


