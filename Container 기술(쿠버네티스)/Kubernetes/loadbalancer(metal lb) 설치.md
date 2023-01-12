## Kubernetes LoadBalancer (metal lb)설치
  * 쿠버네티스 클러스터에 존재하는 Pod 서비스를 외부로 노출시키기 위한 가장 원시적인 방법은 NodePort를 이용하는 것입니다. 하지만 NodePort는 인스턴스의 IP가 변경되면 해당 서비스에도 이를 반영해야합니다. 따라서 Cloud 벤더에서는 LoadBalancer나 Ingress 타입을 통해 서비스를 노출할 수 있도록 지원한다.

  * 온 프레미스 환경에서 LoadBalancer 타입을 지원하기 위한 MetalLB를 설치하는 방법을 다루고자 한다.

## 1. nginx Deployment 하나 생성
  $ kubectl create deploy nginx --image=nginx

## 2. 생성한 Deployment를 LoadBalancer 타입으로 서비스를 노출시킨다.
  $ kubectl expose deploy nginx --port 80 --type LoadBalancer
  
## 3. 서비스의 상태를 모니터링
  $ watch kubectl get scv
  
![image](https://user-images.githubusercontent.com/96723249/211975182-a5cc70dd-85fe-4989-8af7-834f5cf150c2.png)
  * 기다려도 해당 Service 오브젝트의 External-IP는 Pending 상태입니다. 이는 LoadBalancer가 존재하지 않기 때문입니다.
  
## 4. Service와 Deployment 오브젝트를 제거한다
  $ kubactl delete svc nginx   
  $ kubectl delete deploy nginx
  
<hr>

## 1. Metal LB 설치 (공식 홈페이지 설치가이드 (매니페스트에 의한 설치))
  * Metal LB를 위한 네임스페이스 생성
    $ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
    
  * Metal LB Components를 생성
    $ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
      
  * Secret 생성
    $ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
    
  * 라우팅 처리를 위한 ConfigMap 생성
    $ cat << EOF | kubectl create -f -
      >apiVersion: v1
      >kind: ConfigMap
      >metadata:
      > namespace: metallb-system
      > name: config
      >data:
      > config: |
      >   address-pools:
      >   - name: default
      >     protocol: layer2
      >     addresses:
      >     - 192.168.72.102-192.168.72.103
      >EOF
   
## 2. 설치 확인
  * 기존과 마찬가지로 nginx를 deploy한 다음 LoadBalancer 타입으로 노출시킨 결과 이전과는 다르게 nginx Service 오브젝트의 External IP가 할당된 것을 확인할 수 있음

  $ kubectl create deploy nginx --image=nginx   
  $ kubectl expose deploy nginx --port 80 --type LoadBalancer   
  $ kubectl get svc   

  
  

## 참고문헌
https://cla9.tistory.com/94
