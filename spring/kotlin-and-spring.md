# Kotlin과 Spring: 도입사와 적용 방식

> 코틀린이 언제, 어떻게 Spring에 들어왔는가 — 그리고 자바와 코틀린이 같은 프로젝트에서 어떻게 공존하는가

## 코틀린 언어 약사

코틀린(Kotlin)은 JetBrains(IntelliJ IDEA 제작사)가 만든 JVM 언어다. 이름은 러시아 상트페테르부르크 인근의 코틀린 섬에서 따왔다.

| 시기 | 사건 |
|------|------|
| 2011-07 | JetBrains가 JVM용 신규 언어 "Kotlin" 공개 발표 |
| 2012-02 | Apache License 2.0으로 오픈소스화 |
| 2016-02-15 | **Kotlin 1.0 출시** — 첫 공식 안정 버전, 하위 호환성 장기 보장 시작 |
| 2017-05 | Google I/O에서 **안드로이드 1급 언어로 공식 채택** (Android Studio 3.0, 2017-10부터 기본 포함) |
| 2018-10 | Kotlin 1.3 — **코루틴(coroutines) 안정화** |
| 2019-05 | Google, 안드로이드 개발에서 **"Kotlin 우선(Kotlin-first)"** 선언 |
| 2020 이후 | 1.4, 1.5, 1.6, 1.7, 1.8, 1.9 → 2.0(2024, K2 컴파일러) 으로 진화 |

코틀린의 핵심 매력은 **자바와의 100% 상호운용성**, **널 안정성(null-safety)**, **간결한 문법(data class, 확장 함수, 후행 람다 등)**, 그리고 **코루틴 기반 비동기**다. 자바 바이트코드로 컴파일되므로 기존 자바 라이브러리/프레임워크를 그대로 쓸 수 있다 — 이 점이 Spring이 코틀린을 빠르게 받아들일 수 있었던 토대다.

## 타임라인: Spring의 Kotlin 채택

| 시기 | 사건 |
|------|------|
| 2016 초 | **start.spring.io에 Kotlin 언어 옵션이 (실험적으로) 추가** — Spring 진영의 첫 공식 코틀린 발자국 |
| 2017-01-04 | spring.io 블로그 "Introducing Kotlin support in Spring Framework 5.0" 게시 |
| 2017-09 | **Spring Framework 5.0 GA — Kotlin 1급 지원 정식 포함** (널 안정성 메타데이터, 확장 함수, 빈 DSL, 라우터 DSL) |
| 2018-03 | **Spring Boot 2.0 GA — Kotlin 1급 지원** (start.spring.io 정식 Kotlin 옵션, 컴파일러 플러그인/스타터 자동 구성) |
| 2019-04 | 블로그 "Going Reactive with Spring, Coroutines and Kotlin Flow" — 코루틴/Flow 통합 발표 |
| 2019-09 | **Spring Framework 5.2 — 코루틴(suspend 함수) 정식 지원** (WebFlux 코루틴, `Flow` ↔ `Flux` 연동) |
| 2022~ | Spring Framework 6 / Spring Boot 3 — Java 17·Jakarta 기반에서 Kotlin 지원 지속 (코루틴/DSL 강화) |

> 주의: start.spring.io의 Kotlin 옵션은 Spring Framework 5.0 발표보다 **앞서(2016년경)** 등장했다. 즉 "프로젝트 생성기에서 먼저 실험 → 프레임워크 본체(5.0)에서 1급 지원 → Spring Boot 2.0에서 스타터·플러그인까지 정식 지원"의 순서로 정착했다. 다만 코루틴(`suspend`) 1급 지원은 그 다음 단계인 Spring Framework 5.2(2019)에서 별도로 추가된 것이므로, "Boot 2.0에서 코틀린 지원이 모두 끝났다"는 의미는 아니다.

## Spring이 제공하는 Kotlin 지원 기능

### 널 안정성(null-safety)과 플랫폼 타입
Spring은 모든 패키지에 **`@NonNull`/`@Nullable` 메타데이터**를 선언했다. 덕분에 코틀린 컴파일러가 Spring API의 널 가능성을 인식하고, 자바 타입을 "플랫폼 타입"으로 두루뭉술 넘기지 않고 정확한 코틀린 타입(`String` vs `String?`)으로 다룬다.

```kotlin
// Spring이 required 여부를 코틀린 널 가능성으로 추론한다.
@GetMapping("/greet")
fun greet(@RequestParam name: String): String = "Hi $name"      // 필수 파라미터
@GetMapping("/greet2")
fun greet2(@RequestParam name: String?): String = "Hi ${name ?: "stranger"}" // 선택 파라미터
```

`name: String`은 `required=true`, `name: String?`은 `required=false`로 자동 해석된다. 자바에서는 `@RequestParam(required=false)`를 명시해야 하는 부분이 타입으로 표현된다.

### 코틀린 확장 함수 (Spring이 추가한 확장들)
Spring은 기존 API를 건드리지 않고 코틀린 **확장 함수**로 더 관용적인 사용법을 제공한다. reified 타입 파라미터로 `Class<T>` 인자를 없앤다.

```kotlin
// 자바: restTemplate.getForObject(url, User::class.java)
val user = restTemplate.getForObject<User>(url)             // reified 확장

// WebClient
val users: Flow<User> = client.get().uri("/users")
    .retrieve().bodyToFlow<User>()

// ApplicationContext에서 빈 조회
val service = context.getBean<MyService>()
```

### 빈 등록 DSL (`beans { }`)
함수형으로 빈을 등록하는 코틀린 DSL. 리플렉션/CGLIB 없이 명시적으로 빈을 구성할 수 있어 네이티브 이미지와도 궁합이 좋다.

```kotlin
val myBeans = beans {
    bean<UserRepository>()
    bean<UserService>()
    bean {
        UserController(ref())          // ref()로 의존성 주입
    }
}

fun main(args: Array<String>) {
    runApplication<DemoApplication>(*args) {
        addInitializers(myBeans)
    }
}
```

### 함수형 라우터 DSL (`router { }`, WebMvc.fn / WebFlux.fn)
애너테이션 대신 코틀린 DSL로 라우팅을 선언한다.

```kotlin
@Bean
fun routes(handler: UserHandler) = router {
    "/users".nest {
        GET("", handler::all)
        GET("/{id}", handler::byId)
        POST("", handler::create)
    }
}
```

### 코루틴 지원 (suspend 함수, Spring 5.2+ / WebFlux 코루틴)
Spring Framework 5.2부터 컨트롤러/핸들러에서 **`suspend` 함수**와 **`Flow`**를 직접 쓸 수 있다. 내부적으로 Reactor의 `Mono`/`Flux`와 양방향 변환된다. 리액티브의 성능을 명령형처럼 읽히는 코드로 얻는다.

```kotlin
@RestController
class UserController(private val repo: UserRepository) {

    @GetMapping("/users/{id}")
    suspend fun byId(@PathVariable id: Long): User =      // suspend: Mono<User> 대응
        repo.findById(id) ?: throw NotFoundException()

    @GetMapping("/users")
    fun all(): Flow<User> = repo.findAll()                // Flow: Flux<User> 대응
}
```

코루틴 기반 Spring Data 리포지토리(`CoroutineCrudRepository`)도 제공된다.

## 자바와 코틀린의 공존

### 같은 프로젝트에서 혼용 (상호운용성)
코틀린은 자바 바이트코드로 컴파일되므로, **한 프로젝트에 자바 클래스와 코틀린 클래스를 섞어 둘 수 있다.** 코틀린에서 자바 빈을 주입받거나, 자바에서 코틀린 빈을 주입받는 데 제약이 거의 없다. 점진적 마이그레이션(자바 → 코틀린 부분 전환)이 가능한 이유다.

```kotlin
// 코틀린 서비스가 자바로 작성된 리포지토리를 주입받음
@Service
class OrderService(private val orderRepository: OrderRepository /* 자바 인터페이스 */) {
    fun total(id: Long): Long = orderRepository.findById(id).orElseThrow().amount
}
```

### Gradle/Maven 설정 — 코틀린 + 스프링 플러그인
Spring Boot 프로젝트에서 코틀린을 쓰려면 보통 다음 플러그인을 함께 적용한다.

```kotlin
// build.gradle.kts
plugins {
    id("org.springframework.boot") version "3.x.x"
    id("io.spring.dependency-management") version "1.x.x"
    kotlin("jvm") version "1.9.x"
    kotlin("plugin.spring") version "1.9.x"   // = all-open 래퍼
    kotlin("plugin.jpa") version "1.9.x"      // = no-arg 래퍼 (JPA 사용 시)
}

dependencies {
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin") // 코틀린 데이터 클래스 직렬화
    implementation("org.jetbrains.kotlin:kotlin-reflect")
}
```

플러그인 역할:
- **`kotlin-spring` (all-open 래퍼)**: 아래 "final class 문제" 참조.
- **`kotlin-jpa` (no-arg 래퍼)**: `@Entity`/`@Embeddable`/`@MappedSuperclass`에 합성 기본 생성자를 생성해 JPA의 "인자 없는 생성자" 요구를 충족.
- **`jackson-module-kotlin`**: data class를 기본 생성자 없이도 JSON 역직렬화.

### 코틀린에서 Spring 어노테이션 사용 시 주의점 — final class 문제와 all-open
코틀린은 **클래스와 멤버가 기본적으로 `final`**이다. 그런데 Spring은 `@Configuration`, `@Transactional`, AOP 등에서 **CGLIB 프록시(서브클래싱)**를 만들기 때문에, final 클래스는 프록시를 만들 수 없어 문제가 된다.

`kotlin-spring`(all-open) 플러그인이 이를 해결한다. Spring 스테레오타입 애너테이션(`@Component`, `@Configuration`, `@Service`, `@Repository`, `@Controller`, `@RestController` 등 `@Component` 메타 애너테이션이 붙은 것)이 달린 클래스와 그 멤버를 **자동으로 `open` 처리**한다. 따라서 개발자가 일일이 `open class`라고 쓰지 않아도 된다.

```kotlin
// kotlin-spring 플러그인이 없으면 직접 open을 붙여야 한다:
open class MyService { open fun work() { } }

// 플러그인이 있으면 그냥 이렇게 써도 프록시가 동작한다:
@Service
class MyService { fun work() { } }
```

마찬가지로 JPA 엔티티는 `kotlin-jpa`(no-arg)가 없으면 기본 생성자가 없어 Hibernate가 인스턴스를 만들지 못한다. 다만 `kotlin-jpa`는 인자 없는 생성자만 합성할 뿐 클래스를 `open`으로 만들지는 않는다. 게다가 `@Entity`는 Spring 스테레오타입이 아니어서 `kotlin-spring`(all-open)의 기본 대상에도 포함되지 않는다. 따라서 Hibernate의 지연 로딩(프록시 서브클래싱) 등을 위해 엔티티를 non-final로 두려면, `allOpen` 설정에 JPA 애너테이션(`jakarta.persistence.Entity`/`MappedSuperclass`/`Embeddable`)을 직접 지정하거나 클래스에 `open`을 붙여야 한다.

## 자바 vs 코틀린: 같은 Spring 코드 비교

### 엔티티 (Entity)
```java
// Java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private String email;

    protected User() {}                      // JPA용 기본 생성자
    public User(String name, String email) { this.name = name; this.email = email; }
    public Long getId() { return id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    // ... equals/hashCode/toString
}
```
```kotlin
// Kotlin (kotlin-jpa로 no-arg 생성자 생성;
//         지연 로딩 프록시를 쓰려면 allOpen에 @Entity 추가하거나 class에 open 필요)
@Entity
class User(
    var name: String,
    var email: String,
    @Id @GeneratedValue
    var id: Long? = null
)
```

### 컨트롤러 (Controller)
```java
// Java
@RestController
@RequestMapping("/users")
public class UserController {
    private final UserService service;
    public UserController(UserService service) { this.service = service; }

    @GetMapping("/{id}")
    public UserDto get(@PathVariable Long id) {
        return service.findById(id);
    }
}
```
```kotlin
// Kotlin
@RestController
@RequestMapping("/users")
class UserController(private val service: UserService) {

    @GetMapping("/{id}")
    fun get(@PathVariable id: Long): UserDto = service.findById(id)
}
```

### 서비스 (Service)
```java
// Java
@Service
public class UserService {
    private final UserRepository repo;
    public UserService(UserRepository repo) { this.repo = repo; }

    public UserDto findById(Long id) {
        User u = repo.findById(id)
            .orElseThrow(() -> new NotFoundException(id));
        return new UserDto(u.getId(), u.getName(), u.getEmail());
    }
}
```
```kotlin
// Kotlin
@Service
class UserService(private val repo: UserRepository) {

    fun findById(id: Long): UserDto {
        val u = repo.findById(id).orElseThrow { NotFoundException(id) }
        return UserDto(u.id, u.name, u.email)
    }
}
```

생성자 주입은 코틀린에서 주 생성자(primary constructor)로 더 간결해지고, 보일러플레이트(게터/세터/생성자)는 data class/프로퍼티로 사라진다. 핵심은 **둘이 같은 Spring API와 같은 빈 컨테이너 위에서 동작**한다는 점이다.

## 참고 출처
- [Introducing Kotlin support in Spring Framework 5.0 (spring.io, 2017-01)](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0/)
- [Spring Framework 5 Kotlin APIs, the functional way (spring.io, 2017-08)](https://spring.io/blog/2017/08/01/spring-framework-5-kotlin-apis-the-functional-way/)
- [Going Reactive with Spring, Coroutines and Kotlin Flow (spring.io, 2019-04)](https://spring.io/blog/2019/04/12/going-reactive-with-spring-coroutines-and-kotlin-flow/)
- [Kotlin support — Spring Framework 5.0 Reference](https://docs.spring.io/spring-framework/docs/5.0.0.RELEASE/spring-framework-reference/kotlin.html)
- [Coroutines :: Spring Framework Reference](https://docs.enterprise.spring.io/spring-framework/reference/languages/kotlin/coroutines.html)
- [All-open compiler plugin (Kotlin docs)](https://kotlinlang.org/docs/all-open-plugin.html)
- [No-arg compiler plugin (Kotlin docs)](https://kotlinlang.org/docs/no-arg-plugin.html)
- [Kotlin — Wikipedia](https://en.wikipedia.org/wiki/Kotlin)
- [Android Announces Support for Kotlin (Android Developers Blog, 2017-05)](https://android-developers.googleblog.com/2017/05/android-announces-support-for-kotlin.html)
- [Spring Boot 2.0 goes GA (spring.io, 2018-03)](https://spring.io/blog/2018/03/01/spring-boot-2-0-goes-ga/)
