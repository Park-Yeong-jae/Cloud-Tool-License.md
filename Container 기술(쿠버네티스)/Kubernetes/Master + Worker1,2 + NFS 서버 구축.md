## Master + workder1 + worker2 구성은 상위 폴더에 정리되어있다.
  * 여기서는 NFS 서버 구축한다.

## 1. NFS 서버 패키지 설치 (NFS 노드에서)
  * nfs-kernel-server 패키지를 설치하면 nfs 구축에 필요한 nfs-common 패키지, rpcbind 등을 포함하고 있어서 따로 설치할 필요가 없다.
    * $ sudo apt install nfs-kernel-server
  * 설치확인
    * $ dpkg -l|grep nfs
    * $ dpkg -l|grep rpc

## 2. 공유할 디렉터리 만들기 (NFS 노드에서)
  * 디렉터리 생성 
    * $ mkdir /nfsdir
  * 공유파일 생성 및 테스트
    * $ cat > /nfsdir/nfstest
    * Hello NFS 입력
    * Ctrl + D
  * Hello nfs 테스트
    * $ cat /nfsdir/nfstest
  * 권한수정
    * $ chmod 777 /nfsdir/*

## 3. 허용할 호스트 IP 확인
  * 허용할 VM에 접속 후 IP확인
    * $ hostname -I

## 4. export 설정 (NFS 노드에서)
  * export 수정
    * $ vi /etc/exports
  * 파일 제일 밑에 추가
    * <경로> <허용할 IP>
    * ex) /nfsdir 10.10.0.1

## 5. 방화벽 해제
  * $ systemctl stop firewalld
  * ( $ systemctl stop ufw )
  * 리눅스 방화벽 중 가장 많이 사용되는게 iptables 인데, 이 작업을 간편화 해주는게 ufw이다. firewalld는 CentOS7 부터 iptables를 대체하여 새롭게 나온 패킷 필터링 방화벽

## 6. NFS 서비스 시작
  * 서비스 시작
    * $ systemctl restart nfs-kernel-server
  * 상시 가동
    * $ systemctl enable nfs-kernel-server
  * 확인
    * $ showmount -e

## 7. NFS 클라이언트 테스트 (연결한 Worker node에서)
  * Worker Node에 nfs 사용을 위해 nfs-common 설치
    * $ sudo apt install nfs-common

## 8. 쿠버네티스 명령어를 통해 NFS를 통한 볼륨 마운트 테스트 (Master node에서)
  * nfs경로를 nginx파드 경로에 마운트됨을 확인하기
  * master node 접속
    * $ vi pv.yaml
    
![image](https://user-images.githubusercontent.com/96723249/214499674-37e5cfdf-279e-4f19-888f-0c4a54f4e9ed.png)

    * $ vi pvc.yaml
![image](https://user-images.githubusercontent.com/96723249/214499832-857f4cd5-dfe2-46f6-9797-b5e28734757c.png)

    * $ vi pod.yaml
![image](https://user-images.githubusercontent.com/96723249/214499868-c427c6be-36f5-4f82-bb0e-df357fa9ef04.png)

  * NFS 서버에서 /nfsdir에 아무 파일을 추가한 후 아래 명령 실행
    * $ kubectl exec -it mypod --bash
    * mypod내에 마운트한 경로인 /var/www/html 디렉토리를 확인해보면 추가된 것을 볼 수 있음
