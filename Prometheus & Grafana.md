## Kubernetes Monitoring 
(https://velog.io/@ironkey/Prometheus-Grafana%EB%A1%9C-Kubernetes-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%ED%95%98%EA%B8%B0)
   Kubernetes 모니터링 툴로는 **Prometheus**와 **Grafana**가 거의 표준일 정도로 많이 사용된다.


- ### **Prometheus**
   대상 시스템으로부터 각종 모니터링 지표를 수집하여 저장하고 검색할 수 있는 시스템

- ### **Grafana**
   Prometheus를 비롯한 여러 데이터들을 시각화해주는 모니터링 툴  
   
   
   ![](https://images.velog.io/images/pingping95/post/3dfc85bc-956b-41a0-a077-d6be20857902/prometheus.png)  
   
   ▲ kubernetes 리소스의 metric 정보를 **수집(prometheus)** 하여 **대시보드로 제공(grafana)**


## 실습

- 설치에 사용할 namespace 생성 (설치 부분은 1명만 진행 or 내가 미리 해놓기)
```
kubectl create namespace monitoring
```

- helm 설치하기
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

- helm을 활용해 prometheus & grafana 설치
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
```

- cf. 설치 안 되면 [여기](https://github.com/grafana/helm-charts/blob/main/charts/grafana/values.yaml) 내용 복사한 뒤 values.yaml 파일 생성한 후 아래 명령어 실행
```
helm install prometheus prometheus-community/kube-prometheus-stack -f "values.yaml" --namespace monitoring
```

- 설치 완료되면 아래 명령어로 확인
```
kubectl --namespace monitoring get pods -l "release=prometheus"
```

- 웹 브라우저로 Grafana에 접근 (ip주소:3000), 접근 안 될 시 EC2의 인바운드 규칙 수정
```
kubectl port-forward service/prometheus-grafana 3000:80 --namespace monitoring
```
![](https://github.com/KubeHatesMe/datacon-k8s/blob/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana1.png?raw=true)

- 로그인 화면 (계정: admin, pw: prom-operator)
![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/grafana_login.png?raw=true)

- 왼쪽 메뉴에서 톱니바퀴 > datasource 클릭
![](https://github.com/KubeHatesMe/datacon-k8s/blob/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana-datasource.png?raw=true)  

- datasource로 프로메테우스 선택
![](https://github.com/KubeHatesMe/datacon-k8s/blob/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana-prometheus.png?raw=true)

- test 버튼 클릭
![](https://github.com/KubeHatesMe/datacon-k8s/blob/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana-prometheus2.png?raw=true)

- "datasource is working" 팝업창 뜨는지 확인
![](https://github.com/KubeHatesMe/datacon-k8s/blob/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana-prometheus3.png?raw=true)

- 대시보드 설정
  - https://grafana.com/grafana/dashboards 에서 마음에 드는 대시보드 선택 후 copy ID to clipboard 버튼 눌러 ID 복사
    (여기선 13332번 사용)
  - Dashboard > Import 클릭
  ![](https://github.com/KubeHatesMe/datacon-k8s/blob/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana-prometheus4.png?raw=true)
  - import 할 dashboard 번호(13332) 입력 후 Load 버튼 클릭
  ![](https://github.com/KubeHatesMe/datacon-k8s/blob/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana-prometheus5.png?raw=true)
  - prometheus datasource 선택 후 import 버튼 클릭
  ![](https://raw.githubusercontent.com/KubeHatesMe/datacon-k8s/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana-prometheus6.png)
  - Dashboard 출력 화면 예시
  ![](https://github.com/KubeHatesMe/datacon-k8s/blob/853bf85d8ecd0229901d75adbab67dacddfbb773/grafana-dashboard-done.png?raw=true)

- cf. Prometheus web UI 접속 : https://www.leafcats.com/339?category=701097 , https://freestrokes.tistory.com/149
