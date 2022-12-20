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

#####Kubernetes 실습 가이드
######EC2로 클러스터 만들고, namespace로 구분하는 방식으로! 도커 이미지는 미리 github에 올려놓기

1. 기본적인 명령어 사용
  - namespace 구분
  - pod 확인 (예시로 미리 생성해놓기)
    - kubectl get pods, kubectl get pods -o wide, kubectl describe pod (pod name)
  - node 확인
    - kubectl get nodes
  - pod 삭제
    - kubectl delete pod (pod name)
2. 기본 object들 실습 : pod(생성), replicaset, deployment, services
  - Command Line 활용
  - yaml파일 활용
3. (label and selectors, taints and tolerations)
4. ingress
5. monitoring
6. rolling update (vs Blue/Green, Canary)
7. voting-app 실습 (https://wefree.tistory.com/87)
8. [advanced] Istio - bookinfo 

```
add source code
```

