
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
