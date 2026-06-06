# Spring Framework 1.x (2004 ~)

> EJB 없이도 엔터프라이즈 자바를 만들 수 있다는 것을 증명한 출발점. IoC/DI 컨테이너와 AOP, 일관된 추상화 계층으로 "경량 컨테이너" 시대를 열었다.

## 릴리스 정보
- 최초 출시: 1.0 GA — 2004년 3월 24일 (Apache 2.0 라이선스, 초기 코드 공개는 2003년 6월)
- 주요 마이너 버전과 시기: 0.9(2003-06-26 SourceForge 공개) → 1.0(2004-03) → 1.1(2004) → 1.2(2005) — 책 *Expert One-on-One J2EE*는 2002년 출간
- 최소 자바 버전(baseline): J2SE 1.3 / 1.4 (주류는 어노테이션·제네릭 미사용이나, 1.2부터 JDK 1.5에서 `@Transactional` 어노테이션 사용 가능)
- Java EE / Jakarta EE 기준: J2EE 1.3 / 1.4 (Servlet 2.3/2.4, EJB 2.x 시대)

## 시대적 배경
2000년대 초반 자바 엔터프라이즈 개발의 표준은 EJB(Enterprise JavaBeans) 2.x였다. EJB는 분산 트랜잭션·보안·영속성을 제공했지만, 다음과 같은 고질적 문제가 있었다.

- Home/Remote 인터페이스, 디플로이먼트 디스크립터(XML) 등 과도한 보일러플레이트
- 컨테이너에 강하게 결합되어 단위 테스트가 사실상 불가능
- 무거운 애플리케이션 서버가 반드시 필요

Rod Johnson은 2002년 저서 *Expert One-on-One J2EE Design and Development*에서 "EJB 없이 POJO(Plain Old Java Object)로 엔터프라이즈 애플리케이션을 만들 수 있다"고 주장하며 직접 작성한 인프라 코드를 공개했다. 이 코드가 발전하여 Spring Framework가 되었고, 1.0은 그 첫 정식 결과물이다. 핵심 메시지는 단순했다. **프레임워크가 객체의 생성과 의존성 연결을 책임지고(IoC), 개발자는 비즈니스 로직(POJO)에 집중한다.**

## 핵심 추가/변경 기능

### IoC / DI 컨테이너 (BeanFactory, ApplicationContext)
Spring의 심장. 객체(빈)의 생성·초기화·의존성 주입·소멸 생명주기를 컨테이너가 관리한다.

- `BeanFactory`: 가장 기본적인 IoC 컨테이너 (지연 로딩)
- `ApplicationContext`: BeanFactory를 확장해 메시지 소스, 이벤트 발행, AOP, 리소스 로딩 등을 추가한 상위 컨테이너

대표적인 XML 빈 정의(setter 주입):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- applicationContext.xml -->
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN"
    "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url"             value="jdbc:mysql://localhost/test"/>
        <property name="username"        value="root"/>
        <property name="password"        value="secret"/>
    </bean>

    <bean id="accountDao" class="com.example.dao.JdbcAccountDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 생성자 주입(constructor injection)도 지원 -->
    <bean id="accountService" class="com.example.service.AccountServiceImpl">
        <constructor-arg ref="accountDao"/>
    </bean>

</beans>
```

```java
// 컨테이너 부트스트랩
ApplicationContext ctx =
    new ClassPathXmlApplicationContext("applicationContext.xml");
AccountService service = (AccountService) ctx.getBean("accountService");
```

EJB와의 결정적 차이: `AccountServiceImpl`은 어떤 Spring 인터페이스도 구현하지 않는 순수 POJO이며, 컨테이너 없이도 `new`로 만들어 단위 테스트할 수 있다.

### AOP (관점 지향 프로그래밍)
트랜잭션·로깅·보안 같은 횡단 관심사(cross-cutting concern)를 비즈니스 코드와 분리한다. 1.x 시절에는 어노테이션이 아니라 XML + 프록시 기반으로 구성했다.

```xml
<bean id="loggingAdvice" class="com.example.aop.LoggingInterceptor"/>

<bean id="accountServiceProxy"
      class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target"            ref="accountServiceTarget"/>
    <property name="interceptorNames">
        <list>
            <value>loggingAdvice</value>
        </list>
    </property>
</bean>
```

`MethodBeforeAdvice`, `AfterReturningAdvice`, `MethodInterceptor` 등의 어드바이스 인터페이스를 구현하는 방식이었다.

### 선언적 트랜잭션 추상화
EJB의 CMT(Container-Managed Transaction)를 대체. `PlatformTransactionManager` 인터페이스 하나로 JDBC, JTA, Hibernate 등 백엔드와 무관하게 동일한 방식으로 트랜잭션을 다룬다. 1.x에서는 `TransactionProxyFactoryBean`으로 선언적 트랜잭션을 적용했다.

```xml
<bean id="transactionManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="accountService"
      class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="target"             ref="accountServiceTarget"/>
    <property name="transactionAttributes">
        <props>
            <prop key="transfer*">PROPAGATION_REQUIRED</prop>
            <prop key="get*">PROPAGATION_REQUIRED,readOnly</prop>
        </props>
    </property>
</bean>
```

### JDBC 추상화 (JdbcTemplate)
JDBC의 반복 코드(Connection/Statement/ResultSet 열고 닫기, 예외 처리)를 템플릿 메서드 패턴으로 제거. 체크 예외인 `SQLException`을 런타임 예외 계층(`DataAccessException`)으로 변환해 일관성을 높였다.

```java
public class JdbcAccountDao {
    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource ds) {
        this.jdbcTemplate = new JdbcTemplate(ds);
    }

    public int getBalance(long id) {
        return jdbcTemplate.queryForInt(
            "SELECT balance FROM account WHERE id = ?",
            new Object[] { new Long(id) });
    }
}
```

### ORM / 데이터 접근 통합
Hibernate, JDO, iBATIS 등을 위한 통합 템플릿(`HibernateTemplate` 등)과 일관된 예외 변환을 제공했다.

### 경량 MVC (Spring Web MVC)
`DispatcherServlet`을 프런트 컨트롤러로 하는 웹 MVC 프레임워크. 1.x에서는 `Controller` 인터페이스를 구현하고 XML에 핸들러 매핑을 등록하는 방식이었다(어노테이션 기반 컨트롤러는 2.5부터).

## 설정 스타일의 변화
이 시대의 주류는 **XML 구성**이다(Java 1.4 기준에서는 어노테이션·제네릭을 쓰지 않았다). 다만 1.2부터 JDK 1.5 환경에서는 `@Transactional` 어노테이션을 쓸 수 있었다. 대부분의 빈, 의존성, AOP, 트랜잭션은 XML에 명시적으로 선언되었다. DTD 기반 검증을 사용했으며, 이 장황함이 훗날 2.x의 XML 네임스페이스 간소화와 3.x의 Java Config 등장 동기가 된다.

## 마이너 버전별 변화
- 1.0 (2004-03): 첫 정식 릴리스. IoC 컨테이너, AOP, JDBC/트랜잭션 추상화, Web MVC, ORM 통합 등 핵심 모듈 확립.
- 1.1 (2004): 컨테이너·AOP 안정화 및 성능 개선.
- 1.2 (2005): JDK 1.5(Java 5) 런타임 지원 강화 및 **`@Transactional` 등 JDK 1.5 트랜잭션 어노테이션 도입**, AOP 개선. (소스 레벨 메타데이터 기반 Commons Attributes 트랜잭션은 1.0부터 제공됐다.) 이후 어노테이션 시대로 가는 징검다리.

## 영향과 의의
- **EJB 2.x의 대안**으로 자리 잡으며 "경량 컨테이너(lightweight container)" 패러다임을 대중화했다. 훗날 EJB 3.0(2006)이 의존성 주입·POJO·어노테이션을 받아들인 것은 Spring의 영향이 컸다.
- IoC/DI를 자바 엔터프라이즈의 표준 설계 원칙으로 정착시켰고, **테스트 가능성(testability)**을 1급 관심사로 끌어올렸다.
- JdbcTemplate·트랜잭션 추상화·예외 변환 등 "일관된 추상화 + 템플릿" 패턴은 이후 모든 Spring 모듈의 설계 철학이 되었다.

## 참고 출처
- [Spring Framework - Wikipedia](https://en.wikipedia.org/wiki/Spring_Framework)
- [History of Spring Framework and Spring Boot](https://www.quickprogrammingtips.com/spring-boot/history-of-spring-framework-and-spring-boot.html)
- [Introduction to the Spring Framework | TheServerSide](https://www.theserverside.com/news/1364527/Introduction-to-the-Spring-Framework)
- [Spring framework version history - codejava.net](https://www.codejava.net/frameworks/spring/spring-framework-version-history)
- [Spring Framework, History, and Its Structure - DEV Community](https://dev.to/jeanv0/spring-framework-history-and-its-structure-361)
