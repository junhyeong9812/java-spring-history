# Spring Framework 2.x (2006 ~)

> XML을 네임스페이스로 간소화하고, 어노테이션 기반 설정의 문을 연 전환기. 2.5에서 `@Autowired`와 컴포넌트 스캔이 등장하며 "XML 탈출"이 시작됐다.

## 릴리스 정보
- 최초 출시: 2.0 — 2006년 10월
- 주요 마이너 버전과 시기: 2.0(2006-10) → 2.5(2007-11)
- 최소 자바 버전(baseline): 2.0은 J2SE 1.3 이상, 2.5에서 1.3 지원을 제거하고 J2SE 1.4.2 이상 (어노테이션·제네릭 기능은 Java 5에서 활성화)
- Java EE / Jakarta EE 기준: J2EE 1.3 이상 호환 유지(Servlet 2.3/2.4), 2.5에서 Java EE 5(Servlet 2.5) 지원 강화

## 시대적 배경
2006년은 Java 5(2004)가 보급되며 **어노테이션·제네릭**이 본격적으로 쓰이기 시작한 시점이다. 같은 해 EJB 3.0이 발표되며 어노테이션과 DI를 받아들였는데, 이는 역설적으로 Spring이 옳았음을 증명했다. 한편 Spring 1.x의 가장 큰 불만은 **장황한 XML**이었다. 2.x는 이 두 흐름에 답한다.

1. XML을 더 읽기 쉽고 짧게 — XML 네임스페이스(스키마 기반 커스텀 태그) 도입
2. 어노테이션으로 XML 자체를 줄이기 — 2.5의 컴포넌트 스캔과 `@Autowired`

## 핵심 추가/변경 기능

### XML 네임스페이스 (구성 간소화)
DTD 대신 XSD 스키마를 채택하고, 도메인별 전용 네임스페이스(`<context:>`, `<aop:>`, `<tx:>`, `<jee:>`, `<util:>`)를 도입했다. 장황한 `ProxyFactoryBean`/`TransactionProxyFactoryBean` 선언이 한 줄로 줄었다.

이전(1.x)과 비교 — 선언적 트랜잭션:

```xml
<!-- 2.x: tx 네임스페이스 + AOP 포인트컷 -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="...">

    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="*"    propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="serviceOps"
            expression="execution(* com.example.service.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="serviceOps"/>
    </aop:config>
</beans>
```

### AspectJ 통합 (@AspectJ 스타일 AOP)
Spring 자체 AOP(프록시 기반)를 그대로 유지하면서, AspectJ의 포인트컷 표현식과 `@Aspect`(@AspectJ) 어노테이션 스타일을 추가로 채택. AOP가 훨씬 강력하고 표현력 있게 바뀌었다.

```xml
<aop:aspectj-autoproxy/>
```

```java
@Aspect
public class LoggingAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object logTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        System.out.println(pjp.getSignature() + " took "
            + (System.currentTimeMillis() - start) + "ms");
        return result;
    }
}
```

### 어노테이션 기반 설정의 시작 (2.5)
2.5는 Spring 역사에서 결정적인 전환점이다. **컴포넌트 스캔과 어노테이션 의존성 주입**이 도입되어, 빈을 더 이상 XML에 일일이 등록하지 않아도 됐다.

- `@Component`, `@Service`, `@Controller` — 2.5에서 추가된 스테레오타입 어노테이션 (`@Repository`는 이미 2.0부터 제공)
- `@Autowired` — 타입 기반 자동 주입
- `@Qualifier` — 동일 타입 다중 빈 구분
- `<context:component-scan>` — 패키지 스캔으로 빈 자동 등록

```xml
<!-- XML은 스캔 지시만 남는다 -->
<beans xmlns:context="http://www.springframework.org/schema/context" ...>
    <context:component-scan base-package="com.example"/>
</beans>
```

```java
@Repository
public class JdbcAccountDao implements AccountDao { /* ... */ }

@Service
public class AccountServiceImpl implements AccountService {

    private final AccountDao accountDao;

    @Autowired
    public AccountServiceImpl(AccountDao accountDao) {
        this.accountDao = accountDao;   // 생성자 자동 주입
    }
}
```

### 어노테이션 기반 Spring MVC 컨트롤러 (2.5)
1.x의 `Controller` 인터페이스 구현 방식을 대체하는 `@Controller` / `@RequestMapping` 모델 도입. 오늘날 우리가 쓰는 Spring MVC의 원형이다.

```java
@Controller
public class AccountController {

    @Autowired
    private AccountService accountService;   // 타입 기반 자동 주입

    @RequestMapping("/account/view")
    public String view(@RequestParam("id") long id, ModelMap model) {
        model.addAttribute("account", accountService.find(id));
        return "accountView";   // 뷰 이름
    }
}
```

### JPA 지원
Java EE 5의 JPA(Java Persistence API)를 지원하는 `JpaTemplate`, `LocalContainerEntityManagerFactoryBean`, `@PersistenceContext` 주입 등을 제공.

### 빈 스코프 확장
기존 `singleton`/`prototype`에 더해 웹 환경용 `request`, `session`, `globalSession` 스코프와 커스텀 스코프 등록 기능 추가.

### 기타
- `<bean>`에 대한 라이프사이클 콜백 어노테이션(`@PostConstruct`/`@PreDestroy`, JSR-250) 지원
- 동적 언어(Groovy, JRuby, BeanShell)로 빈 작성 지원
- 메시지 기반 POJO 등 JMS 개선

## 설정 스타일의 변화
**"XML 일색"에서 "XML + 어노테이션 혼합"으로** 넘어가는 과도기.
- 2.0: 여전히 XML 중심이지만 네임스페이스로 훨씬 간결해짐.
- 2.5: 컴포넌트 스캔 + `@Autowired`로 빈 정의가 코드로 이동. XML에는 인프라성 설정과 `<context:component-scan>`만 남기는 스타일이 유행하기 시작. (단, 빈을 100% 자바 코드로 정의하는 `@Configuration`/`@Bean`은 아직 없음 — 3.0에서 등장)

## 마이너 버전별 변화
- 2.0 (2006-10): XML 네임스페이스(`<aop:>`, `<tx:>` 등), `@AspectJ` 스타일 AOP, `@Repository`(예외 변환 스테레오타입), 새 빈 스코프, JPA 지원, 동적 언어 빈.
- 2.5 (2007-11): **`@Autowired`, `@Component`/`@Service`/`@Controller`(나머지 스테레오타입), 컴포넌트 스캔, 어노테이션 기반 MVC(`@RequestMapping`)**, JSR-250(`@PostConstruct`) 지원, 통합 테스트용 TestContext 프레임워크 도입.

## 영향과 의의
- XML 네임스페이스로 1.x의 가장 큰 약점(장황함)을 크게 완화했다.
- **2.5의 어노테이션 도입은 Spring 설정 패러다임의 분기점**이다. 이후 "설정을 어디에 둘 것인가(XML vs 어노테이션 vs Java Config)" 논쟁의 출발점이 되었고, 3.x의 Java Config와 훗날 Spring Boot의 "설정 최소화" 철학으로 이어진다.
- AspectJ 통합으로 AOP가 실무에서 본격적으로 쓰일 만큼 강력해졌다.

## 참고 출처
- [Spring Framework - Wikipedia](https://en.wikipedia.org/wiki/Spring_Framework)
- [Spring framework version history - codejava.net](https://www.codejava.net/frameworks/spring/spring-framework-version-history)
- [Spring Framework 2.0.8 Reference (PDF)](https://docs.spring.io/spring-framework/docs/2.0.8/spring-reference.pdf)
- [History of Spring Framework and Spring Boot](https://www.quickprogrammingtips.com/spring-boot/history-of-spring-framework-and-spring-boot.html)
- [Spring Framework Versions: Feature list by version](https://bluebirdinternational.com/spring-framework-versions/)
