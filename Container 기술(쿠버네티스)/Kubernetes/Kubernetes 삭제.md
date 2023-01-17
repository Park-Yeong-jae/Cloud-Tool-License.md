# 쿠버네티스 삭제 가이드

  ### 1) kubernetes 초기화
    $ systemctl stop kubelet
    $ docker rm -f $(docker ps -aq)
    $ docker rmi -f $(docker images -q)
    $ systemctl restart docker
    $ systemctl start kubelet
    $ rm -f  ~/.kube/config
    $ kubeadm reset
      * The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d
      * CNI 항목 제거가 필요함
  
  ### 2) CNI 항목 제거 및 설정파일 삭제
    * flannel : rm -f /etc/cni/net.d/10-flannel.conflist
    * calico : rm -f /etc/cni/net.d/*calico*
      $ rm -rf /var/run/calico/
      $ rm -rf /var/lib/calico/
      $ rm -rf /etc/cni/net.d/
      $ rm -rf /var/lib/cni/
    
      $ iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
      $ rm -f $HOME/.kube/config
      
    
  ### 3) 쿠버네티스 & 도커 가동 중지
    $ systemctl stop kubelet
    $ systemctl stop docker
    
  ### 4) docker 삭제
    * docker 설치 확인
      $ dpkg -l | grep -i docker
    * docker 삭제
      $ sudo apt-get purge -y docker-engine docker docker.io docker-ce
      $ sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce
    * 모든 이미지, 컨테이너, 볼륨 삭제
      $ rm -rf /var/lib/docker /etc/docker
      $ rm /etc/apparmor.d/docker
      $ groupdel docker
      $ rm -rf /var/run/docker.sock
    * DOCKER 관련 파일 삭제
      $ sudo find / -name "*docker*" -exec `rm -rf` {} +
      
    
  ### 3) 쿠버네티스 calico CNI (Container Network Interface) 삭제
    * CNI는 실제 노드에 파일을 생성하거나 iptable을 수정하는 모듈이기 때문에 kubectl delete -f 만으로는 부족하다
    * 관련 파일과 iptables 룰 및 tunl0 인터페이스등을 모두 삭제해주어야 한다.
    
   * calico 삭제
      $ rm -rf /var/lib/cni/
      $ rm -rf /var/lib/kubelet/
      $ rm -rf /run/calico
      $ rm -rf /etc/cni/
      $ rm -rf /etc/kubernetes
      $ rm -rf /root/.kube/
      $ rm -rf /root/.k8s/
      $ rm -rf /var/lib/etcd/

      $ ip link delete cni0   # 설치한 cni의 네트워크 인터페이스 삭제
      $ ip link delete calico   # 설치한 cni의 네트워크 인터페이스 삭제
      
  ### 4) CRI-O 삭제
    * 설치 확인
      $sudo systemctl status crio
    
  ### 4) 쿠버네티스 삭제
    * 쿠버네티스 관련 파일 삭제
      $ rm -rf /var/lib/cni/
      $ rm -rf /var/lib/kubelet/*
      $ rm -rf /var/lib/etcd
      $ rm -rf /run/flannel
      $ rm -rf /etc/cni
      $ rm -rf /etc/kubernetes
      $ rm -rf ~/.kube
    * 쿠버네티스 관련 패키지 삭제(Ubuntu)
      $ apt-get purge kubeadm kubectl kubelet kubernetes-cni kube* -y
      $ apt-get autoremove
    * kubectl kubelet 삭제
      $ apt remove kubeadm kubectl kubelet
    * 삭제확인
      $ kubectl version -oyaml --client|awk '/gitVersion/{print $2;}'
      
