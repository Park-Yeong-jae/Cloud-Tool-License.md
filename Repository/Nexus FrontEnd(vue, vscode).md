## 폐쇄망 환경 / docker에 nexus / VScode , vue 환경

## 온라인 환경에서 구축

### 1. 프로젝트의 package-lock.json, yarn.lock, node_modules, .yarnrc 중에 있는건 다 지워준다.
  * npm 으로 모듈을 받았을경우 package-lock.json 생성
  * yarn 으로 모듈을 받았을경우 yarn.lock 생성
  * 어찌됐던 node_modules 은 있을것이다.
  * 다 지워주자

### 2. 프로젝트의 루트 경로에 .yarnrc 파일 생성후 아래 내용을 넣어준다.
  * .yarnrc 파일생성
  * .yarnrc 내용 입력   
     yarn-offline-mirror "./npm_packages"    // 패키지 명은 하고싶은대로    
     yarn-offline-mirror-pruning true   
     
### 3. yarn 캐시 삭제
  * 설치가 되어있지 않을 경우 설치를 해준 후 캐시 삭제. 후 다시 생성   
    // 아래 명령어를 통해 yarn의 설치 유무 확인
    $ yarn --version

    // yarn이 설치되어있지 않은 경우 설치
    $ npm install -g yarn

    // yarn 캐시 삭제
    $ yarn cache clean

    // npm_package 를 파일로 받기위한 yarn install
    $ yarn install
  * 여기까지 진행을 하게 되면 프로젝트의 루트 경로에 npm_packages 디렉토리에 *.tgz 파일들이 생성된다
  * 해당 파일들이 패키지가 되는 것이고 해당 패키지로 모듈을 만드는것이다.
  * 그래서 지금 뽑아낸 파일들을 폐쇄망 환경의 nexus에 올려야 한다.
  * 폐쇄망에 node.js 설치파일도 함께 반입해야 한다.
  * 폐쇄망에서 yarn으로 환경구성을 하려면 yarn.js 파일을 받아서 반입을 하면 된다.

### 4. 여기까지가 온라인 환경에서의 준비과정.

<hr>

## 오프라인 환경 구성

### 1. 넥서스에 npm(hosted) 로 Repository 하나 만들어주고 추가 설정
  * 나는 'npm-private' 만들겠다.
  * 생성후 톱니바퀴 클릭 > Security > Realms 클릭
  * Availabel 에 있는 npm Bearer Token Realm 클릭(클릭하게되면 Active쪽으로 넘어간다)
  * 하단 save 클릭
  * 여기까지가 nexus 설정 완료

### 2. node.js 설치 파일을 실행하여 node.js 설치

### 3. npm설정 (git bash)
  * 가장먼저 auth 토큰키를 발급 받아야 한다.
    $ echo -n admin:1234 | openssl base64
    // 해당 명령어는 단순히 'admin:1234' 를 base64로 인코딩 한것이다.
  * 발급받은 토큰 키 복사해놓기 (YWRtaW46MTIzNA==)
  * C:\Users\Yeongjae Park 해당위치의 .npmrc 폴더를 메모장으로 열어준다(없으면 만들어주기)
    registry=http://localhost:8081/repository/npm-private/  //nexus 저장소 url
    strict-ssl=false              // ssl 인증서 검사를 하지 않겠다
    auth=YWRtaW46MTIzNA==         // 아까 복사한 npm 계정의 토큰 키값
    always-auth=true              // 모든 npm 명령에 위의 토큰 키값을 보낸다
    
 ### 4. F/E 프로젝트에 npm_packages 폴더를 넣어준다. (위에서 작업했음)
 
 ### 5. 프로젝트 루트경로에 스크립트 파일을 만들어준다.
  $ vi npmImport.sh
  // 해당 명령어를 통해 shell script 파일 생성
  
  #!/bin/bash

  REPOSITORY="http://localhost:8081/repository/npm-private/"
  PACKAGES_PATH=./npm_packages

  npm login --registry=$REPOSITORY

  for package in $PACKAGES_PATH/*.tgz; do
          npm publish --registry=$REPOSITORY $package
  done
  
### 6. package.json 파일 하단에 publishing 할 저장소를 적어준다
    "publishConfig": {
    "registry": "http://localhost:8081/repository/npm-private/"
    }
    
### 7. .sh 스크립트를 실행시킨다

### 8. userID / userPW / Email 입력을 하라고 하면 그냥 하면된다.


이상태로 npm install 하면 되는데 안될때가 많으니

1. npm install --force
2. npm install --legacy-peer-deps 명령어를 통하여 npm install 을 해주면 된다.
그래도 오류가 발생하면 packages.json 에서 오류나는 dependency를 찾아 nexus와 비교하여 존재하는 버전으로 명시후 다시 install을 해주면 된다


## 참고
https://github.com/jerry950208/study_development/blob/main/Repository/Nexus/Nexus-FE.md

