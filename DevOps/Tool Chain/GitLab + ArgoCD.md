## 사전 작업 1 (Local jar파일을 docker 서버로 복사)

- WSL (windows 하위 Linux)에 Ubuntu , Docker 설치
- Local intelliJ와 같은 framework에서 Hello, world Project 생성 후 build 하여 jar 파일생성
- jar 파일을 WSL Docker 에 복사 (WSL은 windows 폴더에서 바로 확인이되며 FTP 없이 복사 가능)
    - WSL이 아닌 타 서버의 경우 **scp로** 로컬 > 서버로 복사하는 방법,
    - **Filezila(FTP)로** SSH 연결을 통해 jar 파일을 복사하는 방법이 있다.
    - !! 여기서는 1번 방법을 채택

## 사전작업 2 (GitLab의 project를 command에서 git clone 후, build 하여 jar파일 생성)

- GitLab에 프로젝트 Push
- 프로젝트에서 git clone
- Command에서
    - $ git clone [http://10.00.00.00:31004/msa/msa1.git](http://10.10.30.2/msa/msa2.git)
    - username / pwd로 로그인하면 project clone 됨을 확인
- project 내부폴더의 gradlew 에 권한을 주고 gradlew 빌드
    - $ chmod +x ./gradlew
    - $ ./gradlew build
- jar 파일 확인 및 구동, 접속
    - $ cd build/libs/ 폴더에 jar 파일 확인
    - $ java -jar ~~SNAPSHOT.jar 파일 구동
    - IP:8080 접속하여 project 확인
- jar 파일 밖으로 복사
    - $ cp ~SNAPSHOT.jar (옮기고자 하는 경로)

## 공통 작업

## Dockerfile 작성 (jar 파일 위치랑 같은곳에 위치)

```docker
FROM openjdk:8-jdk-alpine3.8
ARG JAR_NAME=helloworld-0.0.1-SNAPSHOT.jar
COPY ${JAR_NAME} msa1.jar
ENTRYPOINT ["java","-jar","/msa1.jar"]
```

FROM :  이미지를 생성할 때 사용할 기반 이미지입니다. 

ARG : 변수 선언

COPY : 실행할 jar파일을 Docker 컨테이너 내부에 hello1.jar라는 이름으로 복사합니다. 상대 경로로 위치도 같이 설정 가능합니다.

ENTRYPOINT : 컨테이너가 시작될 때 실행할 스크립트 혹은 명령을 정의

- alpine 이미지 용량이 작기 때문에 alpine 이미지를 베이스로 한다.

## Image 빌드

$ docker build -f dockerfile-msa2 -t (docker hub repository명:tag) .

- -t : 특정 이름으로 이미지를 빌드하는 옵션
- . : Dockerfile의 경로 (.은 현재경로)

$ docker images 로 빌드된 이미지 확인

- 베이스 이미지로 다운받은 jdk 8-alpine 과 빌드한 이미지가 있으면 정상.
- 생성된 이미지의 repository는 docker hub repository의 이름과 같아야 한다.

## Docker hub에 Push

- $ docker push (docker hub repository (계정/repository) 형식임):tagname
- Docker hub에서 이미지 올라갔는지 확인

## Docker hub에서 push한 이미지를 pull 받고 실행하여 확인

- Docker Hub의 Tags에서 올린 이미지 pull
    - $ docker pull (docker hub 계정/repository):msa2

## Image를 docker run하여 container 생성

$ docker run -d --name msa1 -p 9001:8080 (repository:TAG)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c9bac3b7-bbc9-4e70-8996-461b1e820329/Untitled.png)

- -d : 백그라운드에서 실행
- -p : 호스트와 컨테이너같의 port 바인딩을 위한 옵션 (ex) -p (host port):(container port))
- —name : 이름을 지정해준다 (여기에선 잘 안나오지만 하이퍼가 2개임)

$ docker ps 로 생성된 컨테이너 확인

## ※ docker 내부에서 java 코드 수정 후, 프로젝트 적용

- src/main/java/com/example/helloworld/controller 에서 java 파일 vi로 실행
- 내용 수정 후 저장
- 수정 내용 적용을 위해선 재차 빌드 해야됨
    - $ ./gradlew clean
    - build 폴더 삭제된 것 확인
    - $ ./gradlew build
    - 수정된 jar파일을 위의 일련의 과정을 재차 진행.
    - 확인만 위해서는 생성된 jar파일 java -jar로 실행해서 확인 가능.

## 참고문헌

- dockerfile : [https://ssyoni.tistory.com/m/22](https://ssyoni.tistory.com/m/22)
- Push : [https://techblog-history-younghunjo1.tistory.com/246](https://techblog-history-younghunjo1.tistory.com/246)
- image build : [https://ssyoni.tistory.com/m/22](https://ssyoni.tistory.com/m/22)
