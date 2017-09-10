원문 출처 : https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html

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

Spring AOP는 포괄적인 AOP 솔루션을 제공하기 위해 AspectJ와 경쟁하지는 않을 것입니다. 우리는 Spring AOP와 같은 프록시 Spring AOP와 같은 프록시 기반의 프레임워크와 AspectJ 같은 모든 것이 갖춰진 프레임워크가 모두 중요하다고 생각합니다, 그리고 둘은 경쟁 관계이기 보다는 보완 관계라고 믿습니다. Spring은 일관된 스프링 기반 애플리케이션 구조에 전적으로 AOP를 적용하기 위해 AspectJ로 Spring AOP와 Spring IoC를 탄탄하게 결합시켰습니다. 이 결합은 Spring AOP API 혹은 AOP 연합 API에 영향을 미치지 않습니다: Spring AOP는 뒷단 호환으로 남아 있습니다. Spring AOP API에 관한 논의는 [이 챕터](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop-api.html)를 참고하세요.

> 스프링 프레임워크의 핵심적인 지침 중 하나는 <i>비침습성</i>입니다: 이것은 당신의 비즈니스/도메인 모델에 프레임워크 종속적인 클래스나 인터페이스를 강제해서는 안 된다는 사고 방식입니다. 그러나, 몇몇 부분에 있어서는 스프링프레임워크는 프레임워크 종속적인 의존성이 당신의 코드베이스에 선택사항으로 남길 수도 있습니다: 그러한 옵션들을 제공하는 이유는 특정 시나리오에서는 그것이 확실히 읽기 쉽거나 그러한 방법으로 특정 기능의 일부를 코드로 작성하기 쉽기 때문입니다. 스프링 프레임워크는 (거의) 언제나 선택의 여지를 남깁니다. : 당신은 자유롭게 특정한 경우나 시나리오에 대해 최적의 선택을 내릴 수 있습니다.
이번 챕터와 밀접한 관련이 있는 그러한 선택 중 하나는 AOP 프레임워크 (및 AOP 스타일)을 선택하는 것입니다. 당신은 AspectJ 및/혹은 Spring AOP를 택할 수 있습니다, 또한 @AspectJ 어노테이션 스타일의 접근 방식 혹은 Spring XML 설정 방식의 스타일을 선택할 수도 있습니다. 이 챕터가 @AspectJ 스타일의 접근 방식을 먼저 소개하기로 한 것은 Spring 팀이 @AspectJ 어노테이션 스타일의 접근 방식을 Spring XML 설정 스타일보다 선호한다는 것을 의미하지는 않습니다.
각 스타일을 언제, 어디서 사용해야 하는가에 대한 좀 더 완전한 논의는 [섹션 11.4. "어떤 AOP선언 스타일을 사용할지 선택하기"](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-choosing)를 참고하세요.

### 11.1.3 AOP 프록시
Spring AOP는 기본적으로 표준 JDK <i>동적 프록시</i>를 AOP 프록시로 사용합니다. 이것은 어떠한 인터페이스(혹은 인터페이스들의 집합)든 프록시화 시키는 것을 가능케 합니다.
Spring AOP는 CGLIB 프록시를 사용할 수도 있습니다. 이것은 인터페이스보다는 클래스를 프록시화하는 데 필요합니다. CGLIB는 비즈니스 객체가 인터페이스를 구현하지 못할 때 기본적으로 사용됩니다. 프로그램에게는 클래스보다는 인터페이스가 좋은 관습이므로 비즈니스 클래스는 일반적으로 하나 이상의 비즈니스 인터페이스를 구현합니다. [CGLIB의 사용을 강제](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-proxying)할 수도 있습니다, 이런 경우는 (드물지만) 인터페이스에서 선언되지 않은 메소드에 advise하거나 프록시 된 객체를 구체적인 타입의 메소드에 전달할 필요가 있을 때입니다.
Spring AOP는 <i>프록시 기반</i>이라는 것을 인식하는 것이 중요합니다. [섹션 11.6.1. "AOP 프록시 이해하기"](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-understanding-aop-proxies)를 참조하여 이러한 구현이 구체적으로 무엇을 의미하는지 예제를 살펴보세요.

### 11.2 @AspectJ 지원
@AspectJ는 어노테이션이 붙은 일반적인 자바 클래스로 aspect를 선언하는 스타일을 의미합니다. @AspectJ 스타일은 AspectJ 5 릴리즈의 일부로 [AspectJ 프로젝트](https://www.eclipse.org/aspectj)에서 소개되었습니다. 스프링은 pointcut 파싱과 매칭을 위해 AspectJ에서 지원되는 라이브러리를 활용하여, AspectJ 5와 동일한 어노테이션을 해석합니다. AOP 런타임은 여전히 Spring AOP입니다, 또한 AspectJ 컴파일러나 weaver에 의존성은 없습니다.

> AspectJ 컴파일러나 weaver를 사용하는 것은 완전한 AspectJ 언어 사용을 활성화합니다, 이에 대해서는 [섹션 11.8. "스프링 애플리케이션에서 AspectJ 사용하기"](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html#aop-using-aspectj)에서 논의합니다.

스프링 설정에서 @AspectJ aspect를 사용하기 위해 @AspectJ aspect 기반의 Spring AOP 설정을 하기 위한 스프링 지원을 활성화 시키고, 그 aspect들에게 advice되었는지 여부에 기반하여 빈들을 <i>autoproxying</i>해야 합니다. autoproxying이란 Bean이 하나 이상의 Aspect에 의해 advise 되었다고 스프링이 정하면, 자동으로 빈이 메소드 실행을 가로채고 원하는대로 advice가 실행된다고 확인하는 프록시를 자동으로 생성하는 것을 의미합니다.

@AspectJ 지원은 XML 혹은 자바 스타일 설정에 의해 활성화 될 수 있습니다. 각 케이스별로 당신은 AspectJ의 `aspectjweaver.jar` 라이브러리가 어플리케이션의 클래스패스에 존재하는지 확인할 필요가 있습니다 (version 1.6.8 혹은 그 이상). 이 라이브러리는 AspectJ 배포판의 `'lib'` 디렉토리 혹은 메이븐 중앙 저장소에서 사용 가능합니다.

#### 자바 설정으로 @AspectJ 지원 활성화하기
자바 `@Configuration`으로 @AspectJ 지원을 활성화하기 위해서는 `@EnableAspectJAutoProxy` 어노테이션을 추가해야 합니다.
```Java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

#### XML 설정으로 @AspectJ 지원 활성화하기
XML 기반 설정으로 @AspectJ 설정을 활성화하기 위해서는 `aop:aspectj-autoproxy` 요소를 사용하세요
```xml
<aop:aspectj-autoproxy/>
```
이 설정은 당신이 [챕터 41. XML 스키마 기반 설정](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/xsd-configuration.html)에서 설명하는 스키마 지원을 사용한다고 가정합니다. [섹션 41.2.7. "aop 스키마"](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/xsd-configuration.html#xsd-config-body-schemas-aop)를 참조하여 `aop` 네임스페이스에 태그를 import하는 방법을 알아보세요.

### 11.2.2 aspect 선언하기
@AspectJ 지원이 활성화 되면, 어플리케이션 컨텍스트에 @AspectJ aspect(`@AspectJ` 어노테이션을 가진) 클래스로 정의된 bean들은 자동적으로 스프링에게 탐색되어 Spring AOP를 설정하는데 사용됩니다. 아래의 예제는 그다지 not-very-useful-aspect를 위해 최소한으로 필요한 정의를 보여줍니다.

`@Aspect` 어노테이션을 가지는 bean 클래스를 가리키는, 애플리케이션 컨텍스트에 존재하는 일반적인 bean 정의:
```XML
<bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
    <!-- 여기에 일반적인 aspect 속성들을 설정하세요 -->
</bean>
```
그리고 `org.aspectj.lang.annotation.Aspect` 애노테이션이 붙은 `NotVeryUsefulAspect` 클래스 정의입니다.
```Java
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```
Aspects (`@Aspect` 어노테이션이 붙은 클래스) 는 다른 클래스들과 마찬가지로 메소드와 필드를 가질 것입니다. pointcut, advice 및 (타입 간) 선언 소개를 포함할 것입니다.

> aspect 클래스들을 Spring XML 설정에 일반적인 bean으로 등록하거나, 다른 Spring이 관리하는 bean들과 마찬가지로, 클래스 패스 탐색을 통해 자동 검출할 수 있습니다. 그러나 @Aspect 어노테이션은 클래스패스 자동 검출을 위해서는 <i>충분하지 않다</i>라는 것을 기억하십시오, 그러한 목적으로 개별적인 @Component 어노테이션(혹은 스프링의 컴포넌트 스캐너의 규칙을 만족하는, 대안으로 사용되는 맞춤형 스테레오 타입 어노테이션)을 추가할 필요가 있습니다.

> Spring AOP에서, 자기 자신이 다른 aspect의 advice 대상이 되는 aspect를 가지는 것은 불가능합니다. 클래스에 붙은 @Aspect 어노테이션은 이것을 aspect로 표시합니다, 그러므로 이것은 auto-proxying에서 제외됩니다.

### 11.2.3 pointcut 선언하기
pointcut은 관심을 가진 join point를 결정하고, 그리하여 언제 advice가 실행될지를 제어할 수 있도록 한다는 것을 떠올려 보십시오. <i>Spring AOP는 스프링 bean에 대한 메소드 실행 join point만을 지원합니다</i>, 그러므로 pointcut을 스프링 bean에서 일치하는 메소드의 실행으로 생각할 수 있습니다. pointcut 선언은 두 가지 부분을 가집니다: 이름과 파라미터들로 구성된 시그니처와 우리가 관심을 가지는 메소드 실행이 <i>정확히</i> 어떤 것인지를 결정하는 pointcut 표현식입니다. @AspectJ 어노테이션 스타일의 AOP에서는, pointcut 시그니처는 일반적인 메소드 정의로 이루어집니다, 그리고 pointcut 표현식은 `@Pointcut` 어노테이션을 이용하여 지시됩니다.(pointcut 시그니처를 제공하는 메소드는 <i>반드시</i> `void` 타입을 반환해야 합니다)
