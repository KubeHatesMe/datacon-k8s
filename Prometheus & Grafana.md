## Kubernetes Monitoring
   Kubernetes 모니터링 툴로는 **Prometheus**와 **Grafana**가 거의 표준일 정도로 많이 사용된다.


- ### **Prometheus**
   대상 시스템으로부터 각종 모니터링 지표를 수집하여 저장하고 검색할 수 있는 시스템

- ### **Grafana**
   Prometheus를 비롯한 여러 데이터들을 시각화해주는 모니터링 툴  
   
   
   ![](https://images.velog.io/images/pingping95/post/3dfc85bc-956b-41a0-a077-d6be20857902/prometheus.png)  
   
   ▲ kubernetes 리소스의 metric 정보를 **수집(prometheus)** 하여 **대시보드로 제공(grafana)**
