# Java 11 (2018년 9월) — LTS

> 새 릴리스 모델에서 나온 첫 번째 LTS. 표준 HTTP 클라이언트, 단일 파일 실행, Oracle JDK 라이선스 변화로 자바 생태계의 분기점이 된 버전.

## 릴리스 정보
- 정식 출시일: 2018년 9월 25일
- 개발 주체: Oracle (OpenJDK)
- LTS 여부: **예 (Long-Term Support)** — 6개월 케이던스 도입 이후 첫 LTS
- 포함 JEP 수: 17개

## 시대적 배경
Java 10이 6개월 릴리스 케이던스를 증명했다면, Java 11은 그 모델 위에서 **장기 지원(LTS)** 개념을 정립했다. 모든 릴리스가 다음 릴리스까지만(6개월) 지원되면 기업이 따라가기 어렵기 때문에, Oracle은 3년마다 한 번씩(이후 2년으로 단축) LTS 릴리스를 지정해 수년간 보안 패치를 제공하기로 했다. Java 11이 그 첫 LTS로, Java 8을 잇는 사실상의 "기업 표준" 버전이 되었다.

또한 Java 11은 **라이선스 측면의 전환점**이기도 하다. 이 버전부터 Oracle은 자사 빌드(Oracle JDK)를 상용 OTN(Oracle Technology Network) 라이선스로 배포해 운영/상업적 사용에 유료 구독을 요구했고, 무료로 쓰려면 OpenJDK 기반 빌드(Adoptium/Temurin, Amazon Corretto, Azul Zulu, Red Hat 등)를 선택하는 흐름이 자리 잡았다.

## 주요 추가 기능

### 표준 HTTP 클라이언트 (JEP 321)
- 상태: **정식 기능(standard)** — Java 9/10에서 incubator였던 `jdk.incubator.http`가 표준 `java.net.http` 패키지로 승격.
- HTTP/1.1과 HTTP/2, WebSocket을 지원하며 동기/비동기(CompletableFuture) 호출이 모두 가능하다. 오래된 `HttpURLConnection`을 대체한다.

```java
import java.net.http.*;
import java.net.URI;

HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://api.example.com/data"))
        .GET()
        .build();

// 동기 호출
HttpResponse<String> response =
        client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.statusCode());
System.out.println(response.body());

// 비동기 호출
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println);
```

### 람다 파라미터에 대한 지역변수 문법 (JEP 323)
- 상태: 정식 기능
- Java 10의 `var`를 **람다 파라미터**에도 쓸 수 있게 했다. 단독으로는 타입을 명시하는 것과 차이가 없지만, 모든 파라미터에 어노테이션(`@NonNull` 등)이나 수식어를 일관되게 붙일 수 있다는 장점이 있다. 단, 일부 파라미터만 `var`로 쓰거나 명시적 타입과 섞을 수 없다(모두 `var`이거나 모두 명시이거나).

```java
// 모든 파라미터에 어노테이션 적용 가능
BiFunction<Integer, Integer, Integer> add =
        (@NonNull var x, @NonNull var y) -> x + y;
```

### 단일 파일 소스 코드 실행 (JEP 330)
- 상태: 정식 기능
- 컴파일(`javac`) 없이 단일 `.java` 파일을 `java` 런처로 바로 실행할 수 있다. 학습·스크립팅·프로토타이핑에 유용하다.

```bash
# 이제 컴파일 단계 없이 바로 실행 가능
java Hello.java
```

### 새로운 String / Files 메서드 (API)
- 상태: 정식 기능
- `String`에 `strip()`, `stripLeading()`, `stripTrailing()`(유니코드 인지 공백 제거), `isBlank()`, `lines()`(스트림으로 줄 분리), `repeat(int)` 추가.
- `Files.readString(Path)`, `Files.writeString(Path, CharSequence)` 추가로 파일 ↔ 문자열 변환이 한 줄로 가능.

```java
"  hello  ".strip();          // "hello" (trim보다 유니코드 공백을 더 정확히 처리)
"   ".isBlank();              // true
"=".repeat(20);              // "===================="
"a\nb\nc".lines().count();    // 3

import java.nio.file.*;
String content = Files.readString(Path.of("data.txt"));
Files.writeString(Path.of("out.txt"), "Hello, Java 11");
```

## 그 외 변경 / API 추가
- **JEP 318 — Epsilon GC(실험적)**: 메모리를 할당만 하고 회수하지 않는 "No-Op" GC. 성능 테스트, 매우 단명하는 작업, GC 오버헤드 측정용.
- **JEP 333 — ZGC(실험적)**: 대용량 힙에서도 일시정지를 10ms 이하로 유지하는 것을 목표로 하는 확장 가능 저지연 GC(이 시점에는 Linux/x64 실험적).
- **JEP 328 — Flight Recorder(JFR)**: 과거 상용 기능이던 저오버헤드 프로파일링/진단 도구를 오픈소스화해 OpenJDK에 포함.
- **JEP 331 — 저오버헤드 힙 프로파일링**.
- **JEP 320 — Java EE 및 CORBA 모듈 제거**: `java.xml.ws`(JAX-WS), `java.xml.bind`(JAXB), `java.activation`, `java.corba` 등 제거. JavaFX도 JDK에서 분리되어 별도 오픈소스(OpenJFX)로 제공.
- **JEP 332 — TLS 1.3** 지원.
- **JEP 181 — Nest 기반 접근 제어**: 중첩 클래스 간 private 멤버 접근을 컴파일러의 합성 브리지 메서드 없이 JVM 레벨에서 처리.
- **JEP 309 — 동적 클래스 파일 상수(`CONSTANT_Dynamic`)**, **JEP 324 — Curve25519/Curve448 키 교환**, **JEP 329 — ChaCha20/Poly1305 암호 알고리즘**, **JEP 327 — 유니코드 10**.
- **JEP 335 — Nashorn 자바스크립트 엔진 deprecate**, **JEP 336 — Pack200 도구/API deprecate**.

## 영향과 의의
Java 11은 **Java 8 이후 가장 중요한 LTS**로 평가받으며, 많은 기업이 8에서 곧바로 11로 이주했다(중간 9·10은 단기 릴리스라 건너뛴 경우가 많다). 표준 HTTP 클라이언트는 외부 라이브러리(Apache HttpClient, OkHttp) 없이도 현대적 HTTP 통신을 가능하게 했고, 모듈 시스템과 함께 자바를 더 모듈화된 플랫폼으로 만들었다. 동시에 Java EE/CORBA/JavaFX/Nashorn 제거는 "레거시 청산"의 신호였다.

가장 큰 파장은 **라이선스 변화**였다. Oracle JDK가 운영 환경에서 유료가 되면서, 생태계는 OpenJDK 기반 무료 배포판(Eclipse Temurin/Adoptium, Amazon Corretto, Azul Zulu, Microsoft Build of OpenJDK 등)으로 다변화되었다. 오늘날 "어떤 JDK 배포판을 쓸 것인가"라는 질문이 보편화된 출발점이 바로 Java 11이다.

## 참고 출처
- [JDK 11 — OpenJDK Project](https://openjdk.org/projects/jdk/11/)
- [JEP 321: HTTP Client (Standard)](https://openjdk.org/jeps/321)
- [Introducing Java SE 11 — Oracle Blog](https://blogs.oracle.com/java-platform-group/introducing-java-se-11)
- [JDK 11 Release Notes — Oracle](https://www.oracle.com/java/technologies/javase/11-relnote-issues.html)
- [Java version history — Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [90 New Features and APIs in JDK 11 — Azul](https://www.azul.com/blog/90-new-features-and-apis-in-jdk-11/)
