# Spring Boot 3.x (2022 ~)

> Java 17 베이스라인, `javax`→`jakarta` 대전환, 그리고 **GraalVM 네이티브 이미지 1급 지원**. 4년 반 만의 메이저 업그레이드로 Spring을 클라우드 네이티브 시대에 맞춰 다시 정렬했다.

## 릴리스 정보
- **최초 출시**: Spring Boot 3.0 GA — 2022년 11월 24일
- **주요 마이너 버전과 시기**:
  - 3.0 (2022-11) — Spring Framework 6, Java 17, Jakarta EE 9, GraalVM 네이티브, 관측성 신규
  - 3.1 (2023-05) — Docker Compose 지원, Testcontainers 통합, Spring Authorization Server 1.1
  - 3.2 (2023-11) — Java 21 지원, **가상 스레드(Virtual Threads)**, `RestClient`/`JdbcClient`, CRaC, Spring Framework 6.1
  - 3.3 (2024-05) — CDS(Class Data Sharing) 통합, Docker Compose에서 Bitnami 이미지 지원, 보안/관측성 개선
  - 3.4 (2024-11) — 구조화된 로깅(structured logging), `RestClient`/`RestTemplate` 클라이언트 자동 설정 확장, 기본 Buildpacks 빌더 변경(Paketo `builder-jammy-java-tiny`)
  - 3.5 (2025-05) — 3.x 후속 라인 (Testcontainers/Docker Compose SSL 구성, 설정/Actuator/빌드 개선)
- **기반 Spring Framework 버전**: Spring Framework 6.x (3.0은 6.0, 3.2는 6.1)
- **최소 자바 버전**: **Java 17** (가상 스레드 등 일부 기능은 Java 21 필요)

## 시대적 배경 (왜 3.0인가)

2.0 출시(2018) 이후 4년 반, JVM과 클라우드 환경은 다시 한번 크게 바뀌었다.

- **Java LTS의 세대교체**: Java 8/11 시대에서 **Java 17 LTS**(2021)가 새 기준점이 되었다. record, sealed class, 패턴 매칭, `var` 등 현대 문법을 전제할 수 있게 되었다.
- **Java EE → Jakarta EE 전환**: Oracle이 Java EE를 Eclipse Foundation에 이관하면서, 상표 문제로 패키지 네임스페이스가 `javax.*` → `jakarta.*`로 강제 변경되었다. 생태계 전체가 이 전환을 따라야 했다.
- **서버리스/컨테이너 비용 압박**: 빠른 기동과 낮은 메모리 사용이 비용으로 직결되며, **네이티브 이미지(GraalVM)**와 빠른 스타트업이 핵심 경쟁력이 되었다.
- **관측성 표준화**: 메트릭에 더해 **분산 추적(distributed tracing)**까지 표준 요구사항이 되었다.

Spring Boot 3.0은 Spring Framework 6.0을 기반으로 이 변화들을 한꺼번에 흡수했다. 그만큼 **마이그레이션 비용이 큰** 메이저 버전이다.

## 핵심 기능

### Java 17 베이스라인
최소 요구 버전이 Java 17로 올라갔다. record를 DTO/설정 객체로, sealed class를 도메인 모델로 자유롭게 쓸 수 있다.

```java
public record UserDto(Long id, String name, String email) {}

@RestController
class UserController {
    @GetMapping("/users/{id}")
    UserDto get(@PathVariable Long id) {
        return new UserDto(id, "Alice", "alice@example.com");
    }
}
```

### Jakarta EE 9 (`javax` → `jakarta`)
모든 Java EE API import가 `jakarta.*`로 변경되었다. 이는 3.x 마이그레이션에서 가장 광범위한 변화다.

```java
// Spring Boot 2.x
import javax.persistence.Entity;
import javax.servlet.http.HttpServletRequest;
import javax.validation.constraints.NotNull;

// Spring Boot 3.x
import jakarta.persistence.Entity;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.validation.constraints.NotNull;
```

JPA, 서블릿, Bean Validation, JMS 등을 쓰는 모든 코드와 서드파티 라이브러리가 Jakarta 호환 버전이어야 한다.

### GraalVM 네이티브 이미지 1급 지원 (AOT)
Spring Framework 6의 **AOT(Ahead-of-Time) 처리 엔진**을 내장해, GraalVM Native Image로 OS 네이티브 실행 파일을 빌드할 수 있다. 기동 시간이 수십 밀리초로 줄고 메모리 사용량과 이미지 크기가 크게 감소한다.

```bash
# 네이티브 실행 파일 빌드
mvn -Pnative native:compile

# 또는 네이티브 컨테이너 이미지(Buildpacks)
mvn -Pnative spring-boot:build-image
```

빌드 시점에 빈 구성과 프록시를 미리 계산해 리플렉션/동적 프록시 사용을 최소화한다. 서버리스·콜드스타트 민감 워크로드에 적합하다.

### 관측성 통합 (Observability — Micrometer Tracing)
2.x의 Micrometer 메트릭에 더해, **Micrometer Observation API**와 **Micrometer Tracing**(구 Spring Cloud Sleuth의 후신)을 통합했다. 하나의 `Observation`으로 메트릭과 트레이스를 함께 생산하며 OpenTelemetry/Zipkin으로 내보낸다.

```java
@Service
class OrderService {
    private final ObservationRegistry registry;

    void place(Order order) {
        Observation.createNotStarted("order.place", registry)
            .observe(() -> process(order)); // 메트릭 + 트레이스 동시 계측
    }
}
```

### 가상 스레드 (Virtual Threads, 3.2 / Java 21)
Java 21의 가상 스레드(Project Loom)를 설정 한 줄로 활성화한다. 블로킹 MVC 코드를 거의 그대로 두고도 높은 동시성을 얻을 수 있다.

```properties
spring.threads.virtual.enabled=true
```

### RestClient / JdbcClient (3.2)
`RestTemplate`을 대체하는 현대적 **동기 HTTP 클라이언트** `RestClient`(플루언트 API, WebClient와 유사한 사용감)와, 간결한 `JdbcClient`가 도입되었다. Boot가 `RestClient.Builder`를 자동 구성한다.

```java
RestClient client = RestClient.create();
User user = client.get()
    .uri("https://api.example.com/users/{id}", 1)
    .retrieve()
    .body(User.class);
```

### 개발/테스트 생산성 (3.1)
- **Docker Compose 지원**: `compose.yaml`이 있으면 `bootRun` 시 의존 컨테이너(DB 등)를 자동 기동/종료한다.
- **Testcontainers 1급 통합**: 테스트에서 컨테이너 기반 의존성을 선언적으로 구성(`@ServiceConnection`).

```yaml
# compose.yaml — bootRun 시 자동 기동
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    ports: ["5432:5432"]
```

## 마이너 버전별 변화
- **3.0 (2022)**: Java 17, Jakarta EE 9, Spring Framework 6, GraalVM 네이티브/AOT, 관측성(Observation/Tracing), `spring.factories` 자동 설정 → `AutoConfiguration.imports` 전환 완료.
- **3.1 (2023)**: Docker Compose 지원, Testcontainers 통합(`@ServiceConnection`), SSL 번들, Spring Authorization Server 1.1.
- **3.2 (2023)**: Java 21·가상 스레드, `RestClient`/`JdbcClient`, CRaC(Coordinated Restore at Checkpoint), Spring Framework 6.1.
- **3.3 (2024)**: CDS로 기동 가속, Docker Compose에서 Bitnami 이미지 지원, 보안/관측성 보강.
- **3.4 (2024)**: 구조화된 로깅(JSON 로그), `RestClient`/`RestTemplate`용 HTTP 클라이언트(Reactor Netty/JDK) 자동 설정, 기본 Buildpacks 빌더를 Paketo `builder-jammy-base`에서 `builder-jammy-java-tiny`로 변경.
- **3.5 (2025)**: Testcontainers·Docker Compose SSL 구성 지원, 설정·Actuator·빌드 도구 다듬기 등 후속 개선.

## 영향과 의의
- **클라우드 네이티브로의 재정렬**: 네이티브 이미지·가상 스레드·관측성 표준 통합으로, Spring을 서버리스/컨테이너 비용 모델에 맞게 현대화했다.
- **Jakarta 전환의 분수령**: 생태계 전체가 `javax`→`jakarta`로 넘어가는 강제 분기점이 되었고, 3.x로의 이동은 단순 버전 업이 아니라 의존성 전수 점검을 요구했다.
- **현대 Java 전제**: record·sealed·패턴 매칭을 자연스럽게 쓰는 코드 스타일을 표준화했고, Kotlin과의 시너지도 더 깊어졌다.

## 참고 출처
- [Spring Boot 3.0 Goes GA (spring.io blog)](https://spring.io/blog/2022/11/24/spring-boot-3-0-goes-ga/)
- [Spring Boot 3 and Spring Framework 6 ... GraalVM (InfoQ)](https://www.infoq.com/news/2022/11/spring-6-spring-boot-3-launch/)
- [Spring Boot 3.0 Release Notes (GitHub wiki)](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Release-Notes)
- [Spring Boot 3.2 — Virtual Threads, RestClient, JdbcClient (InfoQ)](https://www.infoq.com/news/2023/12/spring-boot-virtual-threads/)
- [Spring Boot 3.2 and Spring Framework 6.1 — Java 21, Virtual Threads, CRaC (InfoQ)](https://www.infoq.com/articles/spring-boot-3-2-spring-6-1/)
- [Spring Boot 3.x Features: Complete Guide (danvega.dev)](https://www.danvega.dev/blog/spring-boot-3-features)
- [Spring Boot 3.5.0 available now (spring.io blog)](https://spring.io/blog/2025/05/22/spring-boot-3-5-0-available-now/)
