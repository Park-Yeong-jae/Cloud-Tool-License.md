# 클러스터 서버 구축을 위한 작업
  ## 1. NTP 서버 동기화

  ## 2. 스왑메모리 해제

  ## 3. Docker 엔진 설치
  
  ## 4. 노드간 통신을 위한 브릿지 설정 (마스터/워커)
  
  ## 5. Docker Daemon systemd 와 cgroup을 systemd로 맞춰주어야 함 (마스터/워커)
  
  ## 6. kubeadm, kubelet and kubectl 설치 (마스터/워커)
  
  <hr>
  <hr>
  
# 클러스터 실행
  ## 1. 마스터노드 Kubeadm init (마스터)
  
  ## 2. kubectl 명령어 수행을 위한 명령어 (마스터)
  
  ## 3. 마스터 노드 init 시 생성된 토큰값으로 Join (워커)
    (재차 마스터 init 시엔 토큰이 재발급되니 sudo kubeadm reset을 하고 진행)
  
  ## 4. Pod 네트워크 애드온 설치
    ( Calico, flannel, weave net 등 많은 종류가 있는데 여기서는 calico 설치 (weave net이 안됨))
