# Spring Boot 2.x (2018 ~)

> Spring Framework 5 기반의 대약진. 리액티브(WebFlux), Micrometer 기반 관측성, 그리고 **Kotlin 1급 지원**으로 "현대 JVM 백엔드"의 표준을 다시 썼다.

## 릴리스 정보
- **최초 출시**: Spring Boot 2.0 GA — 2018년 3월 1일
- **주요 마이너 버전과 시기**:
  - 2.0 (2018-03) — Spring Framework 5, WebFlux, Micrometer, Kotlin 1급 지원, HikariCP 기본
  - 2.1 (2018-10)
  - 2.2 (2019-10) — Java 13 지원, RSocket, JUnit 5 기본
  - 2.3 (2020-05) — 그레이들 기반 OCI 이미지 빌드(Buildpacks), Liveness/Readiness 프로브, Graceful shutdown
  - 2.4 (2020-11) — `application.yml` 설정 파일 처리 방식 개편(config data API), 도커 이미지/k8s 강화
  - 2.5 (2021-05) — SQL 초기화 개편, 환경변수 prefix
  - 2.6 (2021-11) — Actuator 보강, 순환 참조 기본 금지
  - 2.7 (2022-05) — 2.x 마지막 피처 라인, 3.0 준비(점진적 마이그레이션 가이드 제공)
- **기반 Spring Framework 버전**: Spring Framework 5.x (2.0은 5.0, 2.7은 5.3)
- **최소 자바 버전**: **Java 8** (Java 8 베이스라인, 이후 9~17 지원 확대)

## 시대적 배경 (왜 2.0인가)

1.x가 "설정 자동화"로 성공을 거둔 사이, JVM 생태계는 빠르게 변했다.

- **리액티브 프로그래밍**의 부상: 고동시성·논블로킹 I/O 수요가 커지며 Reactor/RxJava 기반 모델이 주목받았다.
- **관측성(Observability)** 요구 증가: 마이크로서비스가 보편화되며 Prometheus, Datadog 등 다양한 모니터링 백엔드와의 연동이 필수가 되었다.
- **Kotlin의 약진**: 2016년 Kotlin 1.0, 2017년 안드로이드 1급 언어 채택으로 JVM 진영에서 Kotlin이 급부상했다.
- **기반 변화**: Spring Framework 5.0(2017)이 Reactor 기반 리액티브 스택과 Kotlin 지원을 정식 도입했다.

Spring Boot 2.0은 이 Spring Framework 5의 새 능력을 자동 설정으로 흡수해, 명령형(MVC)과 리액티브(WebFlux)를 한 프레임워크에서 선택할 수 있게 했다.

## 핵심 기능

### Spring WebFlux (리액티브 웹 스택)
서블릿 블로킹 모델 대신 Reactor 기반 논블로킹 스택을 제공한다. `Mono`/`Flux`를 반환값으로 쓰며, 기본 서버는 Netty다. 애너테이션 방식과 함수형(WebFlux.fn) 방식을 모두 자동 설정한다.

```java
@RestController
public class UserController {
    private final UserRepository repo; // ReactiveCrudRepository

    @GetMapping("/users")
    public Flux<User> all() {
        return repo.findAll();         // 논블로킹 스트림
    }

    @GetMapping("/users/{id}")
    public Mono<User> one(@PathVariable String id) {
        return repo.findById(id);
    }
}
```

함수형 라우터(WebFlux.fn) 예시:

```java
@Bean
RouterFunction<ServerResponse> routes(UserHandler handler) {
    return route(GET("/users"), handler::all)
        .andRoute(GET("/users/{id}"), handler::one);
}
```

### Actuator 재설계 + Micrometer
2.0은 자체 메트릭 API를 버리고 **Micrometer**(메트릭 파사드, "메트릭계의 SLF4J")로 전면 교체했다. 하나의 계측 코드로 Prometheus, Datadog, Influx, Graphite, JMX, New Relic 등 다양한 백엔드에 내보낸다.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

엔드포인트 경로도 `/actuator/*` 아래로 통일되고 노출 정책이 명시적(opt-in)으로 바뀌었다.

### Kotlin 1급 지원
Spring Framework 5의 Kotlin 지원을 Boot 차원에서 흡수했다. **start.spring.io의 언어 선택지에 Kotlin이 정식 포함**되었고, 생성 시 `kotlin-spring`/`kotlin-jpa` 컴파일러 플러그인과 Kotlin용 스타터 의존성이 자동 구성된다.

```kotlin
@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
    runApplication<DemoApplication>(*args)   // Boot가 제공하는 Kotlin 확장 함수
}

@RestController
class HelloController {
    @GetMapping("/")
    fun hello() = "Hello, Kotlin + Spring Boot 2"
}
```

(Kotlin 채택사와 적용 방식은 `kotlin-and-spring.md`에서 깊게 다룬다.)

### HikariCP 기본 커넥션 풀
기본 JDBC 커넥션 풀이 Tomcat JDBC Pool에서 **HikariCP**로 교체되었다. 더 빠르고 가벼운 풀이 표준이 되었다.

### 설정 프로퍼티 바인딩 개선
타입 안전 바인딩 엔진이 새로 작성되어 relaxed binding 규칙이 정리되고, `@ConstructorBinding`을 통한 불변(immutable) 설정 객체 바인딩이 가능해졌다.

```java
@ConfigurationProperties("app")
@ConstructorBinding
public class AppProperties {
    private final String name;
    private final int retries;
    public AppProperties(String name, int retries) { ... }
}
```

### 그 외
- **Java 8 베이스라인**: 람다/스트림/`java.time` 전제. 이후 마이너에서 9~17까지 지원 확대.
- **OCI 이미지 빌드(2.3+)**: Dockerfile 없이 Cloud Native Buildpacks로 컨테이너 이미지를 생성(`mvn spring-boot:build-image`).
- **Graceful shutdown / Liveness·Readiness 프로브(2.3+)**: 쿠버네티스 환경 대응.

## 마이너 버전별 변화
- **2.0 (2018)**: WebFlux, Micrometer, Kotlin 1급, HikariCP 기본, Actuator/바인딩 재설계.
- **2.1 (2018)**: 빈 오버라이딩 기본 비활성화, Actuator/JDBC 보강.
- **2.2 (2019)**: JUnit 5 기본, RSocket 지원, Java 13 대응, lazy initialization 옵션.
- **2.3 (2020)**: Buildpacks 이미지 빌드, k8s 프로브, graceful shutdown, layered jar.
- **2.4 (2020)**: 설정 파일 처리 방식(config data) 개편, `spring.config.import` 도입.
- **2.5 (2021)**: SQL 스크립트 초기화 정비, 환경변수 prefix, Docker 이미지 개선.
- **2.6 (2021)**: 순환 참조 기본 금지, Actuator/`@ConfigurationProperties` 스캔 개선.
- **2.7 (2022)**: 2.x 마지막 라인. `spring.factories` 기반 자동 설정에서 신규 임포트 파일 방식으로의 전환을 시작하며 3.0 마이그레이션 길을 닦음.

## 영향과 의의
- 명령형(MVC)과 리액티브(WebFlux)를 한 지붕 아래 두어, 워크로드 특성에 맞는 스택 선택을 표준화했다.
- Micrometer 채택으로 Spring 애플리케이션의 **관측성**이 벤더 중립적으로 표준화되었고, 이는 3.x의 분산 추적(Tracing)으로 이어진다.
- **Kotlin을 1급 시민으로 받아들여**, 이후 "Kotlin + Spring Boot"가 주류 조합 중 하나로 자리잡는 결정적 전환점이 되었다.
- 4년 반 동안(2018~2022) 사실상 업계 표준 백엔드 플랫폼으로 군림했다.

## 참고 출처
- [Spring Boot 2.0 goes GA (spring.io blog)](https://spring.io/blog/2018/03/01/spring-boot-2-0-goes-ga/)
- [Spring Boot 2.0 Release Notes (GitHub wiki)](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes)
- [Spring releases Spring Boot 2.0 (SD Times)](https://sdtimes.com/webdev/spring-releases-spring-boot-2-0/)
- [Spring Boot 2.0 Goes GA — Phil Webb interview (InfoQ)](https://www.infoq.com/news/2018/03/spring-boot-2.0-release-ga-webb)
- [Spring Boot version history (codejava.net)](https://www.codejava.net/frameworks/spring-boot/spring-boot-version-history)
