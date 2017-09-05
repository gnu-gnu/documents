Spring Security Architecture
============================
이 가이드는 프레임워크의 디자인과 기본적인 구성요소들에 대한 통찰력을 제공하는 입문서입니다. 우리는 애플리케이션 보안의 가장 기본적인 부분만을 다룰 것이지만, 이를 통해 스프링 시큐리티를 사용하는 개발자들이 경험하는 혼란스러운 점들 중의 일부를 명확히 할 수 있습니다. 이를 위해 우리는 필터 및 일반적인 메소드 어노테이션들을 이용하는 웹 어플리케이션에 적용되는 보안에 대해 좀 더 자세히 알아볼 것입니다. 안전한 응용 프로그램이 어떻게 동작하는지, 그리고 그것이 어떻게 커스터마이징되는지 혹은 애플리케이션 보안에 대해 어떻게 생각하는지 알 필요가 있을 때 이 가이드를 활용하십시오

이 가이드는 매뉴얼 혹은 대부분의 기본적인 문제 이상을 해결하기 위한 해법을 의도하지 않습니다. (그런 것들을 위하여 다른 소스들을 참고하십시오), 그러나 이 가이드는 초심자와 전문가 모두에게 유용할 것입니다. 스프링 부트 또한 자주 언급될 것입니다. 왜냐하면 그것은 애플리케이션을 보호하기 위한 약간의 기본적인 동작을 제공하며, 어떻게 그것이 전체적인 아키텍처에 부합하는지를 이해하는데 유용하기 때문입니다. 모든 원칙은 스프링 부트를 사용하지 않는 애플리케이션에도 동일하게 적용됩니다.

인증 및 접근제어
----------------
애플리케이션 보안은 두가지 이상 혹은 그 이하의 독립적인 문제로 귀결됩니다. 인증(당신이 누구인가?)과 허가(당신이 할 수 있는 일은 무엇인가?)입니다. 때때로 사람들은 "허가"라는 말 대신 혼란을 일으킬 수 있는 "접근제어"라는 말을 하곤 합니다. 그러나 "허가"라는 말은 다른 곳에서도 매우 많이 사용되기 때문에 이것에 대해 생각 해보는 것은 유용할 수 있습니다. 스프링 시큐리티는 허가로 부터 인증을 분리하도록 설계된 아키텍처를 가지고 있으며, 둘 모두를 위한 전략 및 확장 지점을 가지고 있습니다.

인증
----
인증을 위한 주요 전략 인터페이스는 단지 하나의 메소드를 가진 `AuthenticationManager` 이다.
```java
public interface AuthenticationManager{
  Authentication authenticate(Authentication Authentication) throws AuthenticationException
}
```
`AuthenticationManager` 는 `authenticate()` 메소드로 3가지 중 하나를 할 수 있다.
1. 입력이 유효한 주체를 나타내는 것을 검증가능한 경우 `Authentication`(일반적으로 `authenticated=true`와 함께)을 반환한다.
2. 유효하지 않은 주체를 입력이 나타낼 경우 `AuthenticationException`을 throw한다.
3. 결정할 수 없는 경우 `null`을 반환한다

`AuthenticationException`은 런타임 예외입니다. 애플리케이션은 이를 형식이나 애플리케이션의 목적에 따라 포괄적인 방법으로 다룹니다. 즉, 사용자의 코드는 일반적으로 그것을 잡아 내거나 처리할 것이라고 기대하지 않습니다. 예를 들어 인증 실패를 나타내는 페이지를 web UI가 렌더링하고, 백엔드 HTTP서비스는 `WWW-Authenticate `를 컨텍스트에 따라 포함하거나 포함하지 않는 401 응답을 보낼 것입니다.

가장 흔하게 사용하는 `AuthenticationManager`의 구현체는 `AuthenticationProvider` 인스턴스의 사슬에 참여하는 `ProviderManager`입니다. `AuthenticationProvider`는 `AuthenticationManager`와 유사하지만, 주어진 `Authentication` 형식을 지원하는지 질의하는 것을 호출자에게 허용하는 추가적인 메소드를 가지고 있습니다.

```java
public interface AuthenticationProvider{
  Authentication authenticate(Authentication authentication) throws AuthenticationException;
  boolean supports(Class<?> authentication);
}
```
`supperts()` 메소드의 `Class<?>`인자는 실제로는 `Class<? extends Authentication>`입니다. (`authentication()`에 전달되는 무언가를 지원하는지만 묻습니다. `ProviderManager`는 `AuthenticationProviders`에 위임하는 방법으로 동일한 애플리케이션에서 여러가지 다른 인증 메카니즘을 지원할 수 있습니다.  만약 `ProviderManager`가 특정한 `Authentication` 인스턴스를 식별하지 못한다면 이것은 생략됩니다.

`ProviderManager`는 모든 공급자(providers)가 null을 반환한다면 참고할 수 있는 선택적인 부모를 가지고 잇습니다. 만약 부모가 사용 가능하지 않다면 `null Authentication`은 `AuthenticationException`을 야기합니다.

때때로 애플리케이션은 보호된 자원들의 논리적인 집합(예를 들어, `/api` 하위 경로 패턴에 부합하는 모든 웹 리소스)을 가집니다. 그리고 각 그룹은 그들만의 전용 `AuthenticationManager`를 가질 수 있습니다. 종종 그것들은 `ProviderManager`이기도 하며, 그들은 부모를 공유합니다. 부모는 모든 공급자들의 대체자 역할을 수행하는 일종의 "전역" 자원입니다.

<img src="https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/authentication.png" width="500">
###### 자료1. `ProviderManager`를 사용한 `AuthenticationManager` 계층 구조

인증 관리자 맞춤 수정
---------------------
스프링 시큐리티는 당신의 애플리케이션에 설정된 공통된 인증 관리자 기능을 빠르게 얻어오기 위한 몇가지 설정 도우미를 제공합니다. 가장 자주 공통적으로 사용되는 도우미는 메모리 내장, JDBC 혹은 LDAP 사용자 정보, 또는 맞춤형 `UserDetailsService`를 설정하는데 매우 좋은 `AuthenticationManagerBuilder` 입니다. 전역(부모) `AuthenticationManager`를 구성하는 어플리케이션의 예제가 아래에 있습니다.

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter{
  ... // web stuff here

 @Autowired
 public initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
   builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
     .password("secret").roles("USER");
 }
}
```
이 예제는 웹 애플리케이션에 관한 것이지만, `AuthenticationManagerBuilder`는 좀 더 광범위하게 활용될 수 있습니다 (웹 애플리케이션 보안이 어떻게 구현되는지 좀 더 자세한 내용은 아래를 참고하세요) `AuthenticationManagerBuilder`는 `@Bean`의 메소드로 `@Autowired` 됩니다. 그리하여 전역(부모) `AuthenticationManager`가 됩니다. 반면 아래와 같은 방법으로 했다면

```java

@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

  @Autowired
  DataSource dataSource;

   ... // web stuff here

  @Override
  public configure(AuthenticationManagerBuilder builder) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```
(설정 메소드에서 `@Override` 사용) `AuthenticationManagerBuilder`는 단지 전역의 자식인 "로컬" `AuthenticationManagerBuilder`를 생성하는데 사용되었습니다.

출처 : https://spring.io/guides/topicals/spring-security-architecture/
