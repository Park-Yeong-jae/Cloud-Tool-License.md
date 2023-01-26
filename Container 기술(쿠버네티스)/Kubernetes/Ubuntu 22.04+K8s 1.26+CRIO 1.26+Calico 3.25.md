## K8s 1.24버전부터 Docker 런타임의 지원 중단
## dockerShim 지원중단 (docker engine은 지원)
## Containerd(CRI) 및 CRI-O로 K8s 설치.

### Architecture
  * 마스터노드 1대
  * 워커노드 2대
  * nas 1대
  * 총 서버 4대 구축
### 개발 인프라
 * Ubuntu  -> 22.04.1 LTS
 * CRI-O    -> 1.26.1	(CRI containerd 를 설치하려면 k8s v1.26에선 containerd 1.6 이상 필요)
 * K8s 	 -> 1.26.0
 * CNI 	 -> Calico 3.25.0

### 참고자료
 * CRI-O 설치 : https://tech.hostway.co.kr/2022/05/12/1029/
 * K8s 설치 : https://andrewpage.tistory.com/234
 * Calico 설치 : https://projectcalico.docs.tigera.io/maintenance/kubernetes-upgrade

### 인프라 구축 참고사항
 * K8s 와 CRI는 버전이 일치해야한다. (k8s v1.26 버전에서는 기본적으로 CRI APIversion v1을 사용한다)
 * CRI-O는 기본적으로 systemd cgourp을 사용한다.
 * CRI-O는 K8s용 Docker를 대체하는 경량 컨테이너 런타임이며, K8s 추가 도구나 코드 조정 없이 직접 컨테이너를 실행할 수 있습니다

### 취약점
 * 1. Containerd의 임의 호스트 파일 액세스(CVE-2022-23648)
  * containerd  버전 1.6.1, 1.5.10 및 1.14.12에서 발견
 * 2. 컨테이너 이스케이프 취약점(CVE-2022-0811)
  * CRI-O의 컨테이너 이스케이프 취약점은 CVE점수 9.0(심각)으로 작년초 공개
 * 3. execSync 요청을 통한 메모리 고갈 방식의 노드 DOS(CVE-2022-1708)
  * CRI-O 컨테이너 런타임의 취약성으로 CVE 점수는 7.5(높음)다

<hr>

## MASTER & WORKER 작업

## 방화벽 설정
systemctl stop ufw   
systemctl disable ufw   

## root 계정으로 진행.
sudo su 

## swap 영역 비활성화 
sed -i '/swap/d' /etc/fstab   
swapoff -a   

## iptables 설정 (K8s는 iptables를 이용하여 pod 간 통신)
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf   
overlay   
br_netfilter   
EOF   

modprobe overlay   
modprobe br_netfilter

## 요구되는 sysctl 파라미터 설정
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf   
net.bridge.bridge-nf-call-iptables  = 1   
net.ipv4.ip_forward                 = 1   
net.bridge.bridge-nf-call-ip6tables = 1   
EOF   

## 재부팅간에도 유지
sysctl --system

## CRI-O 설치
OS=xUbuntu_22.04   
VERSION=1.26   

echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list   
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -   
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -

apt-get update

apt-get install -y cri-o cri-o-runc

systemctl daemon-reload   
systemctl enable crio --now   
systemctl status crio   

apt install cri-tools   

## CRI-O 구축 끝

<hr>

## Kubernetes 구축 시작
## MASTER & WORKER 작업

apt update   
apt-get install -y apt-transport-https ca-certificates curl   

## 구글 클라우드 public signing key 다운로드
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

## K8s apt repository 추가
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update   
apt-get install -y kubelet kubeadm kubectl   
apt-mark hold kubelet kubeadm kubectl   
systemctl start kubelet && systemctl enable kubelet   

## MASTER 에서만 작업
## init
kubeadm init --apiserver-advertise-address [마스터 IP] --pod-network-cidr=192.168.0.0/16 --cri-socket /var/run/crio/crio.sock

## CNI 설치 (calico)
## Tigera calico 연산자 및 사용자 정의 자원 정의 설치
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml

## 필요한 사용자 정의 자원을 작성하여 calico 설치
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml

## status running 될 때 까지 기다림
watch kubectl get pods -n calico-system

## 파드에서 스케줄 할 수 있게 마스터의 taint 제거
kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-

## kubectl get all --all-namespaces 보면 coredns Status Running으로 변경됨을 확인.

## Calicoctl 설치 (바이너리로 설치)
curl -L https://github.com/projectcalico/calico/releases/download/v3.25.0/calicoctl-linux-amd64 -o calicoctl   
chmod +x calicoctl && mv calicoctl /usr/bin

## kubectl 사용을 위한 K8s 클러스터 인증서 (root 계정에서 빠져나와서 진행)
 * ctrl + D = root 계정 로그아웃 진행   
mkdir -p $HOME/.kube   
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config   
sudo chown $(id -u):$(id -g) $HOME/.kube/config   

## root 계정에서 사용을 하려면 export 하면됨 (root 계정에서 사용은 권장사항은 아님)
 * export KUBECONFIG=/etc/kubernetes/admin.conf

<hr>

## WOKER NODE 에서만 작업
## join (Join  (init 할 때마다 바뀜주의)
kubeadm join **** --token *** \
        --discovery-token-ca-cert-hash **** --cri-socket /var/run/crio/crio.sock


<hr>

## 오류정리

 * Root 계정에서 나와서 x509: certificate signed by unknown authority 오류
  * 인증서오류
mkdir -p $HOME/.kube      
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config      
sudo chown $(id -u):$(id -g) $HOME/.kube/config   
 
 * root 계정에서 인증서 오류
  * 인증서오류
  * export KUBECONFIG=/etc/kubernetes/admin.conf


 * The connection to the server localhost:8080 was refused - did you specify the right host or port?
 * docker 와 CRIO 중복 오류
 * kube init 시 뒤에 --cri-socket /var/run/crio/crio.sock 이걸 붙여줌으로써 socker 지정해줌
