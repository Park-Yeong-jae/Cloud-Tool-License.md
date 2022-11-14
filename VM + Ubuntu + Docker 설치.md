
## 1. Docker 오래된 버전 삭제하기 (첫 설치라면 skip)
  * sudo apt-get remove docker docker-engine docker.io containerd runc

## 2. apt package index 업데이트 및 HTTP를 통해 repository 이용을 위한 package 설치
  * sudo apt update
  * sudo apt-get install -y ca-certificates \curl \gnupg \lsb-release

## 3. docker의 Official GPG key 등록
  * curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

## 4.stable repository를 등록 (apt 패키지 소스리스트에 해당 gpg키 경로를 포함한 저장소 위치를 추가하는 작업)
  * echo \ "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

## 5. Docker 엔진 설치
  * sudo apt-get update
  * sudo apt-get install docker-ce docker-ce-cli containerd.io

## 6. 버전확인으로 설치확인
  * docker --version

## 7. Docker는 root 권한에서 실행해야하므로 권한을 주어야 함.
  * sudo usermod -aG docker $USER   ($USER은 현재 접속자를 뜻함)
  *  logout (logout을 했다가 재로그인하면 적용됨)

## 8. Hello world 실행
  * sudo docker run hello-world

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
   * 순서로 작업하여 root 계정으로 접근

<hr>   
  ## 번외 : Kubernetes 와 Dcoker compose의 차이   
   ①. 쿠버네티스   
   : 쿠버네티스 클러스터 내에서 컨테이너를 생성하고 관리하고 yaml 파일에 정의된 내용을 바탕으로 클러스터 내에서 오브젝트를 만들고 컨트롤러를 만든다.   

   ②. Docker compose   
   마찬가지로 yaml 파일에 정의된 내용을 바탕이지만, 클러스터 내에서가 아닌 Host 컴퓨터 내에 컨테이너를 만든다.
