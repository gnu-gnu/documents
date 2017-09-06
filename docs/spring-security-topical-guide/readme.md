출처 : https://spring.io/guides/topicals/spring-security-architecture/

Spring Security Architecture
============================
이 가이드는 프레임워크의 디자인과 기본적인 구성요소들에 대한 통찰력을 제공하는 입문서입니다. 우리는 애플리케이션 보안의 가장 기본적인 부분만을 다룰 것이지만, 이를 통해 스프링 시큐리티를 사용하는 개발자들이 경험하는 혼란스러운 점들 중의 일부를 명확히 할 수 있습니다. 이를 위해 우리는 필터 및 일반적인 메소드 어노테이션들을 이용하는 웹 어플리케이션에 적용되는 보안에 대해 좀 더 자세히 알아볼 것입니다. 안전한 응용 프로그램이 어떻게 동작하는지, 그리고 그것이 어떻게 커스터마이징되는지 혹은 애플리케이션 보안에 대해 어떻게 생각하는지 알 필요가 있을 때 이 가이드를 활용하십시오

이 가이드는 매뉴얼 혹은 대부분의 기본적인 문제 이상을 해결하기 위한 해법을 의도하지 않습니다. (그런 것들을 위하여 다른 소스들을 참고하십시오), 그러나 이 가이드는 초심자와 전문가 모두에게 유용할 것입니다. 스프링 부트 또한 자주 언급될 것입니다. 왜냐하면 그것은 애플리케이션을 보호하기 위한 약간의 기본적인 동작을 제공하며, 어떻게 그것이 전체적인 아키텍처에 부합하는지를 이해하는데 유용하기 때문입니다. 모든 원칙은 스프링 부트를 사용하지 않는 애플리케이션에도 동일하게 적용됩니다.

인증 및 접근제어
----------------
애플리케이션 보안은 두가지 이상 혹은 그 이하의 독립적인 문제로 귀결됩니다. 인증(당신이 누구인가?)과 권한부여(당신이 할 수 있는 일은 무엇인가?)입니다. 때때로 사람들은 "권한부여"라는 말 대신 혼란을 일으킬 수 있는 "접근제어"라는 말을 하곤 합니다. 그러나 "권한 부여"라는 말은 다른 곳에서도 매우 많이 사용되기 때문에 이것에 대해 생각 해보는 것은 유용할 수 있습니다. 스프링 시큐리티는 권한 부여로 부터 인증을 분리하도록 설계된 아키텍처를 가지고 있으며, 둘 모두를 위한 전략 및 확장 지점을 가지고 있습니다.

인증
----
인증을 위한 주요 전략 인터페이스는 단지 하나의 메소드를 가진 `AuthenticationManager` 이다.
```java
public interface AuthenticationManager{
  Authentication authenticate(Authentication Authentication) throws AuthenticationException
}
```
`AuthenticationManager` 는 `authenticate()` 메소드로 3가지 중 하나를 할 수 있습니다.
1. 입력이 유효한 주체를 나타내는 것을 검증가능한 경우 `Authentication`(일반적으로 `authenticated=true`와 함께)을 반환합니다
2. 유효하지 않은 주체를 입력이 나타낼 경우 `AuthenticationException`을 throw 합니다.
3. 결정할 수 없는 경우 `null`을 반환합니다.

`AuthenticationException`은 런타임 예외입니다. 애플리케이션은 이를 형식이나 애플리케이션의 목적에 따라 포괄적인 방법으로 다룹니다. 즉, 사용자의 코드는 일반적으로 그것을 잡아 내거나 처리할 것이라고 기대하지 않습니다. 예를 들어 인증 실패를 나타내는 페이지를 web UI가 렌더링하고, 백엔드 HTTP서비스는 `WWW-Authenticate `를 컨텍스트에 따라 포함하거나 포함하지 않는 401 응답을 보낼 것입니다.

가장 흔하게 사용하는 `AuthenticationManager`의 구현체는 `AuthenticationProvider` 인스턴스의 사슬에 참여하는 `ProviderManager`입니다. `AuthenticationProvider`는 `AuthenticationManager`와 유사하지만, 주어진 `Authentication` 형식을 지원하는지 질의하는 것을 호출자에게 허용하는 추가적인 메소드를 가지고 있습니다.

```java
public interface AuthenticationProvider{
  Authentication authenticate(Authentication authentication) throws AuthenticationException;
  boolean supports(Class<?> authentication);
}
```
`supports()` 메소드의 `Class<?>`인자는 실제로는 `Class<? extends Authentication>`입니다. (`authentication()`에 전달되는 무언가를 지원하는지만 묻습니다. `ProviderManager`는 `AuthenticationProviders`에 위임하는 방법으로 동일한 애플리케이션에서 여러가지 다른 인증 메카니즘을 지원할 수 있습니다.  만약 `ProviderManager`가 특정한 `Authentication` 인스턴스를 식별하지 못한다면 이것은 생략됩니다.

`ProviderManager`는 모든 공급자(providers)가 null을 반환한다면 참고할 수 있는 선택적인 부모를 가지고 잇습니다. 만약 부모가 사용 가능하지 않다면 `null Authentication`은 `AuthenticationException`을 야기합니다.

때때로 애플리케이션은 보호된 자원들의 논리적인 집합(예를 들어, `/api` 하위 경로 패턴에 부합하는 모든 웹 리소스)을 가집니다. 그리고 각 그룹은 그들만의 전용 `AuthenticationManager`를 가질 수 있습니다. 종종 그것들은 `ProviderManager`이기도 하며, 그들은 부모를 공유합니다. 부모는 모든 공급자들의 대체자 역할을 수행하는 일종의 "전역" 자원입니다.

<img src="https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/authentication.png" width="500">

###### 자료1. `ProviderManager`를 사용한 `AuthenticationManager` 계층 구조

인증 관리자 맞춤 수정
---------------------
스프링 시큐리티는 당신의 애플리케이션에 설정된 공통된 인증 관리자 기능을 빠르게 얻어오기 위한 몇가지 설정 도우미를 제공합니다. 가장 자주 공통적으로 사용되는 도우미는 메모리 내장, JDBC 혹은 LDAP 사용자 정보, 또는 맞춤형 `UserDetailsService`를 설정하는데 매우 좋은 `AuthenticationManagerBuilder` 입니다. 전역(부모) `AuthenticationManager`를 구성하는 어플리케이션의 예제가 아래에 있습니다.
>번역자 주) 원본 문서에서 아래의 몇몇 소스는 메소드 시그니처가 완전하지 않습니다. 추후 수정 예정입니다.

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
(설정 메소드에서 `@Override` 사용) `AuthenticationManagerBuilder`는 단지 전역의 자식인 "로컬" `AuthenticationManagerBuilder`를 생성하는데 사용되었습니다. 스프링 부트 애플리케이션에서 당신은 전역을 다른 Bean에 `@Autowired` 할 수 있습니다, 그러나 당신은 명시적으로 그것을 노출하지 않는 이상 로컬을 사용할 수는 없습니다.

스프링 부트는 당신이 만든 Bean에 의해 제공되는 `AuthenticationManager`가 선점하지 않는다면, 기본적인 전역 (단지 하나의 사용자를 위한)`AuthenticationManager`를 제공합니다. 당신이 적극적으로 맞춤형 global `AuthenticationManager`를 필요로 하지 않는다면 기본값도 당신이 그다지 걱정을 하지 않아도 될 만큼 충분합니다. 만약 `AuthenticationManager`를 생성하는 어떠한 설정을 한다면, 당신이 보호하는 자원에 대해 부분적으로 이를 수행할 수 있으며, 전역 기본값에 대해서는 걱정하지 마십시오

권한 부여와 접근제어
---------------
인증이 성공했다면, 우리는 권한 부여로 눈을 돌릴 수 있습니다, 그리고 여기의 핵심 전략은 `AccessDecisionManager`입니다. 프레임워크에 의해 제공되는 구현체들은 세가지가 있습니다 세가지 모두 `AccessDecisionVoter`에 행동을 위임합니다. `ProviderManager`가 `AuthenticationProviders`에 위임하는 것과 닮아 있습니다.

`AccessDecisionVoter`는 `ConfigAttributes`로 장식된 보안 `Object`와 (본인임을 나타내는)`Authentication`을 고려합니다.
```java
boolean supports(ConfigAttribute attribute);
boolean supports(Class<?> clazz);
int vote(Authentication authentication, S object,
        Collection<ConfigAttribute> attributes);
```
`Object`는 `AccessDecisionManager`와 `AccessDecisionVoter`의 매개변수 목록에서 전적으로 제네릭입니다. - 이것은 유저가 접근하기를 원하는 무언가(웹 리소스 혹은 자바 클래스에 존재하는 메소드가 두가지 가장 공통된 경우입니다.)를 대표합니다. `ConfigAttributes` 또한 접근에 필요한 권한의 수준을 결정하는 일부 메타데이터 보안가 포함된 `Object`의 장식을 나타내는 제네릭입니다. `ConfigAttribute`는 인터페이스입니다. 그러나 매우 일반적이며 `String`을 반환하는 단 하나의 메소드만을 가지고 있습니다. 그러므로 이 문자열은 어떠한 방법을 통해 리소스 소유자의 의도를 표현하고 누가 그것에 접근할 수 있는지 규칙을 드러냅니다. 전형적인 `ConfigAttribute`는 사용자 역할(예를 들어 `ROLE_ADMIN`이나 `ROLE_AUDIT`)의 이름입니다, 그리고 그것들은 종종 특별한 형식(예를 들어 접두사 `ROLE_`)이나 연산될 필요가 있는 표현식을 가집니다.

대부분의 사람들은 그저 기본적인 `긍정적 동작 기반`(거부가 없다면 접근을 허용하는) `AccessDecisionManager`를 사용합니다. 맞춤형 수정은 투표자(voters)에서 새로운 투표자를 추가하거나, 기존의 투표자가 동작하는 방식을 수정하는 방향으로 일어나는 경향이 있습니다.

`ConfigAttributes`를 Spring Expression Language(SpEL) 표현식으로 사용하는 것은 매우 자주 일어나는 일입니다. 예를 들자면 `isFullyAuthenticated() && hasRole('FOO')`입니다. 이것은 표현식을 다룰 수 있으며, 그것들을 위한 컨텍스트를 생성할 수 있는 `AccessDecisionVoter`가 이것을 지원합니다. 다룰 수 있는 표현식의 범위를 확장하기 위하여 `SecurityExpressionRoot` 및 가끔은 `SecurityExpressionHandler`의 맞춤형 구현이 필요합니다.

웹 보안
-------
<img src="https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/filters.png" height="350px">

클라이언트는 애플리케이션에 요청을 전송하고, 컨테이너는 어떤 필터와 서블릿을 적용할지 request URI의 경로에 기반하여 결정합니다. 한 개의 서블릿은 하나의 요청을 다룰 수 있지만, 필터는 사슬을 형성합니다, 그래서 필터들은 순서를 가집니다, 그리고 필터는 스스로 요청을 다루기를 원할 경우 다른 사슬들을 거부할 수 있습니다. 필터는 또한 요청 혹은 내려보내는 필터 혹은 서블릿에서 사용되는 응답을 조작할 수 있습니다. 필터 사슬의 순서는 매우 중요합니다, 그리고 스프링 부트는 이것을 2가지의 메카니즘으로 관리합니다: 하나는 `Filter` 타입의 `@Beans`가 `@Order`을 갖거나, `Ordered` 인터페이스를 구현할 수 있습니다. 또 다른 하나는 그것들이 API를 통해 순서를 스스로 가지는 `FilterRegistrationBean`의 일부가 되는 것입니다. 몇몇 이미 제공되는 필터들은 서로 간에 순서를 알려주는 자신만의 상수를 정의합니다. (예를 들어, 스프링 세션의 `SessionRepositoryFilter`의 `DEFAULT_ORDER`는 `Integet.MIN_VALUE + 50`이며 이것은 필터 사슬에서 앞부분에 있다는 것을 의미하지만, 앞에 위치한 다른 필터들을 막지는 않습니다.)

스프링 시큐리티는 사슬 내에서 단일 `Filter`로 위치합니다, 그리고 이것의 구체적인 타입은 `FilterChainProxy`입니다, 그 이유는 곧 명백히 밝혀질 것입니다. 스프링 부트 애플리케이션에서 시큐리티 필터는 `ApplicationContext`상의 `@Bean`입니다, 그리고 이것은 모든 요청에 적용하기 위해 기본적으로 위치합니다. 이것은 `FilterRegistrationBean.REQUEST_WRAPPER_FILTER_MAX_ORDER`(스프링부트 애플리케이션이 요청을 감싸고, 행동을 조작한다면 기대할 수 있는 순서의 최대값)에 의해 차례로 고정된 `SecurityProperties.DEFAULT_FILTER_ORDER`에 의해 정의된 곳에 있습니다. 더 생각 해 볼 것은 다음과 같습니다: 컨테이너의 관점에서 스프링 시큐리티는 단일 필터입니다. 그러나 내부에는 각각이 특별한 역할을 수행하는 더 많은 필터들이 존재합니다. 아래의 그림을 보시죠.

<img src="https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/security-filters.png" height="350px">
###### 자료2. 스프링 시큐리티는 하나의 물리적인 `Filter`이지만, 내부 필터들의 사슬에 처리를 위임합니다.
사실 시큐리티 필터에는 간접적인 계층이 하나 이상 있습니다: 그것은 컨테이너상에 스프링 `@Bean`이 되어서는 안되는 `DelegatingFilterProxy`로 존재합니다. 프록시는 보통 `springSecurityFilterChain`이라는 고정된 이름으로 명명된 언제나 `@Bean`인 `FilterChainProxy`에 위임합니다. `FilterChainProxy`는 필터들의 사슬(혹은 사슬들)로서 내부적으로 정렬된 모든 보안 로직을 포함하고 있습니다. 모든 보안 로직은 동일한 API를 가지고 있습니다. (모두 서블릿 스펙에 정의된 `Filter` 인터페이스를 구현하고 있습니다) 그리고 그것들은 모두 다 사슬의 나머지에 대해 거부 할 기회를 가지고 있습니다.

동일한 최상위 레벨의 `FilterChainProxy` 안의 스프링 시큐리티에 의해 전적으로 관리되는 복수의 필터 사슬들이 있을 수 있으며, 컨테이너들은 그것에 대해 인식하지 않습니다. 스프링 시큐리티 필터는 필터 사슬들의 목록을 포함합니다 그리고 일치하는 첫번째 사슬로 요청을 보냅니다. 아래의 그림은 요청 경로(/\*\*에 매칭되기 전 /foo/\*\*에 매칭)매칭에 기반한 처리를 보여주고 있습니다.

<img src="https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/security-filters-dispatch.png" height="300px">

###### 자료3. 스프링 시큐리티의 `FilterChainProxy`는 일치하는 첫번째 사슬로 요청을 보냅니다.

맞춤형 보안 설정이 없는 순수한 스프링 부트 애플리케이션은 몇 가지(n이라고 하겠습니다) 필터 사슬들을 가집니다, 보통 n=6 입니다. 첫번째(n-1) 사슬은 /css/\*\* 나 /images/\*\* 그리고 오류 화면 `/error`(`SecurityProperties` 설정 Bean에서 조절할 수 있는 경로)을 무시하기 위함입니다.

마지막 사슬은 `/\*\*`로 잡히는 모든 경로와 대응하며 좀 더 활동적입니다, 인증, 권한부여, 예외처리, 세션처리, 헤더 쓰기 등의 로직을 담고 있습니다. 이 사슬에는 총 11개의 필터들이 존재합니다, 그러나 어떤 필터를 언제 사용할 것인가를 사용자 스스로 고민하는 것은 일반적으로는 필요하지 않습니다.

> 알아두기
스프링 시큐리티 내부의 모든 필터들이 컨테이너에게는 알려져 있지 않다는 사실은 특히 스프링 부트 애플리케이션에서는 중요합니다. `Filter` 타입의 모든 `@Beans`들은 기본적으로 컨테이너에 자동 등록됩니다. 그러므로 만약 보안 사슬에 맞춤형 필터를 추가하려면 그 필터는 `@Bean`으로 만들지 말아야하고, 명시적으로 컨테이너 등록을 불가능하게 하는 `FilterRegistrationBean`으로 이것을 포장하여야 합니다.

필터 사슬 만들기 및 맞춤형 수정
-------------------------------
(`/\*\*` 경로 요청에만 대응하는) 스프링 부트 애플리케이션이 대비책으로 마련한 기본 필터 사슬은 `SecurityProperties.BASIC_AUTH_ORDER`에서 미리 정의된 순서를 가지고 있습니다. 당신은 `serurity.basic.enable=false`를 설정하여 이것을 완전히 꺼버릴 수 있습니다, 혹은 이것을 대비책으로 사용하면서 다른 룰들을 더 낮은 순서로 지정할 수 있습니다. 이를 위해 `WebSecurityConfigurerAdatap`(혹은 `WebSecurityConfigurer`) 타입의 `@Bean`을 추가하여 봅시다. 그리고 이 클래스를 `@Order`로 꾸며 봅시다. 예를 들어:
```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
     ...;
  }
}
```
이 Bean은 스프링 시큐리티에 새로운 필터 사슬을 등록하도록 할 것입니다, 그리고 대비책으로 마련한 필터보다 순서를 앞에 둘 것입니다.

많은 애플리케이션들은 하나의 자원들의 집합에 대해 다른 집합들과 비교되는 완전히 다른 접근 규칙을 가지고 있습니다. 예를 들자면 UI와 뒷단 API를 제공하는 애플리케이션은 아마도 UI부분으로 리다이렉트하는 쿠키 기반의 인증 방식을 지원할 것입니다, 그리고 API부분에서는 인증되지 않은 요청에 대하여 401을 응답하는 토큰 기반 인증을 지원할 것입니다. 각 자원의 집합은 유일한 순서와 요청 대응을 위한 메소드(request matcher)를 가지는  각 자원 집합의 `WebSecurityConfigurerAdapter`를 가집니다. 만약 매칭 규칙들이 경합한다면, 순위가 높은 필터가 우선입니다.

요청 처리 및 권한 부여를 위한 요청 매칭
---------------------------------------
보안 필터 사슬(혹은 동의어로서 `WebSecurityConfigurerAdapter`)은 HTTP request에 이를 적용할지 여부를 결정하는데 사용하는 request matcher를 가지고 있습니다. 특정 필터 사슬을 적용하기로 결정하였다면, 다른 것들은 적용되지 않습니다. 그러나 필터 사슬에서는 `HttpSecurity` 설정 메소드 안에서 별도의 matcher를 추가하여 더욱 뚜렷한 권한 부여에 대한 제어가 가능합니다. 예를 들어:

```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
      .authorizeRequests()
        .antMatchers("/foo/bar").hasRole("BAR")
        .antMatchers("/foo/spam").hasRole("SPAM")
        .anyRequest().isAuthenticated();
  }
}
```
스프링 시큐리티를 설정하면서 가장 범하기 쉬운 실수 중 하나는 이 matcher들을 다른 프로세스들에 적용하는 것을 잊어버리는 것입니다, 하나는 모든 필터 사슬을 위한 request matcher이며, 다른 것은 적용할 접근 규칙을 선택하기 위한 것입니다.

애플리케이션 보안 규칙과 액추에이터 규칙의 조합
------------------------------------------
만약 스프링 부트 액추에이터를 엔드포인트 관리목적으로 사용한다면, 아마도 당신은 그것이 안전하기를 원할 것입니다, 그리고 기본적으로 그것들은 안전합니다. 사실, 스프링 시큐리티를 적용한 애플리케이션에 액추에이터를 추가하자마 당신은 액추에이터 엔드포인트들을 위한 별도의 필터 사슬을 갖게 됩니다. 이것들은 액추에이터 엔드포인트에만 적용하도록 정의되어 있으며 기본적인 `SecurityProperties` 대비용 필터보다 순서가 5 빠른 `ManagementServerProperties.BASIC_AUTH_ORDER`를 가지고 있습니다. 그러므로 대비책으로 마련된 필터보다 먼저 찾게 됩니다.

만약 액추에이터 엔드포인트에 애플리케이션 보안 규칙을 적용하고자 한다면, 액추에이터의 필터 사슬보다 더 빠른 순서의 필터 사슬을 모든 액추에이터 엔드포인트들을 포함하는 request matcher와 함께 추가할 수 있습니다. 만약 엑추에이터 엔드포인트들을 위한 기본적인 보안 설정을 선호한다면, 가장 쉬운 방법은 당신만의 필터를 엑추에이터의 것보다는 나중에 두면서, 대비책으로 마련된 필터보다는 먼저 두는 것입니다 (예를 들어 `ManagementServerProperties.BASIC_AUTH_ORDER+1`) 예제를 보시겠습니다.

```java
@Configuration
@Order(ManagementServerProperties.BASIC_AUTH_ORDER + 1)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
     ...;
  }
}
```

> 알아두기
웹 계층의 스프링 시큐리티는 현재 서블릿 API와 묶여 있습니다. 그러므로 내장형 혹은 다른 방법으로 작동하는 서블릿 컨테이너 안에서 구동되는 애플리케이션에 대해서만 실제로 적용 가능합니다. 그러나 이것은 Spring MVC나 다른 Spring web stack들과는 묶여 있지 않습니다, 그러므로 어떠한 서블릿 애플리케이션에서든지 사용할 수 있습니다, 예를 들자면 JAX-RS 같은 것들이 사용하고 있습니다.

메소드 보안
-----------
웹 애플리케이션을 보호하는 것 뿐만 아니라, 스프링 시큐리티는 자바 메소드 실행에도 접근 권한을 적용하는 것을 지원합니다. 스프링 시큐리티에게 있어 이것은 단지 다른 형태의 `보호된 자원`일 뿐입니다. 사용자에게 이것이 의미하는 바는 코드의 다른 위치에서 동일한 형식의 `ConfigAttribute` 문자열(예를 들자면, roles 혹은 표현식)을 사용하여 접근 규칙을 선언한다는 것을 의미합니다. 첫번째 단계로는 우리 애플리케이션의 최상위 레벨 설정에서 메소드 보안을 활성화 시켜보는 것입니다.

```java
@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SampleSecureApplication {
}
```

그리고 나서 우리는 메소드 자원들을 바로 장식할 수 있습니다, 예를 들어

```java
@Service
public class MyService {

  @Secured("ROLE_USER")
  public String secure() {
    return "Hello Security";
  }

}
```

이 샘플은 안전한 메소드가 존재하는 서비스입니다. 만약 스프링이 이러한 타입의 `@Bean`을 생성한다면 이것은 프록시를 거치게 됩니다. 그리고 호출자는 메소드가 실제로 실행되기 전 보안 인터셉터를 거쳐야만 합니다. 만약 엑세스가 거부되었다면, 호출자는 실제 메소드 결과 대신 `AccessDeniedException`을 받게 됩니다.

보안 제약사항을 강제하기 위해 메소드에 사용되는 다른 어노테이션들이 있습니다, 메소드 인자 및 결과값에 대하여 참조를 포함한 표현식을 작성할 수 있도록 하는 `@PreAuthorize`와 `@PostAuthorize`가 특히 그러합니다.

> 팁
웹 보안과 메소드 보안을 혼용하는 것은 이상한 일이 아닙니다. 예를 들어 필터 사슬은 인증과 로그인 페이지 전환등의 유저 경험을 제공하며, 메소드 보안은 좀 더 세분화 된 단계의 보호를 제공합니다.

스레드 다루기
-------------
스프링 시큐리티는 근본적으로 스레드에 영향을 받습니다. 현재 인증된 본인을 다양한 다운스트림 소비자들이 사용 가능하도록 해야하기 때문입니다. 기본적인 구성 블록은 `Authentication`을 포함하는 `SecurityContext`입니다. (그리고 사용자가 로그인하였을 때 그것은 명시적으로 `authenticated`된 `Authentication`이 될 것입니다.) 간단하게 `ThreadLocal`을 변경하는 `SecurityContextHolder`의 정적 메소드를 통해 언제나 `SecurityContext`를 접근하거나 변경할 수 있습니다. 예제를 보시죠

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
assert(authentication.isAuthenticated);
```

사용자의 애플리케이션 코드가 이런 것은 **흔한 일은 아닙니다.** 그러나 이것은 예를 들자면, 맞춤형 인증 필터를 작성해야 하는 경우 (`SecurityContextHolder`의 사용을 회피할만한 기본 클래스들이 스프링 시큐리티에 있더라도) 유용합니다.

웹 엔드포인트에서 현재 인증된 사용자에 접근할 필요가 있는 경우 당신은 @RequestMapping 에서 인자 파라미터를 사용할 수 있습니다. 예를 들자면
```java
@RequestMapping("/foo")
public String foo(@AuthenticationPrincipal User user) {
  ... // 사용자에 관계된 작업을 작성
}
```

이 어노테이션은 현재 `Authentication`을 `SecurityContext` 외부로 끄집어내서 메소드 인자로 넘겨주기 위해 `getPrincipal()` 메소드를 호출한다. `Authentication`의 `Principal` 타입은 인증을 검증하는데 사용되는 `AuthenticationManager`에 의존한다. 그러므로 당신의 사용자 정보에 대해 타입에 안전한 참조를 얻는 유용한 수법이 될 수 있다.

스프링 시큐리티를 사용할 경우 `HttpServetRequest`의 `Principal`은 `Authentication` 타입이 될 것이므로 이것을 직접 사용할 수 있다.

```java
@RequestMapping("/foo")
public String foo(Principal principal) {
  Authentication authentication = (Authentication) principal;
  User = (User) authentication.getPrincipal();
  ... // 사용자에 관계된 작업을 작성
}
```

이것은 스프링 시큐리티를 사용하지 않ㅇ르 때 동작하는 코드를 작성할 필요가 있을 때 간혹 유용합니다. (`Authentication` 클래스를 불러올 때 좀 더 주의를 기울이십시오)

안전한 메소드를 비동기적으로 다루기
----------------------------------
`SecurityContext`는 스레드에 영향을 받기 때문에, 안전한 메소드를 호출하는 백그라운드 작업(예를 들어 `@Async`)을 하기를 원할 때는 컨텍스트가 전파되었는지 확인할 필요가 있다. 이것은 백그라운드에서 실행되는 작업(`Runnable`, `Callable` 등)으로 `SecurityContext`를 감싸는 것으로 요약된다. 스프링 시큐리티는 이것을 쉽게하기 위해 `Runnable`과 `Callable`을 감싸는 것과 같은 도우미들을 제공한다. `@Async` 메소드에 `SecurityContext`를 전파하기 위해, `AsyncConfigurer`를  제공하고, `Executor`가 정확한 타입인지 확인을 필요가 있다

```java
@Configuration
public class ApplicationConfiguration extends AsyncConfigurerSupport {

  @Override
  public Executor getAsyncExecutor() {
    return new DelegatingSecurityContextExecutorService(Executors.newFixedThreadPool(5));
  }

}
```


출처 : https://spring.io/guides/topicals/spring-security-architecture/
