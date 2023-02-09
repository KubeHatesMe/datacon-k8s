# Taints & Tolerations

## Taints
- 정의


## Tolerations
- 정의


## Taints & Tolerations 실습
  ### Master Node에 Pod 배포하기
   - **방법 1. Master Node의 Taints를 해제하고, Pod 배포 시 NodeName을 Master Node로 지정하여 배포하기**  
   
      (1) Master Node의 Taints 조회  
      ```
      kubectl ~~~
      ```
      
      (2) Master Node의 Taints 해제
      ```
      kubectl ~~~
      ```
      
      (3) Pod의 yaml 파일 작성  
      
      **[▼ pod-nodename.yaml]**
      ```
      Pod yaml 파일
      ```
      
      (4) Pod 배포하기
      ```
      kubectl ~~~
      ```  
      
       
   - **방법 2. Master Node의 Taints를 조회하고, Pod 배포 시 Tolerations 과 NodeName을 지정하여 배포하기**  
   
      (1) Master Node의 Taints 조회  
      ```
      kubectl ~~~
      ```
      
      (2) Pod yaml 파일 작성  
      
      **[▼ pod-tolerations-nodename.yaml]**
      ```
      Pod yaml 파일
      ```
      
      (3) Pod 배포하기
      ```
      kubectl ~~~
      ```
    
    
    
