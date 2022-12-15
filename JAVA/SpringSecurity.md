## Spring Security 란?
  * Spring 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 스프링 하위 프레임워크
  * '인증'과 '권한'에 대한 부분을 Filter 흐름에 따라 처리
  * Filter는 Dispatcher Servlet으로 가기 전에 적용되므로 가장 먼저 URL 요청을 받지만, 
  * Interceptor는 Dispatcher와 Controller사이에 위치한다는 점에서 적용 시기의 차이가 있다
  * Spring Security는 보안과 관련해서 체계적으로 많은 옵션을 제공해주기 때문에 개발자 입장에서는 일일이 보안관련 로직을 작성하지 않아도 된다는 장점

## 인증관련 Architecture
![image](https://user-images.githubusercontent.com/96723249/207767455-305b2fef-9bdb-4a0f-8dbe-afc654496cb9.png)

1. 사용자가 Form을 통해 로그인 정보를 입력하고 인증 요청을 보낸다.
2. AuthenticationFilter(사용할 구현체 UsernamePasswordAuthenticationFilter)가 HttpServletRequest에서 사용자가 보낸 아이디와 패스워드를 인터셉트한다. 프론트 단에서 유효성검사를 할 수도 있지만, 안전을 위해서 다시 한번 사용자가 보낸 아이디와 패스워드의 유효성 검사를 한다. HttpServletRequest에서 꺼내온 사용자 아이디와 패스워드를 진짜 인증을 담당할 AuthenticationManager 인터페이스(구현체 - ProviderManager)에게 인증용 객체(UsernamePasswordAuthenticationToken)로 만들어줘서 위임한다.
3. AuthenticationFilter에게 인증용 객체(UsernamePasswordAuthenticationToken)을 전달받는다.
4. 실제 인증을 할 AuthenticationProvider에게 Authentication객체(UsernamePasswordAuthenticationToken)을 다시 전달한다.
5. DB에서 사용자 인증 정보를 가져올 UserDetailsService 객체에게 사용자 아이디를 넘겨주고 DB에서 인증에 사용할 사용자 정보(사용자 아이디, 암호화된 패스워드, 권한 등)를 UserDetails(인증용 객체와 도메인 객체를 분리하지 않기 위해서 실제 사용되는 도메인 객체에 UserDetails를 상속하기도 한다.)라는 객체로 전달 받는다.
6. AuthenticationProvider는 UserDetails 객체를 전달 받은 이후 실제 사용자의 입력정보와 UserDetails 객체를 가지고 인증을 시도한다.   
10. (10)인증이 완료되면 사용자 정보를 가진 Authentication 객체를 SecurityContextHolder에 담은 이후 AuthenticationSuccessHandle를 실행한다. (실패시 AuthenticationFailureHandler를 실행한다.)

## 인증(Authorizatoin)과 인가(Authentication)
  * 인증(Authentication): 해당 사용자가 본인이 맞는지를 확인하는 절차
  * 인가(Authorization): 인증된 사용자가 요청한 자원에 접근 가능한지를 결정하는 절차 
  * Spring Security는 기본적으로 인증 절차를 거친 후에 인가 절차를 진행하게 되며, 인가 과젱에서 해당 리소스에 대한 접근 권한이 있는지 확인
  * 인증과 인가를 위해 Principal을 아이디로, Credential을 비밀번호로 사용하는 Credential 기반의 인증 방식을 사용
  * Principal(접근 주체): 보호받는 Resource에 접근하는 대상
  * Credential(비밀번호): Resource에 접근하는 대상의 비밀번호

<hr>
<hr>

## Gradle 환경에서 Build.gradle에 Dependency Injection 
  * spring security 사용시에는 아래 종속성도 추가 해야 한다.
    * implementation 'org.springdoc:springdoc-openapi-security:1.6.13'

## Config 폴더에 Spring Security 관련 설정파일 및 Bean을 만든다.
  1) SecurityConfig.class
    
    * @Bean 등록
      * @Bean
        public JwtAuthFilter jwtFilter() {
        return new JwtAuthFilter(); }
        
      * @Bean
        public AuthFailEntryPoint failEntryPoint() {
        return new AuthFailEntryPoint(); }
        
      * @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          http.cors().configurationSource(corsConfigurationSource());
          http.authorizeRequests()
            .antMatchers( ....
                        ).peermitAll()
            .antMatchers().authenticated();
        * Spring Security 5.7.0-M2 부터 WebSecurityConfigurerAdapter는 컴포넌트 기반의 보안 설정을 권장한다는 이유로 Deprecated 처리되었다
          그래서 SecurityFilterChain 사용을 권장하며, 반환값이 있고 Bean 등록을 함으로써 컴포넌트 기반의 보안 설정이 가능해진다.
        * configurationSource : CORS 요청에 따라 어떤 방법으로 해결할 지 방법을 지정해준다.
        * authorizeRequests : path Matching 기반으로 인증, 인가가 필요한 부분을 설정한다.
        * antMatchers() : 적용할 경로를 설정하고 hasRole, permitAll (모두 접근 가능), authenticated 등을 통해 누가 접근할 수 있는지
                          페이지 권한 설정을 할 수 있다.
        
      * @Bean
        public CorsConfigurationSource corsConfigurationSource()
        : CORS 설정을 추가하여 CorsConfigurationSource 타입의 Bean에 허용할 Origin, Method, Header를 설정한다. 
 
  2) JwtAuthFilter.class
    * protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
      * HTTP 요청이 오면 WAS(tomcat)가 HttpServletRequest, HttpServletResponse 객체를 만들어 줍니다.
      * 만든 인자 값을 받아옵니다.
      * 요청이 들어오면 diFilterInternal 이 딱 한번 실행된다.
      
      * Header에서 Token을 가져와서 유효하다면 파싱해서 인증을 진행
      * 인증에 성공하면 SecurityContextHolder 에 Authentication 객체를 집어넣어 set 함으로써 인가한다 (가장 중요한 부분)
      * JWT가 정상이라면 이미 인증이 완료된 사용자이므로 Spring Security가 인식할 수 있게 해야한다.
      * 다른 요청은 Spring Security에 의해서 차단이되고, AuthFailEntryPoint로 던질거임.
      
    * JWT 로그인의 이해
      ![image](https://user-images.githubusercontent.com/96723249/207778733-6c8893da-4676-4155-b069-c725948e1125.png)


  3) AuthFailEntryPoint.class
    * Spring Security에 의해 차단 되었을 경우 오류메세지 정의 등.
    * 인증이 안된 사용자가 접근 시 차단되었을 경우 정의



## 참고문헌
  https://mangkyu.tistory.com/76
  
