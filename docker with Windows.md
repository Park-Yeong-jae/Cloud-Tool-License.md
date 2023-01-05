## 키보드 Windows 클릭 후 

## Microsoft store 앱 검색하여 실행

## Windows terminal 다운로드, 실행 시 관리자 권한으로 
  * windows terminal이 나오면서 기존의 cmd를 벗어나 더 나은 작업 환경으로 개선됨
  
## Windows Terminal 초기 셋업
  * DISM 명령어로 WSL2 사용 준비 작업 진행
  * dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  * dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

## 컴퓨터 재시작 및 windows terminal 재실행

## 리눅스 배포판인 Ubuntu Microsoft store에서 설치

## Ubuntu 터미널 하나가 실행되고, 사용자 이름과 pwd 설정

## wsl -l로 설치확인
![image](https://user-images.githubusercontent.com/96723249/206599667-20206fac-2d85-4a60-99cb-864bffef3362.png)

## WSL version2로 확인 및 적용
  * wsl -l -v 입력 후 version 확인
  * version 1이라면 2로 설정 (새로 설치하는 모든 배포판에 적용되도록 기본값을 변경)
  * wsl --set-default-version 2
  * wsl -t Ubuntu 으로 ubuntu 종료 후 자동 재실행
  * wsl -l -v로 version 재확인

## ▼ 참고문헌 wsl 셋업 및 uibuntu 설치 (그 이후는 docker desktop을 설치하므로 Pass)
  * https://www.44bits.io/ko/post/wsl2-install-and-basic-usage
<hr>
<hr>

## Ubuntu 내에서 Docker 설치
  * //필요 패키지 설치
  * sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
  * sudo apt-get update
 
  * //docker 공식 GPG 키
  * curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
 
  * //docker stable repo 사용
  * sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 
  * //docker 설치
  * sudo apt install docker-ce docker-ce-cli containerd.io
  * sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  * sudo chmod +x /usr/local/bin/docker-compose

## docker 설치 확인
  * docker -v
  * Docker daemon running? 이 뜰텐데 이 땐 $sudo service docker start로 도커를 실행하면 된다.
  
## ▼ 참고문헌 Ubuntu 내에서 docker 설치 (docker desktop X)
  * https://hahahoho5915.tistory.com/48
  
