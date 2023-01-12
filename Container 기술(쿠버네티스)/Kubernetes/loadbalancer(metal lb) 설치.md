## Kubernetes LoadBalancer (metal lb)설치
  * 쿠버네티스 클러스터에 존재하는 Pod 서비스를 외부로 노출시키기 위한 가장 원시적인 방법은 NodePort를 이용하는 것입니다. 하지만 NodePort는 인스턴스의 IP가 변경되면 해당 서비스에도 이를 반영해야합니다. 따라서 Cloud 벤더에서는 LoadBalancer나 Ingress 타입을 통해 서비스를 노출할 수 있도록 지원한다.

## 여기서는 온 프레미스 환경에서 LoadBalancer 타입을 지원하기 위한 MetalLB를 설치하는 방법을 다루고자 한다. ((23년 1월 12일, MetalLB는 버전에 따라 설치 결과가 조금씩 달라서 날짜를 써놓음)
  
<hr>

## 1. strict ARP mode 활성화하기
 * $ kubectl edit configmap -n kube-system kube-proxy
 * ...
 * apiVersion: kubeproxy.config.k8s.io/vialpha1   
   kind: KubeProxyConfiguration   
   mode: "ipvs"   
   ipvs:   
    strictARP:true //이 부분을 수정한다.   
 * ...

## 2. Metal LB 설치 (공식 홈페이지 설치가이드 (매니페스트에 의한 설치))
  * Metal LB Components를 생성   
    $ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml

## 3. 설치 후 확인
 $ kubectl get -n metallb-system pod   
   잘 설치되었다면 아래와같이, metallb-conroller와 metallb-speaker pod가 Running 상태로 보인다. (25~30초정도 걸림)
   
![image](https://user-images.githubusercontent.com/96723249/211991226-73454cfe-e1d0-49cf-80cb-f068026c81c4.png)

## 4. IP Address Pool 설정
 $ cat my-network.yaml   
![image](https://user-images.githubusercontent.com/96723249/211991536-989e3ab3-f6a0-4d92-b2cb-9101aa6269f5.png)

 $ kubectl apply -n metallb-system -f my-network.yaml     
![image](https://user-images.githubusercontent.com/96723249/211991681-f20abe87-212c-48f2-812c-6350c05b544d.png)

## 5. Example App를 이용하여 MetalLB 동작 확인
 $ cat my-example.yaml   
![image](https://user-images.githubusercontent.com/96723249/211991853-f782af4f-cc0c-45cc-abca-cdfd0c959e69.png)

 $ kubectl create ns ns1
 
 $ kubectl apply -n ns1 -f my-example.yaml
 
 $ kubectl get -n ns1 svc   
![image](https://user-images.githubusercontent.com/96723249/211992003-dbc2818d-7591-4605-b8d1-f3d5059952d1.png)
 
 * 위 결과에서 EXTERNAL-IP에 적절한 IP Address가 출력되면 잘 되는것.   
 * 그리고 아래와 같이 실제로 MetalLB가 할당해준 EXTERNAL-IP 주소를 통해 Networking이 되는지 확인한다.

## 6. MetalLB가 할당해준 IP주소 Networking 확인
 $ curl <EXTERNAL-IP>
  

## 참고문헌
https://cla9.tistory.com/94   
https://andrewpage.tistory.com/23
