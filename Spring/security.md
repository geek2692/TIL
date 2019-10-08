## Spring Security 정리

------

### 관련용어

- Principal(접근주체) : 보호된 대상에 접근하는 유저
- Authenticate(인증) : 현재 유저가 누구인지 확인, 작업을 수행할 수 있는 주체임을 증명
- Authorize(인가) : 현재 유저가 어떤 서비스, 페이지에 접근할 수 있는 권한이 있는지 검사
- Authorization(권한) : 인증된 주체가 애플리케이션의 동작을 수행할 수 있도록 허락되었는지 결정

위 내용은 블로그에서 많이들 언급하길래 가져옴..



### 사용하는 이유?

위 용어에서 말하는 것처럼 비인가 사용자에게 의도되지않은 정보노출이 되는걸 보호하고 사용자가 누군지 증명이 끝나고 나면 각자 권한을 부여하여 관리하기 위해서..?



### Session / Cookie 방식 구조

이전 인증방식에는 HTTP Header에 사용자의 정보를 넣어 인증하는 방식이 있었지만 매번 요청에 넣어서 보내기에는 보안에 매우 취약하므로 .. 
(하지만 이 방식도 HTTP 요청이 중간에 탈취됐다면(세션 하이재킹) 보안에 문제가 됨)

![img](https://t1.daumcdn.net/cfile/tistory/994BEA345B53368401)

순서는

1. 사용자가 로그인을 시도하면 (http request)
2. AuthenticationFIlter에서  사용자 DB까지 타고 들어가 사용자 확인
3. DB에 사용자 정보가 있다면 회원 정보(UserDetails) 세션 생성후..
4. Spring Security의 인메모리 세션저장소 SecurityContextHolder에 저장
5. 유저에게 Session ID와 함께 응답 (http response)
6. 이후 데이터 요청시 요청쿠키에서 JESSIONID 검증 후 유효하면 유저 정보(Authentication)을 쥐어줌

세션 쿠키방식의 인증은 기본적으로 세션 저장소 필요([보통 Redis 사용](http://wiki.sys4u.co.kr/pages/viewpage.action?pageId=8552454))



[참조한 블로그]

https://tansfil.tistory.com/58?category=255594

https://sjh836.tistory.com/165?category=680970