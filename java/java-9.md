# Java SE 9 (2017년 9월)

> Project Jigsaw 모듈 시스템으로 플랫폼을 모듈화하고, 6개월 정기 릴리스 시대 직전에 등장한 마지막 "구(舊)모델" 메이저 릴리스.

## 릴리스 정보
- 정식 출시일: 2017년 9월 21일
- 개발 주체: Oracle (OpenJDK / JCP)
- 플랫폼 명세: JSR 379 (Java SE 9 Platform)
- 코드네임: 별도 마케팅 코드네임 없음 (대표 프로젝트명은 "Project Jigsaw")
- LTS 여부: 비(非)LTS. 단기 지원(단기 피처 릴리스) 성격

## 시대적 배경

Java 9의 중심 주제는 **모듈화**다. 모듈 시스템은 원래 Java 7(2011)에 포함될 예정이었으나 Java 8을 거쳐 Java 9까지 미뤄졌다. JDK 자체가 거대한 모놀리식 `rt.jar` 하나로 묶여 있어, 작은 애플리케이션조차 전체 런타임을 끌고 다녀야 했고, public이지만 내부용인 API(`sun.misc.Unsafe` 등)에 외부 코드가 자유롭게 접근하면서 강한 캡슐화가 불가능했다. "classpath hell"(클래스패스 지옥)이라 불리는 암묵적 의존성과 JAR 충돌 문제도 오래된 골칫거리였다.

모듈 시스템(JSR 376)은 진통을 겪었다. 2017년 4~5월의 1차 Public Review 투표가 하위 호환성·벤더 종속 우려 등으로 부결되었고, 수정된 명세가 6월 재투표를 통과한 뒤, 8월 29일~9월 11일 최종 승인 투표를 거쳐 9월 21일 출시되었다.

또한 Java 9는 **릴리스 모델 전환의 분수령**이다. GA(9월 21일)를 앞둔 2017년 9월 6일, Oracle(Mark Reinhold)은 거대 릴리스를 수년에 한 번 내던 방식을 버리고 **6개월마다 정기 피처 릴리스**를 내며 3년마다 LTS를 지정하는 새 모델로의 전환을 발표했다. 이에 따라 Java 9는 옛 모델의 마지막 메이저 릴리스가 되었고, 6개월 뒤 Java 10(2018년 3월)이 새 모델의 첫 릴리스로 나왔다.

## 주요 추가 기능

### Java 플랫폼 모듈 시스템 (Project Jigsaw / JSR 376, JEP 261 등)
- 모듈(module)은 이름이 있고 자기 자신을 기술하는(self-describing) 코드와 데이터의 묶음이다. 패키지 단위의 가시성을 넘어 모듈 단위의 강한 캡슐화와 명시적 의존성을 제공한다.
- 관련 JEP: **JEP 261 모듈 시스템**, JEP 200 모듈러 JDK, JEP 201 모듈러 소스 코드, JEP 220 모듈러 런타임 이미지, JEP 260 내부 API 캡슐화.
- 모듈은 루트에 `module-info.java`(모듈 디스크립터)를 둔다. `requires`(의존), `exports`(공개 패키지), `opens`(리플렉션 허용), `provides ... with`(서비스 제공), `uses`(서비스 소비)로 관계를 선언한다.

```java
// module-info.java — 모듈 디스크립터
module com.example.app {
    requires java.sql;                 // 다른 모듈에 대한 의존
    requires transitive com.example.api; // 추이 의존 (이 모듈을 쓰는 쪽도 자동 의존)

    exports com.example.app.service;   // 이 패키지만 외부에 공개 (강한 캡슐화)
    opens com.example.app.model;       // 리플렉션 접근 허용 (예: 직렬화/프레임워크)

    uses com.example.spi.PaymentProvider;                 // 서비스 소비
    provides com.example.spi.PaymentProvider
        with com.example.app.KakaoPayProvider;            // 서비스 구현 제공
}
```

```bash
# 모듈 경로로 컴파일/실행
javac -d out --module-source-path src $(find src -name "*.java")
java --module-path out --module com.example.app/com.example.app.Main
```

### jlink — 모듈 기반 커스텀 런타임 이미지 (JEP 282)
- 애플리케이션이 실제로 쓰는 모듈만 골라 최소 크기의 전용 JRE 이미지를 만드는 링커 도구. 모듈 시스템이 있어서 가능해진 기능으로, 컨테이너·임베디드 배포에 유용하다.

```bash
jlink --module-path $JAVA_HOME/jmods:out \
      --add-modules com.example.app \
      --output myapp-runtime
```

### jshell — Java REPL (JEP 222)
- 자바 최초의 공식 Read-Eval-Print Loop. 클래스/`main` 메서드 없이 표현식·문장을 한 줄씩 즉시 실행하고 결과를 확인할 수 있어 학습·프로토타이핑·API 탐색에 적합하다.

```text
$ jshell
jshell> int x = 10
x ==> 10
jshell> IntStream.rangeClosed(1, 5).sum()
$2 ==> 15
jshell> String greet(String n) { return "Hi " + n; }
|  created method greet(String)
jshell> greet("Java")
$4 ==> "Hi Java"
```

### 컬렉션 팩토리 메서드 (JEP 269)
- `List`, `Set`, `Map`에 간결한 불변 컬렉션 생성용 정적 팩토리 메서드 `of(...)` 추가. 반환된 컬렉션은 변경 불가(immutable)다.

```java
// Before (Java 8): 장황한 불변 컬렉션 생성
List<String> list = Collections.unmodifiableList(
        Arrays.asList("a", "b", "c"));
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);
map = Collections.unmodifiableMap(map);
```

```java
// After (Java 9): 팩토리 메서드 한 줄
List<String> list = List.of("a", "b", "c");
Set<String> set   = Set.of("x", "y", "z");
Map<String, Integer> map = Map.of("a", 1, "b", 2);
Map<String, Integer> big = Map.ofEntries(
        Map.entry("a", 1),
        Map.entry("b", 2));
// 모두 불변: list.add(...) 호출 시 UnsupportedOperationException
```

### private 인터페이스 메서드 (Milling Project Coin, JEP 213)
- Java 8에서 도입된 `default`/`static` 인터페이스 메서드 간의 공통 로직을 캡슐화하기 위해, 인터페이스에 `private` 및 `private static` 메서드를 허용.
- JEP 213은 그 외에도 try-with-resources의 effectively-final 변수 사용 허용, 다이아몬드 연산자의 익명 클래스 적용, private 메서드에 `@SafeVarargs` 허용, 식별자 `_`(단독 언더스코어) 금지 등 작은 개선을 포함한다.

```java
interface Logger {
    default void logInfo(String msg)  { log("INFO", msg); }
    default void logError(String msg) { log("ERROR", msg); }

    // private 메서드: default 메서드들의 공통 로직 캡슐화 (외부 비공개)
    private void log(String level, String msg) {
        System.out.println("[" + level + "] " + msg);
    }
}
```

### HTTP/2 클라이언트 (인큐베이터, JEP 110)
- `HttpURLConnection`의 낡은 한계를 대체하는 새 HTTP 클라이언트. HTTP/2와 WebSocket을 지원하며 동기/비동기 요청을 제공한다.
- Java 9에서는 **인큐베이터 모듈**(모듈명 `jdk.incubator.httpclient`, 패키지 `jdk.incubator.http`)로 시범 도입되었고, 이후 Java 11(JEP 321)에서 `java.net.http`로 정식 표준화되었다.

```java
// Java 9 인큐베이터 API (이후 java.net.http로 표준화됨)
HttpClient client = HttpClient.newHttpClient();
HttpRequest req = HttpRequest.newBuilder()
        .uri(URI.create("https://example.com"))
        .GET()
        .build();
HttpResponse<String> res =
        client.send(req, HttpResponse.BodyHandler.asString());
System.out.println(res.statusCode());
```

### 프로세스 API 개선 (JEP 102)
- 운영체제 프로세스를 다루는 API 강화. `ProcessHandle`로 PID 조회, 자식/후손 프로세스 열거, 생존 여부 확인, 종료 시 콜백 등을 지원.

```java
ProcessHandle current = ProcessHandle.current();
System.out.println("PID: " + current.pid());
current.info().command().ifPresent(System.out::println);
ProcessHandle.allProcesses()
        .filter(p -> p.info().command().isPresent())
        .forEach(p -> System.out.println(p.pid()));
```

### G1을 기본 가비지 컬렉터로 (JEP 248)
- 32비트/64비트 서버 구성에서 **G1(Garbage-First) GC를 기본 GC로** 채택. 기존 기본값이던 Parallel GC를 대체하여, 큰 힙에서도 예측 가능한 짧은 정지 시간(low-pause)을 우선하는 방향으로 전환했다.

### Stream / Optional API 개선
- **Stream**: `takeWhile`, `dropWhile`(조건 기반 자르기), 3인자 `iterate(seed, hasNext, next)`(종료 조건을 받는 오버로드), `ofNullable`(null이면 빈 스트림) 추가.
- **Optional**: `ifPresentOrElse`(값 유무에 따라 분기), `or`(빈 경우 대체 Optional), `stream`(Optional → Stream 변환) 추가.

```java
Stream.iterate(1, n -> n <= 100, n -> n * 2)  // 종료 조건 포함 iterate
      .forEach(System.out::println);

Stream.of(1, 2, 3, 4, 5)
      .takeWhile(n -> n < 4)   // [1, 2, 3]
      .forEach(System.out::println);

Optional.ofNullable(user)
        .ifPresentOrElse(
            u -> System.out.println(u.getName()),
            () -> System.out.println("no user"));
```

## 그 외 변경 / API 추가
- **JEP 266 — More Concurrency Updates**: Reactive Streams 표준을 구현한 `java.util.concurrent.Flow`(Publisher/Subscriber/Processor) API 도입, `CompletableFuture` 보강(`completeOnTimeout`, `orTimeout` 등).
- **JEP 193 — Variable Handles (VarHandles)**: `sun.misc.Unsafe` 대체를 위한 안전한 저수준 메모리/필드 원자 연산 API.
- **JEP 238 — Multi-Release JAR Files**: 하나의 JAR에 자바 버전별 클래스를 함께 담아 런타임 버전에 맞는 구현을 선택.
- **JEP 254 — Compact Strings**: `String` 내부 저장을 `char[]`(UTF-16)에서 `byte[]` + 인코딩 플래그(Latin-1/UTF-16)로 변경하여 Latin-1 문자열의 메모리 사용량을 절감.
- **JEP 295 — Ahead-of-Time Compilation**: `jaotc`를 통한 실험적 AOT 컴파일.
- **JEP 158/271 — Unified JVM/GC Logging**: 로그 출력을 단일 프레임워크(`-Xlog`)로 통합.
- **JEP 277 — Enhanced Deprecation**: `@Deprecated`에 `since`, `forRemoval` 속성 추가.
- **JEP 213 부수**: `@SafeVarargs`를 private 메서드에도 적용 등.
- **JEP 222 외 도구**: `jdeps` 강화, `jlink`, `jmod` 등 모듈 관련 도구군 추가.

## 영향과 의의

Java 9의 가장 큰 유산은 두 가지다. 첫째, **모듈 시스템**은 플랫폼을 모듈화하여 보안(내부 API 캡슐화)·확장성·배포 최적화(jlink)의 토대를 놓았다. 다만 강한 캡슐화로 인해 리플렉션에 의존하던 다수의 라이브러리·프레임워크가 깨졌고, 애플리케이션 레벨에서의 모듈 도입은 기대만큼 빠르게 확산되지 않았다. 그럼에도 JDK 자체의 모듈화와 내부 API 차단은 이후 자바의 보안·진화 전략의 핵심이 되었다.

둘째, Java 9는 **릴리스 케이던스 전환의 경계선**이다. 이 버전을 끝으로 자바는 6개월 정기 릴리스 + LTS 모델로 이행했고, 그 결과 Java 9는 비LTS로 짧은 지원 기간을 가진 채 빠르게 Java 10·11로 대체되었다. 실무에서는 많은 조직이 모듈 마이그레이션 부담 때문에 Java 8에 머무르거나, Java 9~10을 건너뛰고 곧바로 LTS인 Java 11로 이동하는 패턴을 보였다. 한편 jshell(REPL)과 컬렉션 팩토리 메서드처럼 일상 개발 경험을 직접 개선한 기능들은 버전과 무관하게 빠르게 자리 잡았다.

## 참고 출처
- [Java SE 9 released today (InfoQ, 2017-09-21)](https://www.infoq.com/news/2017/09/Java-9-release-sept-21/)
- [Java Platform Module System (Wikipedia)](https://en.wikipedia.org/wiki/Java_Platform_Module_System)
- [Java version history (Wikipedia)](https://en.wikipedia.org/wiki/Java_version_history)
- [Java 9 Modules (DigitalOcean)](https://www.digitalocean.com/community/tutorials/java-9-modules)
- [The road to Java 9: The current status (InfoWorld)](https://www.infoworld.com/article/2253040/the-road-to-java-9-the-current-status.html)
- [Java 9 Features with Examples (GeeksforGeeks)](https://www.geeksforgeeks.org/java/java-9-features-with-examples/)
