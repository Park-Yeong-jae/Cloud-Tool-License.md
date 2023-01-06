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

## Nexus repository 생성 및 distribution용 repository에 컴포넌트 업로드 >

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
      Recipe : raw (hosted) 선택
      Name : repository 이름 설정 (distribution_raw) 
      deployment policy : Allow redeploy
      
  ### 5. 상단 톱니바퀴옆에 상자 모양을 클릭 후 Browse에서 만들어진 Repository 확인
  
  ### 6. distribution 용 repository에 1번에서 upload component로 바탕화면에 복사해놓은 압축파일 선택
    * 경로 .gradle > wrapper > dists > gradle-7.5.1-bin > 7hahhrw~~ > gradle-7.5.1-bin.zip
    * upload 잘 되어있는지 확인
  
  ### 7. 여기까지 proxy로 사용할 repository를 생성해주었고 distribution용 repository까지 생성
  
<hr>

  ## 프로젝트와 nexus repository와 연결 및 Nexus repository에 dependency와 plugin
  
  ### 1. 프로젝트를 실행
  
  ### 2. 프로젝트 루트경로의 gradle/wrapper/gradle-wrapper.properties 파일을 열어준다
  
  ### 3. distributionUrl 변경
    * 기존에 services.gradle.org에서 자동으로 해당 gradle 버전을 사용했다면 폐쇄망 환경이니 nexus로 경로를 설정해준다
    * distributionUrl=http://localhost:8081/repository/distribution_raw/nexus_test/gradle-7.5.1-bin.zip
    
  ### 4. /setting.gradle 파일 수정
    * 최상단에 아래 코드를 넣어준다
        pluginManagement {                        // 플러그인 사용을 위한 block 설정
          repositories {                          // 저장소 정보를 위한 block 설정
            maven {                             // Gradle, Maven 상관없이 해당 명령어 사용
              url "http://localhost:8081/repository/plugin_proxy/"  // plugin용으로 만든 넥서스 레포지토리 url 
              allowInsecureProtocol true        // 최신 gradle에서 보안을 위해 http 서버를 사용하는 것을 허용하지 않고 있다. 강제로 http 서버를 사용가능하게끔 설정
              credentials {                     // nexus의 인증을 위한 block 설정
                  username "admin"              // nexus의 ID
                  password "1234"               // nexus의 PW
              }
          }
      }
    }
  
  ### 5. build.gradle 파일 수정
    * repositories 안에 mavenCentral() 이 있을텐데, 폐쇄망에서는 mavencentral에서 자동적으로 라이브러리를 가져올 수 없으므로 변경해준다
        repositories {
      maven {
        url "http://localhost:8081/repository/dependency_proxy/"
        allowInsecureProtocol true
        credentials {
          username "admin"
          password "1234"
        }
      }
    }
  
  ### 6. Gradle reload 클릭 (코끼리 모양)
  
  ### 7. 여기까지가 2차 설정이며 Project와 Nexus를 연결하였다. 인터넷이 되는 환경에서 Nexus repository에 Project의 dependency와 plugin을 넣는 작업.
  
<hr>

  ## proxy로 만든 Nexus repository에서 dependency와 plugin을 빼오는 작업
  
  ### 1. https://github.com/LoadingByte/nexus3-exporter 해당 프로젝트를 clone 한다.
  ![image](https://user-images.githubusercontent.com/96723249/209901989-c75644b6-06d4-4ba3-86a3-85583c81c178.png)

  ### 2. IDE로 nexus3-exporter 프로젝트를 열어준다
  
  ### 3. README.md 파일에 보면 Example call 에 있는 양식대로 명령어를 입력해주면 된다.
    양식 : python3 nexus3_exporter.py https://repo.loadingbyte.com maven-releases
    1) 해당 프로젝트는 python이기 때문에 python3를 먼저 시작 > microsoft store > python3.10 버전설치
    2) dependency용 부터 설치
      python3 nexus3_exporter.py http://localhost:8081/repository dependency_proxy
      >>> python3 nexus3_exporter.py {{ nuxusURL }} {{ Repository 이름 }}
    3) plugin용 설치
      python3 nexus3_exporter.py http://localhost:8081/repository plugin_proxy
      >>> python3 nexus3_exporter.py {{ nuxusURL }} {{ Repository 이름 }}
    4) 정상적으로 작동했다면 사진과 같이 두개의 폴더가 생성이 됨 (dependency_proxy , plugin_proxy)
![image](https://user-images.githubusercontent.com/96723249/209902616-af644b16-a7d8-4210-b151-640a6877fef4.png)

  ### 4. 여기까지가 온라인 환경에서의 작업이 끝. (다음엔 인터넷을 끄고 폐쇄망 환경에 있는 Nexus에 dependency와 plugin을 넣는 작업 진행)

<hr>

  ## 온라인 환경에 있는 Nexus와 Project를 연결하여 Nexus exporter로 dependency_proxy와 plugin_proxy 폴더를 빼왔고 + gradle-7.5.1.zip파일까지 총 3개의 파일을 폐쇄망 환경에 반입 신청을 해야함
![image](https://user-images.githubusercontent.com/96723249/209905688-5cf8d9e7-bbf0-4d8b-bf6e-9a62fadaa77f.png)

    * 파일 반입 & 폐쇄망 환경에 & nexus exporter도 없다는 가정하에 작업
    * 위 가정을 하기위해 기존에 만들었던 dependency_proxy 와 plugin_proxy는 삭제하였다 (distribution은 폐쇄망 nexust에 있다는 가정으로 유지)

  ### 1. dependency repository와 plugin repository를 recipe: maven(host)로 만든다 (폐쇄망 Nexus)
    * nexus에서 Repository는 3가지 종류가 있다.
      proxy : 프록시 역할을 하여 설정한 repository 에서 nexus를 거쳐 로컬(서버)의 원격 저장소 역할을 한다.(온라인 환경에서 사용)
      hosted: 로컬(서버)에서 직접 원격저장소에 저장하여 사용한다.(폐쇄망이나 내부망에서 주로 사용)
      group: 여러개의 레포지토리를 논리적으로 묶어 하나의 저장소 처럼 사용을 한다.
    * proxy와 hosted 모두 원격 레포지토리로 사용을 하며 여러사람이 사용하는것이지만 온라인이나 오프라인이냐에 따라 사용이 갈린다.
    
  ### 2. 반입한 dependency용 proxy와 plugin용 proxy 압축을 푼다
  
  ### 3. 해당 폴더에 들어가서 새로만들기 > .sh 파일을 각자의 폴더에 만든다
  
  ### 4. .sh 파일에 아래 스크립트를 기입 후 (dependency용 , plugin용 수정필요) nexus repository를 뽑아낸 폴더에 저장한다. (nexusurl은 proxy url)
    #!/bin/bash
    files="./files.out"
    username="admin"
    password="1234"
    nexusurl="http://localhost:8081/repository/dependency_host/"
    find . -name '*.*' -type f | cut -c 3- | grep "." > $files
    while read i; do
    echo "upload $i to $nexusurl"
    curl -v -u $username:$password --upload-file $i "$nexusurl$i"
    done <$files
  
  ### 5. .sh 파일 만든 위치에가서 git bash 실행 후 커맨드 창에서 스크립트 문 실행
    * dependencyimport.sh // pluginimport.sh 로 파일명 만들었다
    * 실행 문
      ./dependencyimport.sh // ./pluginimport.sh
    * 실행 완료되었으면 해당 파일 위치에 files.out 이라는 파일이 생성되었을거다. (4번에서 설정한 파일이름)
    * Nexus(폐쇄망 환경을 가정한) repository에 가서 host로 만든 repository에 파일들이 잘 올라갔는지 확인한다
    
  ### 6. Project와 Nexus(폐쇄망 환경을 가정한)를 연결해준다
    * build.gradle에서 url을 폐쇄망 Nexus의 host repository를 바라보게끔 수정
      url "http://localhost:8081/repository/dependency_host/"
    * settings.gradle에서 url를 폐쇄망 Nexus의 host repository를 바라보게끔 수정
      url "http://localhost:8081/repository/plugin_host/"
    * gradle > wrapper > gradle-wrapper.properties의 distributionURL을 폐쇄망 Nexus의 repository를 바라보게끔 수정
      (여기서는 distribution 따로 repository 만지지 않았으므로 그대로 두어도 된다. 실무에서는 바꿔줘야함)
      
      
      
## 참고
https://github.com/jerry950208/study_development/blob/main/Repository/Nexus/Nexus-FE.md
      
