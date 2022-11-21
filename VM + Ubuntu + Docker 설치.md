
## 1. Docker 오래된 버전 삭제하기 (첫 설치라면 skip)
  * sudo apt-get remove docker docker-engine docker.io containerd runc
  
## 2. apt가 https를 통해 패키지를 사용할 수 있도록 몇가지 패키지 설치
 * apt-get install apt-transport-https ca-certificates curl software-properties-common

## 3. Docker GPG Key를 추가 (공식 도커 저장소용)
 * curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add   
  *(※ Docker Repository를 접근하기 위한 인증key를 추가하는 작업입니다.)
  
## 4. apt소스에 도커저장소(stable repository)를 추가
 * add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"   
  *(Docker Repository의 정보를 갖고 오는 작업입니다.)

## 5. 패키지 데이터베이스 업데이트
 * apt-get update

## 6. 설치가 가능한 docker-ce 버전 확인
 * apt-cache madison docker-ce   
  (docker-ce: docker Community Edition)
  
## 7. 도커 저장소 사용 및 Docker-ce 설치
 * sudo apt-get install docker-ce docker-ce-cli containerd.io
  
## 8. 설치 확인
 * docker --version

## 9. 설치 확인 후 작업 및 실행 상태 확인
 * systemctl stop docker
 * systemctl start docker
 * systemctl status docker

## 10. Docker는 root 권한에서 실행해야하므로 권한을 주어야 함.
  * sudo usermod -aG docker $USER   ($USER은 현재 접속자를 뜻함)
  *  logout (logout을 했다가 재로그인하면 적용됨)

## 11. Hello world 실행
  * sudo docker run hello-world

## 12. 컴퓨터 부팅시 도커 자동실행 설정
  * systemctl enable docker

## ubuntu에서 docker 완전 삭제
 * sudo apt-get purge -y docker-engine docker docker.io docker-ce
 * sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce 

 * sudo rm -rf /var/lib/docker /etc/docker
 * sudo rm /etc/apparmor.d/docker 
 * sudo groupdel docker 
 * sudo rm -rf /var/run/docker.sock

<hr>   
<hr>   

## 오류 및 해결방안

## 1. GPG 등록 후 apt 사용 시 문제발생
( invalid value set for option signed-by regarding source ~ )
 * 해당위치에 있는 docker.list 파일 삭제
  ① cd /etc/apt/sources.list.d
  ② sudo rm -rf docker.list
  ③ 삭제하니 , docker 설치가 안됨.
  ④ 결국 아래 방법에 따라 우분투 재설치로 해결.
   https://shanepark.tistory.com/237 이 방법대로 재설치, 해결
   
 ## 2. apt --fix-broken install 오류
  * 의존성이 맞지않는 경우
   ①. 컨테이너 의존성을 설치한다
    * apt install containerd
   ② 의존성 패키지 자동설치
    * sudo apt-get -f install -f
    
  ## 3. Docker 권한주는게 안될 경우
   * 최초 접속 후 sudo su
   * 비밀번호 입력
   * cd ~
   * 순서로 명령어를 입력하여 root 계정으로 접근

<hr>   

  ## 번외 : Kubernetes 와 Dcoker compose의 차이   
   ①. 쿠버네티스   
   : 쿠버네티스 클러스터 내에서 컨테이너를 생성하고 관리하고 yaml 파일에 정의된 내용을 바탕으로 클러스터 내에서 오브젝트를 만들고 컨트롤러를 만든다.   

   ②. Docker compose   
   마찬가지로 yaml 파일에 정의된 내용을 바탕이지만, 클러스터 내에서가 아닌 Host 컴퓨터 내에 컨테이너를 만든다.
   
   ## 번외 : CLI Tool
   ① apt : 패키지를 install, upgrade , clean 할 수 있고 패키지 다루는 툴   
   ② curl : curl [options] [URL...]의 형식, 원격 호스트에서 또는 원격 호스트로 데이터를 전송할 수 있는 명령줄 도구 문제 해결, 파일 다운로드 등에 유용   
