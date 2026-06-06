# Spring Framework 6.x (2022 ~)

> Java 17 baseline과 **`javax.*` → `jakarta.*` 네임스페이스 대전환**으로 한 세대를 끊은 버전. AOT/GraalVM 네이티브 이미지와 옵저버빌리티(Micrometer)를 1급으로 끌어올렸고, 6.1에서 가상 스레드를 받아들였다.

## 릴리스 정보
- 최초 출시: 6.0 — 2022년 11월 16일
- 주요 마이너 버전과 시기: 6.0(2022-11) → 6.1(2023-11) → 6.2(2024-11)
- 최소 자바 버전(baseline): **Java 17** (6.x 최초로 Java 17을 강제. 6.1은 JDK 21을 first-class 지원, 6.2는 JDK 17-25 범위 지원)
- Java EE / Jakarta EE 기준: **Jakarta EE 9 baseline, Jakarta EE 10 호환** (6.2는 Jakarta EE 9-10, JDK 17-25 범위)

## 시대적 배경
6.0의 배경은 자바 생태계의 두 가지 큰 단절이다.

1. **Jakarta EE 네임스페이스 전환** — Oracle이 Java EE를 Eclipse 재단에 이관(Jakarta EE)하면서, 상표 문제로 모든 표준 API 패키지가 `javax.*` → `jakarta.*`로 강제 변경됐다(Jakarta EE 9, 2020). 이는 단순 리네이밍이 아니라 import·서드파티 라이브러리·서블릿 컨테이너까지 모두 갈아엎어야 하는 호환성 단절이다. Spring 6.0은 이 전환을 정면으로 수용했다.

2. **Java 17 LTS 확산** — 2021년 Java 17 LTS가 나오며 레코드·sealed 클래스·패턴 매칭 등 현대 문법이 자리 잡았다. Spring 6은 과감히 baseline을 Java 17로 올려 낡은 호환 코드를 제거하고 새 언어 기능을 활용한다.

여기에 클라우드 네이티브 수요(빠른 시작·낮은 메모리)에 맞춰 **AOT 컴파일/GraalVM 네이티브 이미지**를 1급으로 지원하고, 분산 추적·메트릭을 위한 **Micrometer 기반 옵저버빌리티**를 코어에 통합했다.

## 핵심 추가/변경 기능

### javax → jakarta 네임스페이스 대전환 (6.0의 핵심)
Servlet, JPA, Bean Validation, JMS, Annotations 등 모든 표준 API import가 바뀐다. 이것이 5.x → 6.x 마이그레이션의 가장 큰 장벽이다.

```java
// Spring 5.x (Java EE / javax)
import javax.servlet.http.HttpServletRequest;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.validation.constraints.NotNull;

// Spring 6.x (Jakarta EE / jakarta)
import jakarta.servlet.http.HttpServletRequest;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.validation.constraints.NotNull;
```

```java
@Entity                                  // jakarta.persistence.Entity
public class Account {
    @Id                                  // jakarta.persistence.Id
    private Long id;

    @NotNull                             // jakarta.validation.constraints.NotNull
    private String name;
}
```

영향 범위: 서블릿 컨테이너도 Jakarta 지원 버전 필요(Tomcat 10+, Jetty 11+ 등), 모든 서드파티 라이브러리도 jakarta 호환 버전으로 올려야 한다.

### Java 17 baseline
레코드, sealed 클래스, 텍스트 블록, 패턴 매칭 등을 프레임워크와 애플리케이션에서 자연스럽게 사용. 예를 들어 record를 그대로 DTO/설정 바인딩에 활용.

```java
public record AccountDto(Long id, String name, BigDecimal balance) {}

@GetMapping("/{id}")
public AccountDto get(@PathVariable Long id) { ... }   // record 직렬화
```

### AOT(Ahead-Of-Time) 처리 / GraalVM 네이티브 이미지
빌드 시점에 빈 정의·프록시·리플렉션 메타데이터를 미리 분석·생성(AOT)하여, GraalVM으로 **네이티브 실행 파일**을 만든다. 결과적으로 시작 시간 수십 ms, 메모리 대폭 절감 — 서버리스·컨테이너 환경에 유리. (Spring Boot 3.0이 이 AOT 엔진을 빌드 플러그인으로 노출한다.)

- 런타임 리플렉션 대신 빌드 타임 메타데이터 → 네이티브 이미지에서 동작 보장
- AOT 처리된 컨텍스트의 **테스트**(TestContext AOT 지원)는 6.0부터 제공되며, 6.1에서 `failOnError` 옵션 등으로 보강됐다

### 옵저버빌리티(Observability) — Micrometer 통합
분산 추적과 메트릭을 위한 `Micrometer Observation API`를 코어에 통합. WebFlux/WebMVC 요청, `RestTemplate`/`WebClient`, `@Scheduled` 등에 일관된 관측 지점을 제공한다.

```java
Observation.createNotStarted("account.lookup", registry)
    .observe(() -> accountService.find(id));
```

### HTTP Interface 클라이언트 (선언적 HTTP)
인터페이스 + 어노테이션만으로 HTTP 클라이언트를 선언(Spring Data Repository와 유사한 방식). 내부적으로 WebClient(또는 6.1의 RestClient) 기반.

```java
public interface AccountClient {
    @GetExchange("/accounts/{id}")
    Account get(@PathVariable long id);
}
// HttpServiceProxyFactory로 구현체 생성
```

### RestClient — 동기 유창형 HTTP 클라이언트 (6.1)
`RestTemplate`의 인프라를 쓰면서 `WebClient`처럼 유창한(fluent) API를 제공하는 **동기** 클라이언트. 블로킹 코드에서 WebClient의 리액티브 부담 없이 현대적 API를 쓰게 해준다.

```java
RestClient client = RestClient.create("https://api.example.com");

Account account = client.get()
    .uri("/accounts/{id}", 42)
    .retrieve()
    .body(Account.class);
```

### 가상 스레드(Virtual Threads) 지원 (6.1)
Java 21의 가상 스레드(Project Loom)를 지원. 요청당 스레드(thread-per-request) 모델을 유지하면서도 블로킹 I/O를 값싸게 처리 — 리액티브의 복잡함 없이 높은 동시성을 얻는 대안. (Spring Boot 3.2에서 `spring.threads.virtual.enabled=true`로 Tomcat/Jetty에 적용.)

### 기타 (6.1 / 6.2)
- 6.1: **RestClient**, **JdbcClient**(유창형 JDBC API), 가상 스레드, `@Scheduled` Micrometer 계측, AOT 테스트 보강(`failOnError` 등 — AOT 컨텍스트 테스트 자체는 6.0 도입), Reactor/Jackson 등 의존성 현대화, JDK 21 지원.
- 6.2: 빈 백그라운드 초기화·컨테이너 시작 성능 개선, `@Fallback` 빈, 널 안정성 개선, Jackson/검증/메시징 업데이트, JDK 17-25 범위 지원. (이후 7.0이 2025-11에 등장하며 다음 세대로 이어진다.)

## 설정 스타일의 변화
설정 모델 자체(Java Config + 어노테이션 + Boot 자동 구성 + 함수형 DSL)는 5.x에서 정립된 것을 계승한다. 6.x의 변화는 **"무엇을 import 하느냐"와 "어떻게 빌드/실행하느냐"**에 있다.
- 패키지 네임스페이스가 `jakarta.*`로 전면 교체 — 코드 레벨의 가장 큰 차이.
- **AOT를 전제로 한 설정** — 동적 리플렉션·런타임 빈 등록보다, 빌드 타임에 정적으로 분석 가능한 구성이 권장된다(네이티브 이미지 친화). Kotlin/Java의 함수형 빈 DSL이 이런 면에서 유리.

## 마이너 버전별 변화
- 6.0 (2022-11): **Java 17 baseline**, **`javax→jakarta` 전환(Jakarta EE 9+)**, AOT/GraalVM 네이티브 지원, Micrometer 옵저버빌리티, HTTP Interface 클라이언트, ProblemDetail(RFC 7807) 지원.
- 6.1 (2023-11): **RestClient**, JdbcClient, **가상 스레드(JDK 21)** 지원, `@Scheduled` 관측, AOT 테스트, JDK 21 정식 지원.
- 6.2 (2024-11): 컨테이너 시작 성능·백그라운드 초기화 개선, `@Fallback` 빈, 널 안정성·검증·Jackson 개선, JDK 17-25 / Jakarta EE 9-10 지원.

## 영향과 의의
- **`javax→jakarta` 전환**은 Spring 역사상 가장 큰 호환성 단절이었지만, 동시에 레거시 정리의 분기점이 됐다. 5.x → 6.x 마이그레이션은 단순 버전업이 아니라 의존성·컨테이너·import 전반의 현대화를 강제했다.
- **AOT/네이티브 이미지**로 Spring이 서버리스·클라우드 네이티브의 빠른 시작·저메모리 요구에 정면으로 대응했다.
- **가상 스레드** 지원은 "고동시성을 위해 반드시 리액티브여야 한다"는 전제를 깼다 — 명령형 코드를 그대로 두고도 확장성을 얻는 새로운 길을 열었다. 6.x는 리액티브(WebFlux)와 가상 스레드라는 두 동시성 모델을 모두 품은 세대다.

## 참고 출처
- [Spring Framework 6.0 goes GA (spring.io blog)](https://spring.io/blog/2022/11/16/spring-framework-6-0-goes-ga/)
- [What's New in Spring Framework 6.x (GitHub wiki)](https://github.com/spring-projects/spring-framework/wiki/What's-New-in-Spring-Framework-6.x)
- [Spring Boot 3.2 and Spring Framework 6.1 Add Java 21, Virtual Threads, and CRaC (InfoQ)](https://www.infoq.com/articles/spring-boot-3-2-spring-6-1/)
- [Observability Support :: Spring Framework (docs)](https://docs.spring.io/spring-framework/reference/integration/observability.html)
- [Spring Framework - Wikipedia](https://en.wikipedia.org/wiki/Spring_Framework)
