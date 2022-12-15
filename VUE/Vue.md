# Flow
  1) .vue 파일
    * Computed
      : 연산프로퍼티
        템플릿 안에서 기술해도 되지만, 길어지면 복잡해져 유지보수가 어려워지니 Computed를 사용한다.
        methods가 computed의 역할을 대체할 수 있고, 결과도 똑같다
        Computed를 사용하는 이유는 methods 처럼 함수가 다시 실행되는게 아닌, 이전의 결과를 즉시 되돌려준다. (의존관계에 의해 캐쉬화)
      : mapUtils.mapGetter
        store의 Getter를 가져온다. computed 안에 작성.
        namespace를 사용하기 때문에 키 이름을 적어주면 됨.
    * methods
      : rendering 할 때 마다 항상 함수를 다시 실행시킨다.
        결과는 Computed와 똑같다.
      : Maputils.mapActions
    * async await 사용
      : 비동기처리 패턴
        
  
  2) Store
    * mutation
      : state 값을 변경하는 로직 (setterA)
        인자를 받아 vuex에 넘겨줄 수 있고computed가 아닌 methods에 등록
        actions와는 다르게 동기적 로직 정의임
        ![image](https://user-images.githubusercontent.com/96723249/207990806-ce9244d3-b6b3-4aa2-86b1-4418a07e311b.png)

    * actions request.~~
      : actions에서 비동기 처리를 하는 메소드를 만든다.
        순차적인 로직들만 선언하는 mutaions와는 다르게 비 순차적 또는 비동기 처리(axios 등) 로직들을 선언
    * resource namespace
  
  3) request.js
    * axios
      url : path  > api 경로
      method : post, get , delete , put
    * response를 api 통해서 받아옴

  4) store
    * request.js에서 받아온 response를
      actions request.~~ () response에 저장
      
  5) .vue 에 호출

<hr>
<hr>
