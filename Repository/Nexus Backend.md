## 폐쇄망 환경 / docker 설치 가정 / Nexus3 / Backend projec

<hr>

## Nexus 설치 및 설정 >

  ### 1. Docker 에서 nexus3 설치   
    // 도커 이미지 가져오기   
    $ docker pull sonatype/nexus3   
  
  ### 2. 이미지 빌드하여 nexus container 만들기   
    $ docker run -d -p 8081:8081 --name nexus sonatype/nexus3   
    // -d : 백그라운드로 실행 => 빌드되는 동안과 후 에도 cli 창을 사용할수 있다   
    // -p : 포트 포워딩 => <호스트 포트>:<컨테이너 포트>   
    // --name : 이름설정    
  
  ### 3. 로컬서버 기준 'http://localhost:8081' 접속 시 넥서스가 실행 됨(1~2분 정도 소요됨)

  ### 4. 상단 우측 sign in 버튼 클릭   
    * 초기 ID 값과 PW 값을 넣어야하는데 docker 에서 찾아야한다    
    $ docker exec -it nexus /bin/bash     // 백그라운드로 실행시킨 'nexus' 컨테이너의 내부의 '/bin/bash' 접속   
    $ cd sonatype-work/nexus3           // sonatype-work/nexus3 directory 접근   
    $ cat admin.password                // cat 명령어를 통하여 admin.password 파일 내용 확인 (아이디는 admin, bash-4.4$ 앞 부분이 비밀번호이다.)
    * nexus에서 로그인 후, 비밀번호를 설정해준다
    
  ### 5. nexus 설치부터 사용할 준비는 완료

<hr>

## Project의 .gradle 파일을 넥서스 레포지토리로 올리는 작업 >

  ### 1. windows에서 C:\Users\사용자 폴더.gradle\wrapper\dists\gradle-7.5.1-bin\8ku47ia7tvi8fe9qdzvy2faf8\gradle-7.5.1-bin.zip 해당파일을 복사해서 바탕화면에 붙여넣기 해준다.
    * 여기에서 아무거나 복사 붙여넣기가 아니고 넥서스를 사용할 프로젝트의 루트 경로에서 gradle/wrapper/gradle-wrapper.properties파일에 distributionUrl에 보면 https://services.gradle.org/distributions/gradle-7.5.1-bin.zip 이런형태로 있는데 gradle-7.5.1-bin.zip 파일을 찾아서 복사 붙여넣기 하면된다. (압축파일 형태로 되어있음)
    
  ### 2. C:\Users\사용자폴더.gradle 을 지워준다.(용량이 커서 지우는데 오래걸릴것 같을경우 파일명앞에 _를 붙여서 .gradle을 몾찾게 해준다.)
  
  ### 3. nexus3 를 docker로 실행시켜준다 (위에 실행시키는 방법대로)
  
  ### 4. nexus에 로그인해서 톱니바퀴모양을 누른 후 repository 생성
    * distribution용 , dependency용 , plugin 용 총 3개를 만들 예정
    1) Dependency 용 repository 생성
      Recipe : maven2 (proxy) 선택
      Name : repository 이름 설정 (dependency_proxy)
      Remote Storage는 input 창에 적혀있는 메이븐 센트럴 주소  (https://repo1.maven.org/maven2/)
    
    2) Plugin 용 repository 생성
      Recipe : maven2 (proxy) 선택
      Name : repository 이름 설정 (plugin_proxy)
      remote storage는 plugin 용 이기 때문에 주소가 달라진다. (https://plugins.gradle.org/m2/)
      
    3) distribution 용 repository 생성
      Recipe : row (hosted) 선택
      Name : repository 이름 설정 (distribution_row) 
      deployment policy : Allow redeploy
      
  ### 5. 상단 톱니바퀴옆에 상자 모양을 클릭 후 Browse에서 만들어진 Repository 확인
  
  ### 6. distribution 용 repository에 1번에서 바탕화면에 복사해놓은 압축파일 선택
    * upload 잘 되어있는지 확인
  
  ### 7. 여기까지 proxy로 사용할 repository를 생성해주었고 distribution용 repository까지 생성
  
<hr>

  ## 프로젝트와 nexus repository와 연결
  
  ### 1.
