# 구축하고자 하는 아키텍처   									
![image](https://user-images.githubusercontent.com/96723249/202334390-e3dcbebc-4c69-4ab2-a2fb-174fc54c33f7.png)

# 클러스터 서버 구축을 위한 작업
  ## 1. NTP 서버 동기화 (마스터/워커 조금 다름)
    * Network Time Protocol로 클러스터를 구축하면 각 노드들이 통신을 하는데 이 때 각각의 시간이 맞지 않아 발생할 수 있는 위험을 없애기 위함
    * 마스터노드
      * sudo apt install ntp
      * sudo service ntp reload
      * sudo ntpq -p (pool로 설정되어있는 NTP 서버들에서 시간을 받아옴)
    * 워커노드
      * sudo apt install ntp
      * sudo vi /etc/ntp.conf(주석해제된 pool과 server를 주석처리하고 마스터 노드의 ip를 추가)
        * server [마스터IP] 추가
      * sudo systemctl restart ntp (재시작)
      * sudo ntpq -p (NTP 클라이언트가 NTP 서버에 동기화 됨을 확인)
      
  ## 2. 스왑메모리 해제 (마스터/워커)
    * 메모리 스왑이 활성화 되어 있으면 컨테이너의 성능이 일관되지 않을 수 있어서 대부분의 쿠버네티스 설치 도구(kubeadm 포함)는 메모리 스왑을 허용하지 않음.
    * sudo swapoff -a
    * sed -i '/swap.img/s/^/#/' /etc/fstab (영구적 스왑메모리 해제)
    
  ## 3. Docker install
    * 상위 폴더의 install 
    
  ## 4. 노드간 통신을 위한 브릿지 설정 (마스터/워커)
    * cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    br_netfilter
    EOF
    * cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    
  ## 5. Docker Daemon systemd 와 cgroup을 systemd로 맞춰주어야 함 (마스터/워커)
    * cat <<EOF | sudo tee /etc/docker/daemon.json
    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m"
      },
      "storage-driver": "overlay2"
    }
    EOF
    * sudo systemctl enable docker
    * sudo systemctl daemon-reload
    * sudo systemctl restart docker
    
  ## 6. kubeadm, kubelet and kubectl 설치 (마스터/워커)
    * sudo apt update
    * sudo apt-get install -y apt-transport-https ca-certificates curl
      *** 오류 : could not get lock /var/lib/dpkg/lock-frontend. It is held process~
            * sudo reboot 하면 됨
    * sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
    * echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    * sudo apt-get update
    * sudo apt-get install -y kubelet kubeadm kubectl
    * sudo apt-mark hold kubelet kubeadm kubectl
    * systemctl start kubelet && systemctl enable kubelet
    
  <hr>
  <hr>
  
# 클러스터 실행
  ## 1. 마스터노드 Kubeadm init (마스터)
    * sudo kubeadm reset (처음 설치하면 생략)
      * reset 시, unset KUBECONFIG 선행
      * reset 시,export KUBECONFIG=/etc/kubernetes/admin.conf 선행
    * sudo kubeadm init --apiserver-advertise-address [마스터 IP] --pod-network-cidr=192.168.0.0/16
      * 마스터 IP는 ifconfig를 했을 때 ens4의 inet 주소이다
      * 워크노드가 붙을 마스터 IP와 Pod의 내부 네트워크 지정
        * pod-network 
          calico기반 : pod-network-cidr=192.168.0.0/16
          Flannel기반 : pod-network-cidr=10.244.0.0/16
          Cilium 기반 : pod-network-cidr=192.167.0.0/16
      *** 오류 1) master 노드는 Core가 최소 2개 이상 필요한데 1개로 설정했을 시
                * Setting 들어가서 Core 갯수 수정
               2) Core를 재설정 할 경우 메모리스왑 해제가 풀릴 수 있음
                * sudo swapoff -a
               3) 오류 : error execution phase preflight: [preflight] Some fatal errors occurred:
                  * 기본 Docker 구성은 CLI를 제거함
                  * sudo rm /etc/containerd/config.toml
                  * sudo systemctl restart containerd
    * init 완료 후 토큰 값 저장
      * kubeadm join [마스터 IP]:~ --token 로 시작하는 라인 전부
      
  ## 2. kubectl 명령어 수행을 위한 명령어 (마스터)
    * (재설치 시 선행작업 : unset KUBECONFIG)
    * (재설치 시 선행작업 : export KUBECONFIG=/etc/kubernetes/admin.conf)
    * mkdir -p $HOME/.kube
    * sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    * sudo chown $(id -u):$(id -g) $HOME/.kube/config
    * kubectl get nodes 로 확인
    
  ## 3. 마스터 노드 init 시 생성된 토큰값으로 Join (워커)
    * 마스터노드에서 init 시, 토큰이 재발급되니 sudo kubeadm reset을 하고 진행
      * reset 시, unset KUBECONFIG 선행
      * reset 시,export KUBECONFIG=/etc/kubernetes/admin.conf 선행
    * kubeadm join ~~ 으로 시작하는 토큰 값 전부 복붙하여 join
      * 오류 : error execution phase preflight: [preflight] Some fatal errors occurred:
                * 기본 Docker 구성은 CLI를 제거함
                * sudo rm /etc/containerd/config.toml
                * sudo systemctl restart containerd
    * join 재시도
    * Master Node 에서 kubectl get nodes로 Join 확인
    
  ## 4. Pod 네트워크 애드온 설치 (마스터) - kubectl get nodes 시 Not Ready 해결
    ( Calico, flannel, weave net 등 많은 종류가 있는데 여기서는 calico 설치 (weave net이 안됨))
    * wget https://docs.projectcalico.org/manifests/calico.yaml (calico 설정파일 내려받기)
    // 생략 * sed -i -e 's?192.168.0.0/16?192.168.1.0/24?g' calico.yaml (기본값 192.168.0.0/16 에서 우리가 설정한걸로 변경)
    * kubectl apply -f calico.yaml (구축된 클러스터에 실시간 반영)
    * kubectl get pods --namespace kube-system (네트워크 생성 및 적용 확인)
      * ((출력 결과에 calico-로 시작하는 노드들의 상태를 보며 Init 단계를 지나 PodInitializing 혹은 ContainerCreating을 거쳐 최종적으로 Running까지 확인.
Ready 의 값을 보면 1/1을 제대로 만족하는지 확인하면 된다.))
    * 제대로 나오지 않을 땐 apt update 하고 재시도
    
<hr>
<hr>

# Cluster 아키텍처
![image](https://user-images.githubusercontent.com/96723249/202334290-226e2f6b-783e-4c02-babb-a905dabecf52.png)
