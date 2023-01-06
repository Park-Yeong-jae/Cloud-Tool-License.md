## 빌드서버와 배포서버가 분리되 었을 경우 CI/CD 구성도
![image](https://user-images.githubusercontent.com/96723249/210200986-69720680-e5c6-42db-ad04-736df4f385b9.png)

## 단독 서버에서의 CI/CD 구성도 (여기서는 이 구성도로 진행될 예정)
![image](https://user-images.githubusercontent.com/96723249/210201013-c02f4d61-fd9b-4a64-8293-831082a383c2.png)
  1. 개발자가 코드를 깃허브에 Push 한다
  2. 깃허브가 webhook을 통해 Jenkins로 요청을 보낸다
  3. Jenkins는 Pipeline script에 따라 도커를 사용하여 빌드 및 배포를 한다.

<hr>

### Docker에 jenkins 이미지 다운로드 및 컨테이너 실행
  * 도커허브로 부터 jenkins/jenkins:lts 이미지 pull
    $ docker pull jenkins/jenkins:lts
  * 젠킨스 컨테이너 실행
    $ docker run -d --name jenkins -p 8080:8080 -v /jenkins:/var/jenkins_home -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins/jenkins:lts
    > 핵심은 바인딩(-v) 옵션
      * -v /jenkins:/var/jenkins_home    
        : 젠킨스 컨테이너의 설정을 호스트 서버와 공유함으로써, 컨테이너가 삭제되는 경우에도 설정을 유지할수 있게 해줍니다.
      * -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock   
        : 젠킨스 컨테이너에서도 호스트 서버의 도커를 사용하기 위한 바인딩입니다.
          이렇게 컨테이너 내부에서 설치없이, 외부의 도커를 사용하는 방식을 DooD(Dock out of Docker)[1] 라고 합니다.
      * -p 8080:8080   
        : 젠킨스의 defaul port 인 8080 을 호스트 서버와 매핑해줍니다.
  
  * 도커 컨테이너의 실행 상태 확인   
    $ docker ps

### Jenkins 실행
  * localhost:8080 접속
![image](https://user-images.githubusercontent.com/96723249/210201896-1c542c22-f087-40de-bdfe-e5bed1a91071.png)
  
  * 위에서 필요한 키 값은 docker log에서 확인이 가능
    $ docker logs jenkins
![image](https://user-images.githubusercontent.com/96723249/210202021-9ea68c24-9284-476f-8dfc-c150975835c3.png)

  * install suggested plugins / 설치 / 계정생성 / 실행 
![image](https://user-images.githubusercontent.com/96723249/210202113-e00a0dff-b00a-44a2-b2c5-8dd770e7394e.png)
![image](https://user-images.githubusercontent.com/96723249/210202524-806f7986-84c1-458c-8dd5-394ddbd6396d.png)
![image](https://user-images.githubusercontent.com/96723249/210202531-6ed7055a-2733-43fc-afcd-13fbd53b2356.png)
![image](https://user-images.githubusercontent.com/96723249/210202581-9ebc210b-d9f2-426d-9a7a-d45ff3b02dee.png)

### Jenkins 플러그인 설정
  * 젠킨스 관리 > 플러그인 관리
![image](https://user-images.githubusercontent.com/96723249/210203148-48936ca0-763f-4cbf-bfb1-865501af58ea.png)
  
  * 설치 할 플러그인
    > Blue Ocean 
    
        Dashboard for Blue Ocean   
        Display URL for Blue Ocean   
        Blue Ocean Pipeline Editor   
        autofavorite for Blue Ocean   
        Blue Ocean   
        i18n for Blue Ocean   
        Events API for Blue Ocean   
        Personalization for Blue Ocean   
        Bitbucket pipeline for Blue Ocean   

    > Gitlab
    
        Gitlab   
        Gitlab Authentication   
        gitlab branch source   
        gitlab merge request builder   
        gitlab api   
        gitlab hook 은 다운받지 않아도 됨   

    > 그 외
    
        config file provider   
        SSH   
        Publish Over SSH   
        SSH Pipeline Steps   
        Docker   
        Docker Pipeline   
        nodejs   
      
    > 프론트나 백엔드나 npm을 사용해야하므로 NodeJS 설치   
    
        젠킨스 관리 - Global Tool Configuration - Add NodeJS 클릭 - name:nodejs 설정 후 저장   
![image](https://user-images.githubusercontent.com/96723249/210203715-1262db99-c17e-4f91-b536-bb2fd95eda1f.png)

### Jenkins , GitLab 연동
  1) Gitlab
  
    * Gitlab에서 CI/CD 적용할 프로젝트에 들어가서 access token 설정
    * Spring Boot(B/E) + Vue(F/E) 프로젝트를 사용함
    * gitlab - settings - Acess Tokens
      * name , expires at , Scopes - api 체크 뒤 Create Project access token
    * Access Token이 나오면 잘 저장해둔다.
    
  2) Jenkins
  
    * dashboard - manager jenkins - configure system - gitlab 설정
    * Connection name, gitlab host URL 작성
    * credentials - add - jenkins
      * kind: gitlab api token
      * scope: global
      * api token: gitlab 액세스 토큰 기입
      * ID: gitlab id
      * 생성후 credentials - gitlab api token 설정
    * Test Connection해서 success 나오면 저장(save)한다.
![image](https://user-images.githubusercontent.com/96723249/210208590-d10964c7-94f9-482e-8698-54c19cad9ad8.png)


<hr>
<hr>
<hr>
<hr>
<hr>
<hr>
<hr>
<hr>
<hr>



## 참고문헌
  * https://crispyblog.kr/development/common/10   

Docker 설치 및 Jenkins와 gitlab 연결, 설정
  * https://sinawi.tistory.com/370

연결된 GitLab - Jenkins 자동배포
  * https://velog.io/@dgh03207/Jenkins-Docker-gitLab-Webhook%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC
