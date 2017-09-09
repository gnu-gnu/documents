# 11. 스프링에서의 관점 지향 프로그래밍

## 11.1 들어가기
<i>관점 지향 프로그래밍(AOP)</i>은 프로그램 구조에 대해 다른 사고 방식을 제공하여 객체 지향 프로그래밍(OOP)을 보완하는 것입니다. OOP에서 모듈성의 핵심 단위는 클래스인 반면, AOP에서는 <i>관점(aspect)</i>입니다. 관점은 다양한 타입이나 객체를 관통하는 트랜잭션 관리와 같은 고려사항에 대한 모듈화를 가능케 합니다. (이러한 고려사항들은 AOP에서는 종종 <i>crosscutting</i>이라는 용어로 표현 됩니다.)

스프링의 핵심 컴포넌트 중 하나는 AOP프레임워크입니다. 스프링 IoC 컨테이너는 AOP에 의존하지 않으므로 원하지 않는다면 AOP를 쓰지 않아도 좋습니다. AOP는 매우 뛰어난 미들웨어 솔루션을 제공하여 Spring IoC를 보완합니다.

> 스프링 2.0 AOP<br/>
(번역자 주 : aspectj, pointcut, weaving 등은 실제 코딩에서도 사용되므로 원어로 표현하였습니다.)<br/>
스프링 2.0은 간단하면서도 매우 강력한 [스키마 기반 접근](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-schema) 혹은 [@AspectJ 형식 어노테이션](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-ataspectj)을 사용하여 맞춤형 aspects를 작성하는 방법을 소개하였습니다. 두 스타일 모두 여전히 Spring AOP를 weaving에 사용하면서 완전한 타입의 advice이며, AspectJ pointcut 언어를 사용합니다.
Spring 2.0의 스키마- 혹은 @AspectJ 기반 AOP 지원을 이번 챕터에서 다룰 것입니다. Spring 2.0 AOP 는 Spring 1.2 AOP와 뒷단에서 완전하게 호환됩니다, 그리고 Spring 1.2 API들의 더 낮은 수준의 AOP 지원에 관하여서는 [이 챕터](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop-api.html)를 참고해 주세요.

스프링 프레임워크는 AOP를 다음과 같은 경우에 이용합니다.
* 선언적인 엔터프라이즈 서비스를 제공합니다. 특히 EJB 선언적 서비스를 대체합니다. 그러한 서비스들 중 가장 중요한 것은 [선언적 트랜잭션 관리](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/transaction.html#transaction-declarative)입니다.
* 자신만의 OOP를 AOP로 보완할 수 있도록 맞춤형 aspects를 작성하는 것을 사용자에게 허용합니다.

> 만약 일반적인 선언적 서비스나 풀링 같은 이미 패키징 된 선언적 미들웨어 서비스에만 관심이 있다면, 스프링 AOP로 직접 작업할 필요가 없으며 이 챕터의 대부분은 건너 뛰어도 좋습니다.

### 11.1.1 AOP 컨셉
몇가지 중요한 AOP의 컨셉과 용어에 대해 짚고 넘어 가겠습니다. 유감스럽게도 이 용어들은 스프링에 특화되어 있지 않습니다, AOP의 용어들은 특별히 직관적이지 않습니다. 그러나 스프링이 고유의 용어를 사용하고 있을 경우에는 더욱 혼란스러울 것입니다.

* <i>Aspect</i> : 여러 클래스들을 가로 지르는 문제의 모듈화. 트랜잭션 관리는 엔터프라이즈 자바 애플리케이션에서 가로지르기 문제의 좋은 예시입니다. 스프링 AOP에서 Aspects는 통상적인 클래스([스키마 기반 접근](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-schema)을 활용하거나, (`@AspectJ`[형식](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-ataspectj))의 `@Aspect`) 어노테이션을 사용하여 구현합니다.
* <i>Join Point</i> : 메소드의 실행, 예외 처리등과 같은 프로그램 수행 상의 어떤 지점입니다. 스프링AOP에서 join point는 <i>언제나</i> 메소드의 실행으로 표현됩니다.
* <i>Advice</i> : 특정한 join point에서 aspect에 의해 일어나는 동작입니다. 다양한 타입의 advice는 "around", "before" 그리고 "after"를 포함합니다 (Advicte의 타입에 대해서는 아래에서 다룹니다.) 스프링을 포함하여 많은 AOP프레임워크들은 advice를 join point 주변의 인터셉터들의 사슬을 구성하는 <i>interceptor</i>로 만듭니다.
* <i>Pointcut</i> : join point들과 일치하는지 여부를 판단하는 조건들의 집합입니다. Advice는 pointcut 표현식과 연결되며 pointcut과 일치하는 어떠한 join point에서 구동됩니다. (예를 들어, 특정한 이름을 가지는 메소드). 포인트컷 표현식과 일치하는 join point들은 AOP의 핵심입니다, 그리고 스프링은 AspectJ pointcut표현식 언어를 기본적으로 사용합니다.
* <i>introduction</i> : 타입을 대신하여 추가적인 메소드나 필드를 선언합니다. 스프링AOP는 새로운 인터페이스(혹은 상응하는 구현체)를 검토된 객체(<i>advised object</i>)에 추가할 수 있도록 합니다. 예를 들어, bean이 캐싱을 단순화하기 위해 `IsModified` 인터페이스를 구현하도록 introduction을 사용할 수 있습니다. (introduction은 타입간 선언으로 AspectJ 커뮤니티에서 알려져 있습니다.)
* <i>Target object</i> : 하나 이상의 aspect에서 검토하는 객체입니다. 검토된 객체(<i>advised object</i>)라고도 합니다. 스프링AOP는 런타임 프록시들을 이용하여 구현되었기 때문에, 이 객체들은 언제나 <i>프록시 된</i> 객체입니다.
* <i>AOP Proxy</i> : aspect 조건 실행(메소드 실행 검토 및 그 외의 작업) 구현하기 위하여 AOP프레임워크에 의해 생성된 객체입니다. 스프링 프레임워크에서 AOP Proxy는 JDK 다이내믹 프록시 혹은 CGLIB 프록시입니다.
* Weaving : 검토된 객체를 만들기 위하여 aspect를 다른 애플리케이션 타입이나 객체들과 연결합니다. 이 작업은 (예를 들자면 AspectJ 컴파일러를 이용한) 컴파일 타임이나, 로드 타임, 혹은 런타임에 완료될 수 있습니다. 다른 순수 Java AO프레임워크처럼 Spring AOP도 런타임에 weaving을 수행합니다.

어드바이스의 타입
* <i>Before advice</i> : join point 전에 수행되는 advice입니다, 그러나 (예외가 발생하지 않는 이상) join point로 진행되는 실행을 막을 수는 없습니다.
* <i>After returning advice</i> : join point가 일반적으로 끝난 뒤 수행되는 advice입니다. 예를 들어, 메소드가 예외를 발생시키지 않고 결과를 리턴하였을 때를 말합니다.
* <i>After throwing advice</i> : 메소드가 예외를 발생시키며 종료되었을 때 수행되는 advice입니다.
* <i>After (finally) advice</i> : (일반적이든 예외적인 리턴이 돌아왔든) join point가 어떤 식으로 종료되었든지 상관하지 않고 실행되는 advice입니다.
* <i>Around advice</i> : 메소드 실행과 같은 join point를 둘러싸는 advice입니다. Around advice는 메소드의 실행 전,후에 맞춤형 동작을 수행할 수 있습니다. 또한 join point로 진행을 할지 혹은 자체적인 값이나 예외를 반환하여 권고하는 메소드 실행으로 바로 진행할지를 결정하는 역할을 합니다.

Around advice는 가장 일반적인 종류의 advice입니다. 스프링AOP는, AspectJ와 마찬가지로, 모든 범위의 advice 타입들을 제공하기 때문에, 요구하는 행동을 구현할 수 있는 가장 협소한 advice type을 사용하는 것을 권고 드립니다. 예를 들어, 메소드의 반환값만을 캐시에 업데이트 할 필요가 있다면, around advice가 동일한 목적을 달성할 수 있다고 하더라도, after returning advice를 구현하는 것이 더 나을 것입니다. 가장 구체적인 타입의 advice 타입을 사용하는 것은 에러의 잠재적 가능성을 줄이고, 좀 더 단순한 프로그래밍 모델을 제공합니다. 예를 들어, around advice에서 사용하는 `JoinPoint` 메소드에 존재하는 `proceed()`를 호출할 필요가 없으므로 실행이 실패할 일도 없습니다.

스프링 2.0에서는, 모든 advice 파라미터가 정적인 타입이므로 `Object` 배열보다는 적절한 타입의 advice 파라미터를 사용해야 합니다. (예를 들자면, 메소드 실행의 결과 값의 파입)

(pointcut과 일치하는) join point의 컨셉은 가로채기만을 제공하는 다른 구식 기술들과 AOP를 구분 짓는 핵심입니다. pointcut은 advice를 객체지향 계층구조에 독립적으로 목표로 삼을 수 있게 합니다. 예를 들어 선언적 트랜잭션 관리를 제공하는 around advice는 여러개의 객체들에 맞춰진 메소드들의 집합에 제공될 수 있습니다. (서비스 레이어에서 모든 비즈니스 동작과 같은 것들)

### 11.1.2 스프링 AOP 적용 범위와 목표
Spring AOP는 순수 자바로 구현되었습니다. 특별한 수정 프로세스가 필요 없습니다. Spring AOP는 클래스로더 계층 구조를 컨트롤할 필요가 없습니다, 그러므로 서블릿 컨테이너나 다른 애플리케이션 서버에서 사용하기에 적합합니다.

Spring AOP는 현재 메소드 실행 join point(Spring beans상의 메소드들의 실행을 어드바이스)만을 지원합니다. 핵심적인 Spring AOP API를 깨지 않고도 Field를 가로채는 것이 가능하지만, Field를 가로채는 것은 구현되지 않았습니다. 만약 field 접근에 advise하기를 원하거나 join point를 새로 고칠 필요가 있다면, AspectJ와 같은 언어를 고려 해 보시기 바랍니다.

다른 대부분의 AOP프레임워크에 비해 Spring AOP의 AOP 접근방법은 차이가 있습니다. (Spring AOP가 꽤나 뛰어나지만) 가장 완전한 AOP구현을 제공하는 것이 목표는 아닙니다; 엔터프라이즈 어플리케이션의 공통적인 문제를 해결하는데 도움을 주기 위하여 AOP 구현과 Spring IoC 사이에 밀접한 통합을 제공하는 것에 더욱 더 중점을 두고 있습니다.

그러므로, 예를 들어서, 일반적으로 스프링 프레임워크의 AOP 기능은 Spring IoC 컨테이너와 함께 사용됩니다. Aspect는 일반적인 빈 정의 문법을 사용하여 설정 됩니다. (물론, 이것은 강력한 "자동 프록시" 능력을 부여합니다.): 이것은 다른 AOP 구현체와 비교하여 결정적인 차이점입니다. Spring AOP를 이용하여 쉽게 혹은 효율적으로 다룰 수 없는 것들도 있습니다, (전형적인 도메인 객체 같은) 매우 미세하게 다뤄지는 객체들에 advice하는 것입니다: AspectJ는 이런 경우 가장 좋은 선택입니다. 그러나 Spring AOP는 엔터프라이즈 자바 애플리케이션에서 발생하는 대부분의 문제에 대해 AOP로 처리하기 위한 훌륭한 해법을 제공한다는 것을 우리는 경험하였습니다.
