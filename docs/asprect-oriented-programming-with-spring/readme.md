#11. 스프링에서의 관점 지향 프로그래밍

## 11.1 들어가기
<i>관점 지향 프로그래밍(AOP)</i>은 프로그램 구조에 대해 다른 사고 방식을 제공하여 객체 지향 프로그래밍(OOP)을 보완하는 것입니다. OOP에서 모듈성의 핵심 단위는 클래스인 반면, AOP에서는 <i>관점(aspect)</i>입니다. 관점은 다양한 타입이나 객체를 관통하는 트랜잭션 관리와 같은 고려사항에 대한 모듈화를 가능케 합니다. (이러한 고려사항들 AOP에서는 종종 <i>crosscutting</i>이라는 용어로 표현 됩니다.)

스프링의 핵심 컴포넌트 중 하나는 AOP프레임워크입니다. 스프링 IoC 컨테이너는 AOP에 의존하지 않으므로 원하지 않는다면 AOP를 쓰지 않아도 좋습니다. AOP는 매우 뛰어난 미들웨어 솔루션을 제공하여 Spring IoC를 보완합니다.

> 스프링 2.0 AOP
(번역자 주 : aspectj, pointcut, weaving 등은 실제 코딩에서도 사용되므로 원어로 표현하였습니다.)
스프링 2.0은 간단하면서도 매우 강력한 [스키마 기반 접근](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-schema) 혹은 [@AspectJ 형식 어노테이션](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-ataspectj)을 사용하여 맞춤형 aspects를 작성하는 방법을 소개하였습니다. 두 스타일 모두 여전히 Spring AOP를 weaving에 사용하면서 완전한 타입의 advice이며, AspectJ pointcut 언어를 사용합니다.
Spring 2.0의 스키마- 혹은 @AspectJ 기반 AOP 지원을 이번 챕터에서 다룰 것입니다. Spring 2.0 AOP 는 Spring 1.2 AOP와 뒷단에서 완전하게 호환됩니다, 그리고 Spring 1.2 API들의 더 낮은 수준의 AOP 지원에 관하여서는 [이 챕터](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop-api.html)를 참고해 주세요.

스프링 프레임워크는 AOP를 다음과 같은 경우에 이용합니다.
* 선언적인 엔터프라이즈 서비스를 제공합니다. 특히 EJB 선언적 서비스를 대체합니다. 그러한 서비스들 중 가장 중요한 것은 [선언적 트랜잭션 관리](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/transaction.html#transaction-declarative)입니다.
* 자신만의 OOP를 AOP로 보완할 수 있도록 맞춤형 aspects를 작성하는 것을 사용자에게 허용합니다.

> 만약 인반적인 선언적 서비스나 풀링 같은 이미 ㅍ키징 된 선언적 미들웨어 서비스에만 관심이 있다면, 스프링 AOP로 직접 작업할 필요가 없으며 이 챕터의 대부분은 건너 뛰어도 좋습니다.

### 11.1.1 AOP 컨셉
몇가지 중요한 AOP의 컨셉과 용어에 대해 짚고 넘어 가겠습니다. 유감스럽게도 이 용어들은 스프링에 특화되어 있지 않습니다, AOP의 용어들은 특별히 직관적이지 않습니다. 그러나 스프링이 고유의 용어를 사용하고 있을 경우에는 더욱 혼란스러울 것입니다.

* <i>Aspect</i> : 여러 클래스들을 가로 지르는 문제의 모듈화. 트랜잭션 관리는 엔터프라이즈 자바 애플리케이션에서 가로지르기 문제의 좋은 예시입니다. 스프링 AOP에서 Aspects는 통상적인 클래스([스키마 기반 접근](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-schema)을 활용하거나, (`@AspectJ`[형식](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-ataspectj))의 `@Aspect`) 어노테이션을 사용하여 구현합니다.
* <i>Join Point</i> : 메소드의 실행, 예외 처리등과 같은 프로그램 수행 상의 어떤 지점입니다. 스프링AOP에서 join point는 <i>언제나</i> 메소드의 실행으로 표현됩니다.
* <i>Advice</i> : 특정한 join point에서 aspect에 의해 일어나는 동작입니다. 다양한 타입의 advice는 "around", "before" 그리고 "after"를 포함합니다 (Advicte의 타입에 대해서는 아래에서 다룹니다.) 스프링을 포함하여 많은 AOP프레임워크들은 advice를 join point 주변의 인터셉터들의 사슬을 구성하는 <i>interceptor</i>로 만듭니다.
* <i>Pointcut</i> :
