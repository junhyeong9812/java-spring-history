# Spring Framework 5.x (2017 ~)

> 리액티브 프로그래밍(WebFlux/Reactor)과 **Kotlin 1급 지원**이라는 두 축을 동시에 도입한 세대. 명령형 일변도였던 Spring에 비동기·논블로킹과 함수형 스타일이 정식으로 들어왔다.

## 릴리스 정보
- 최초 출시: 5.0 — 2017년 9월
- 주요 마이너 버전과 시기: 5.0(2017-09) → 5.1(2018-09) → 5.2(2019-09/10) → 5.3(2020-10, 장기 지원)
- 최소 자바 버전(baseline): Java 8 이상. 5.1에서 **Java 11** 정식 지원. 5.3.0 GA(2020-10)는 JDK 8-15 대응으로 출시됐고, **JDK 17 실행 지원은 이후 5.3.x 유지보수 라인에서 추가**됐다(5.3.x는 JDK 21에서도 실행 가능하나, 프레임워크 차원의 가상 스레드 지원은 6.1부터다).
- Java EE / Jakarta EE 기준: Java EE 7 베이스라인(Servlet 3.1, JPA 2.1, Bean Validation 1.1), Java EE 8 호환. **여전히 `javax.*` 네임스페이스 사용**(jakarta 전환은 6.0).

## 시대적 배경
2017년경 백엔드 화두는 **리액티브/논블로킹**이었다. Node.js의 이벤트 루프 모델, RxJava의 확산, 그리고 Reactive Streams 표준(JDK 9의 `java.util.concurrent.Flow`로 편입) 등이 배경이다. 적은 스레드로 높은 동시성을 처리하려는 수요가 커졌고, Spring은 기존 서블릿 기반 `spring-webmvc`와 별개로 **논블로킹 웹 스택 `spring-webflux`**를 새로 만들었다.

또 하나의 큰 흐름은 **Kotlin**이다. 2017년 Google이 Android 공식 언어로 Kotlin을 채택하며 JVM 진영에서 폭발적으로 성장했다. Spring 5는 Kotlin을 단순 호환이 아니라 **1급 시민(first-class)**으로 지원하기로 결정했다 — 널 안정성, 확장 함수, 함수형 빈/라우터 DSL까지. 코드베이스 baseline도 Java 8로 올려 람다·`CompletableFuture`를 적극 활용했고, 테스트는 JUnit 5(Jupiter)를 지원했다.

## 핵심 추가/변경 기능

### Spring WebFlux — 리액티브 웹 스택
서블릿 블로킹 모델에 의존하지 않는 완전 논블로킹 웹 프레임워크. **Reactor**(`Mono`/`Flux`)를 기본 리액티브 라이브러리로 사용하고, Netty/Undertow 같은 논블로킹 런타임 위에서 동작한다. `spring-webmvc`와 공존하며 같은 어노테이션 모델도 쓸 수 있다.

명령형(MVC) vs 리액티브(WebFlux) 비교:

```java
// 기존 Spring MVC — 블로킹, 반환은 구체 타입
@GetMapping("/{id}")
public Account get(@PathVariable long id) {
    return accountService.find(id);     // 스레드가 결과까지 블록
}
```

```java
// Spring WebFlux — 논블로킹, 반환은 Mono/Flux (어노테이션 모델)
@GetMapping("/{id}")
public Mono<Account> get(@PathVariable long id) {
    return accountService.find(id);     // 구독 시점에 비동기 실행
}

@GetMapping("/stream")
public Flux<Account> stream() {
    return accountService.findAll();    // 0..N 스트리밍
}
```

### 함수형 웹 엔드포인트 (RouterFunction)
어노테이션 대신 함수(라우터 + 핸들러)로 라우팅을 구성하는 새 모델.

```java
@Bean
public RouterFunction<ServerResponse> routes(AccountHandler handler) {
    return route()
        .GET("/accounts/{id}", handler::get)
        .GET("/accounts",      handler::list)
        .POST("/accounts",     handler::create)
        .build();
}
```

> 참고: 위 `route()` 플루언트 빌더는 Spring Framework **5.1**에서 추가됐다. 5.0에서는 `RouterFunctions.route(GET("/accounts/{id}"), handler::get).andRoute(...)` 형태를 사용했다.

### WebClient — 리액티브 HTTP 클라이언트
블로킹 `RestTemplate`을 대체하는 논블로킹 클라이언트.

```java
WebClient client = WebClient.create("https://api.example.com");

Mono<Account> account = client.get()
    .uri("/accounts/{id}", 42)
    .retrieve()
    .bodyToMono(Account.class);
```

---

### Kotlin 1급 지원 — Spring 5의 또 다른 핵심
Spring 5는 Kotlin을 깊이 통합했다. 단순히 "Kotlin에서 호출 가능"한 수준이 아니라, **Kotlin 언어 기능에 맞춰 API와 DSL을 설계**했다.

**1) 널 안정성(null-safety)** — Spring API에 JSR-305 기반 `@Nullable`/`@NonNull` 메타 어노테이션을 부착해, Kotlin 컴파일러가 Spring API의 널 가능 여부를 타입 시스템으로 인식한다.

**2) 확장 함수(extension functions)** — Kotlin에서 더 자연스러운 API 제공. 예: `getBean<T>()` 처럼 reified 제네릭을 활용.

```kotlin
// Java: ctx.getBean(AccountService::class.java)
val service = ctx.getBean<AccountService>()   // Kotlin reified 확장 함수
```

**3) 함수형 빈 정의 DSL(bean definition DSL)** — `@Configuration`/`@Bean` 없이 람다로 빈을 등록.

```kotlin
val beans = beans {
    bean<AccountServiceImpl>()
    bean<JdbcAccountDao>()
    bean {
        WebClient.create("https://api.example.com")
    }
}
```

**4) 라우터 DSL(router function DSL)** — WebFlux 라우팅을 Kotlin DSL로.

```kotlin
val routes = router {
    "/accounts".nest {
        GET("/{id}", handler::get)
        GET("", handler::list)
        POST("", handler::create)
    }
}
```

**5) 코루틴 지원(5.2)** — 컨트롤러/핸들러를 `suspend` 함수로 작성하면 Spring이 내부적으로 Reactor와 연결한다.

```kotlin
@GetMapping("/{id}")
suspend fun get(@PathVariable id: Long): Account =
    accountService.find(id)          // suspend, 논블로킹

@GetMapping("/stream")
fun stream(): Flow<Account> =        // Kotlin Flow (5.2+)
    accountService.findAll()
```

---

### JUnit 5(Jupiter) 지원
새 `SpringExtension`으로 JUnit 5와 통합. `@ExtendWith(SpringExtension.class)` 또는 `@SpringJUnitConfig`로 테스트 컨텍스트를 구성한다(JUnit 4도 계속 지원).

### 기타
- 코어를 Java 8 baseline으로 정비(인터페이스 default 메서드, `@FunctionalInterface` 다수).
- `spring-jcl` 도입(자체 Commons Logging 브릿지), 로깅 의존성 정리.
- 5.1: Reactor Netty 0.8 기반 HTTP/2 지원, Kotlin beans DSL 정제, JDK 11 지원.
- 5.2: Kotlin 코루틴/`Flow`, R2DBC(리액티브 관계형 DB) 초기 통합, 성능 개선.
- 5.3: 데이터 바인딩·검증 개선, RSocket 안정화. (JDK 17 실행 지원은 GA 이후 5.3.x 유지보수 라인에서 추가)

## 설정 스타일의 변화
설정의 기본은 이미 4.x/Boot에서 **Java Config + 어노테이션 + Boot 자동 구성**으로 굳어졌다. 5.x가 더한 것은 **함수형/DSL 스타일**이다.
- Java: `RouterFunction`로 라우팅을 함수로 표현.
- Kotlin: `beans { }` DSL과 `router { }` DSL로, 어노테이션 없이도 컨테이너와 웹 라우팅을 선언적으로 구성. 리플렉션·어노테이션 처리 비용을 줄여 시작 속도에도 유리.

즉 "XML → 어노테이션 → Java Config"의 흐름에 **"함수형 DSL"**이라는 선택지가 추가됐다.

## 마이너 버전별 변화
- 5.0 (2017-09): **WebFlux/Reactor**, 함수형 라우팅, WebClient, **Kotlin 1급 지원(널 안정성·확장 함수·beans/router DSL)**, JUnit 5 지원, Java 8 baseline, `javax.*` 유지.
- 5.1 (2018-09): **JDK 11 지원**, Reactor Netty 0.8 기반 HTTP/2, Kotlin beans DSL 정제, 성능·로깅 개선.
- 5.2 (2019-09/10): **Kotlin 코루틴/`Flow` 지원**, R2DBC 통합, RSocket 지원, 관측용 Reactor checkpoint.
- 5.3 (2020-10): 장기 지원(LTS급) 버전. RSocket·데이터 바인딩·CORS 개선. GA 시점은 JDK 8-15 대응이고 **JDK 17 실행 지원은 이후 5.3.x 유지보수 라인에서 추가**됐다(5.3.x는 JDK 21에서 실행 가능하나, 프레임워크 차원의 가상 스레드 지원 API는 6.1부터).

## 영향과 의의
- **리액티브 프로그래밍**을 Spring 생태계의 정식 옵션으로 만들었다. WebFlux·WebClient·R2DBC·RSocket로 풀 논블로킹 스택 구성이 가능해졌다(단, 명령형 MVC가 사라진 것은 아니며 둘은 공존).
- **Kotlin을 JVM 백엔드의 주류 언어로 끌어올린 분기점.** Spring의 1급 지원 덕분에 Kotlin + Spring Boot 조합이 실무에서 폭넓게 채택됐고, 코루틴 지원으로 "리액티브를 명령형처럼" 쓰는 길이 열렸다.
- 함수형 빈/라우터 DSL과 Java 8 baseline 정비는 6.x의 AOT·네이티브 이미지 최적화로 가는 사전 작업이기도 하다.

## 참고 출처
- [Spring Framework 5.0 Release Notes (GitHub wiki)](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-5.0-Release-Notes)
- [Spring Framework 5.1 Release Notes (GitHub wiki)](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-5.1-Release-Notes)
- [Spring Framework 5.2 Release Notes (GitHub wiki)](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-5.2-Release-Notes)
- [Spring Framework 5.3 - versionlog](https://versionlog.com/spring-framework/5.3/)
- [Spring Framework - Wikipedia](https://en.wikipedia.org/wiki/Spring_Framework)
