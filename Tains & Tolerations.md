# Taints & Tolerations

## Taints
- Taint는 얼룩이라는 뜻
- Node에 Taint를 설정하여 아무 Pod들이나 할당되지 못하도록 제약을 건다.
- Taint는 Key=Value:Effect 형태로 설정하며, Effect는 생략 가능하다.
- Effect 종류
  - NoSchedule : 기존 실행 중인 Pod는 어쩔수 없고, 앞으로 실행될 Pod에 대해서만 스케쥴링 제한
  - NoExecute : 앞으로 생성 될 Pod에 대한 스케쥴링을 제한함은 물론, 기존에 해당 Node에 배치된 Pod 모두 방출
  - PreferNoSchedule : Soft한 룰 / 기존 실행 중인 Pod는 허용하고, 앞으로 생성될 Pod도 스케쥴링되는 것을 선호하진 않지만, 해당 Node 밖에 스케쥴링 될 곳이 없다면 허용해줌
  
  ![](https://miro.medium.com/max/1282/1*7e0BwxxNBMrx2J4TQGMjuA.png)


## Tolerations

- Tolerations 요소  
  ![](https://github.com/KubeHatesMe/datacon-k8s/blob/master/image/about_toleration.PNG?raw=true)

## Taints & Tolerations 실습
  ### Master Node에 Pod 배포하기
   - **방법 1. Master Node의 Taints를 해제하고, Pod 배포 시 NodeName을 Master Node로 지정하여 배포하기**  
   
      (1) Master Node의 Taints 조회  
      ```
      kubectl describe node master | grep Taints
      # Taints:  node-role.kubernetes.io/control-plane:NoSchedule
      ```
      
      (2) Master Node의 Taints 해제
      ```
      kubectl taint nodes master node-role.kubernetes.io/control-plane-
      # node/master untainted
      ```
      
      (3) Pod의 yaml 파일 작성  
      
      **[▼ pod-nodename.yaml]**
      ```
      apiVersion: v1
      kind: Pod
      metadata:
        name: my-busybox
        labels:             
          app: busybox        
      spec:
        containers:
        - name: my-busybox
          image: busybox:1.28
          command: ['sh', '-c', 'sleep 3600']
        nodeName: master     #Pod를 올릴 Node의 이름
      ```
      
      (4) Pod 배포하기
      ```
      kubectl apply -f pod-nodename.yaml -n <namespace명>
      ```  
      
      (별첨) master Taints 되돌리기
      ```
      kubectl taint nodes master node-role.kubernetes.io/control-plane:NoSchedule
      # node/master tainted
      ```
      
       
   - **방법 2. Master Node의 Taints를 조회하고, Pod 배포 시 Tolerations 과 NodeName을 지정하여 배포하기**  
   
      (1) Master Node의 Taints 조회  
      ```
      kubectl describe node master | grep Taints
      # Taints:  node-role.kubernetes.io/control-plane:NoSchedule
      ```
      (2) Master Node의 Label 조회
      ```
      kubectl describe node master | grep Labels -A 10 #Labels 이후 10줄까지 출력
      #Labels:       beta.kubernetes.io/arch=amd64
      #              beta.kubernetes.io/os=linux
      #              kubernetes.io/arch=amd64
      #              kubernetes.io/hostname=master
      #              kubernetes.io/os=linux
      #              node-role.kubernetes.io/control-plane=
      #              node.kubernetes.io/exclude-from-external-load-balancers=
      #Annotations:  kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
      #              node.alpha.kubernetes.io/ttl: 0
      #              projectcalico.org/IPv4Address: 172.25.1.97/24
      #              projectcalico.org/IPv4IPIPTunnelAddr: 172.16.219.64
      ```  
      (3) Pod yaml 파일 작성  
      
      **[▼ pod-tolerations-nodename.yaml]**
      ```
      apiVersion: v1
      kind: Pod
      metadata:
        name: my-busybox
        labels:             
          app: busybox        
      spec:
        containers:
        - name: my-busybox
          image: busybox:1.28
          command: ['sh', '-c', 'sleep 3600']
        tolerations:              #Tolerations
        - key: node-role.kubernetes.io/control-plane    #Pod를 올릴 Node의 Taint의 key값
          operator: Exists                              #key값이 
          effect: NoSchedule                            #Pod를 올릴 Node의 Taint의 option값
        nodeSelector:             #Pod를 올릴 Node의 Label
          kubernetes.io/hostname: master
      ```
      
      (3) Pod 배포하기
      ```
      kubectl apply -f pod-tolerations-nodename.yaml -n <nameespace명>
      ```
    
    
    
