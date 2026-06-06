# Java 19 (2022년 9월)

> Project Loom의 **가상 스레드(Virtual Threads)**가 마침내 프리뷰로 등장한, 동시성 역사에서 가장 중요한 전환점 중 하나가 된 릴리스. 구조적 동시성(인큐베이터), 레코드 패턴(프리뷰), Foreign Function & Memory API의 프리뷰 승격도 함께 이루어졌다.

## 릴리스 정보
- 정식 출시일: 2022년 9월 20일
- LTS 여부: 아니오 (단기 지원)
- 포함 JEP 수: 7개

## 시대적 배경
Java 19는 **세 거대 프로젝트(Loom, Panama, Amber)의 결실이 동시에 가시화된** 릴리스다. 특히 십수 년간 진행되어 온 Project Loom의 가상 스레드가 처음으로 프리뷰로 공개되면서, "스레드 하나에 OS 스레드 하나"라는 Java 동시성의 근본 제약을 깨뜨릴 길이 열렸다.

당시 백엔드 생태계는 높은 동시성을 다루기 위해 리액티브 프로그래밍(WebFlux, RxJava, Reactor)에 크게 의존하고 있었다. 그러나 리액티브 스타일은 코드가 복잡하고 디버깅·스택트레이스가 어려웠다. 가상 스레드는 **"익숙한 동기·블로킹 코드를 그대로 쓰면서 리액티브 수준의 확장성"**을 약속하며 이 흐름에 정면으로 도전했다.

## 주요 추가 기능

### 가상 스레드 (JEP 425, Preview/1차 프리뷰) ⭐
- **OS 스레드에 1:1로 묶이지 않는 경량 스레드**. JVM이 다수의 가상 스레드를 소수의 플랫폼(OS) 스레드(캐리어 스레드) 위에 다중화(M:N)한다.
- 가상 스레드가 블로킹 I/O를 만나면, JVM이 해당 가상 스레드를 캐리어에서 **언마운트(unmount)**하고 다른 가상 스레드를 올린다. 따라서 수십만~수백만 개의 가상 스레드를 만들어도 OS 스레드는 적게 유지된다.
- 핵심 가치: 동기·블로킹 스타일의 단순한 코드를 그대로 쓰면서도 높은 처리량을 얻는다. 리액티브의 복잡성 없이 확장성을 확보한다.
- `Thread`/`ExecutorService` 등 기존 API와 호환된다. 가상 스레드는 항상 데몬이며 우선순위 설정이 무시되는 등 일부 의미 차이가 있다.

```java
// --enable-preview 필요 (JDK 19)
// 1) 가상 스레드 하나 실행
Thread vt = Thread.ofVirtual().start(() ->
        System.out.println("Hello from " + Thread.currentThread()));
vt.join();

// 2) 가상 스레드 기반 ExecutorService — 요청마다 스레드 하나
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i ->
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1)); // 블로킹해도 OS 스레드를 점유하지 않음
            return i;
        }));
} // close()가 모든 작업 완료를 대기
```

> 가상 스레드는 19·20에서 프리뷰를 거쳐 **Java 21(JEP 444)에서 정식**이 된다. 이것이 Java 21을 "현대 Java의 분기점"으로 만든 핵심 기능이다.

### 구조적 동시성 (JEP 428, Incubator/1차 인큐베이터)
- 여러 스레드에서 실행되는 연관 작업들을 **하나의 작업 단위로 묶어** 다루는 API다(`jdk.incubator.concurrent`의 `StructuredTaskScope`).
- 부모-자식 작업의 생명주기가 코드 블록 구조와 일치하도록 강제하여, 오류 전파·취소·관찰성을 단순화한다. "한 작업이 실패하면 형제 작업을 자동 취소", "모두 끝날 때까지 대기" 같은 패턴을 안전하게 표현한다.
- 가상 스레드와 결합할 때 진가를 발휘한다 (작업마다 가상 스레드를 자유롭게 생성 가능).

```java
// 인큐베이터 API (JDK 19)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user  = scope.fork(() -> findUser());
    Future<Integer> order = scope.fork(() -> fetchOrder());

    scope.join();            // 두 작업 완료 대기
    scope.throwIfFailed();   // 하나라도 실패하면 예외 전파

    return new Response(user.resultNow(), order.resultNow());
}
```

### 레코드 패턴 (JEP 405, Preview/1차 프리뷰)
- 패턴 매칭에서 **레코드 값을 분해(deconstruct)**할 수 있게 한다. `instanceof`와 `switch` 모두에서 사용 가능하다.
- **중첩(nesting)**이 가능해, 복잡한 객체 구조를 한 번에 풀어낼 수 있다.
- Java 21(JEP 440)에서 정식화된다.

```java
record Point(int x, int y) {}
record Line(Point from, Point to) {}

static String describe(Object obj) {
    // 중첩 레코드 패턴으로 한 번에 분해
    if (obj instanceof Line(Point(var x1, var y1), Point(var x2, var y2))) {
        return "(%d,%d) -> (%d,%d)".formatted(x1, y1, x2, y2);
    }
    return "unknown";
}
```

### Foreign Function & Memory API (JEP 424, Preview/1차 프리뷰)
- 18까지 인큐베이터(`jdk.incubator.foreign`)였던 API가 **정식 패키지 `java.lang.foreign`의 프리뷰**로 승격되었다.
- JNI 없이 네이티브 함수를 호출하고(다운콜/업콜), JVM 힙 밖 메모리를 안전하게 다룬다. `Linker`, `MethodHandle`, `MemorySegment`, `MemorySession` 등을 제공한다.

### Pattern Matching for switch (JEP 427, Third Preview/3차 프리뷰)
- 18의 2차 프리뷰(JEP 420)에 이은 **3차 프리뷰**. 가장 눈에 띄는 변화는 가드 조건을 `&&`가 아니라 **`when` 키워드**로 표현하게 바뀐 점이다.
- `null` 케이스 처리와 패턴 지배 규칙도 더 정련되었다. 정식화는 Java 21(JEP 441).

```java
// 3차 프리뷰: when 절 도입
static String classify(Object obj) {
    return switch (obj) {
        case Integer i when i > 100 -> "큰 수";
        case Integer i              -> "정수 " + i;
        case String s               -> "문자열 " + s;
        default                     -> "기타";
    };
}
```

### Vector API (JEP 426, Fourth Incubator/4차 인큐베이터)
- SIMD 벡터 연산 API의 **네 번째 인큐베이터**. Project Panama의 다른 기능들과 보조를 맞춰 계속 다듬어졌다.

## 그 외 변경
- **JEP 422: Linux/RISC-V 포트** — JDK를 RISC-V(RV64GV) 아키텍처의 Linux로 포팅했다. 벡터 명령을 포함한 범용 64비트 ISA 지원으로, 오픈 하드웨어 생태계로의 확장을 의미한다.

## 영향과 의의
Java 19는 **"Loom의 시대"를 연 릴리스**로 기억된다.
- 가상 스레드(JEP 425)는 Java가 고동시성 워크로드를 다루는 방식을 근본적으로 바꿀 잠재력을 처음으로 손에 잡히게 보여줬다. 이후 21에서 정식화되며 Spring Boot 등 주요 프레임워크가 빠르게 채택했다.
- 구조적 동시성과 레코드 패턴은 각각 "동시성 코드의 구조화"와 "데이터 중심 프로그래밍"이라는 방향을 명확히 했다.
- FFM API의 프리뷰 승격은 JNI 시대의 종말을 예고했다.

이 모든 것이 아직 프리뷰/인큐베이터였기에 운영 환경에 바로 쓰긴 어려웠지만, Java 19는 Java 21 LTS가 담을 청사진을 거의 그대로 드러낸 릴리스였다.

## 참고 출처
- [JEP 425: Virtual Threads (Preview)](https://openjdk.org/jeps/425)
- [JEP 428: Structured Concurrency (Incubator)](https://openjdk.org/jeps/428)
- [JEP 405: Record Patterns (Preview)](https://openjdk.org/jeps/405)
- [JEP 424: Foreign Function & Memory API (Preview)](https://openjdk.org/jeps/424)
- [JEP 427: Pattern Matching for switch (Third Preview)](https://openjdk.org/jeps/427)
- [JEP 426: Vector API (Fourth Incubator)](https://openjdk.org/jeps/426)
- [JEP 422: Linux/RISC-V Port](https://openjdk.org/jeps/422)
- [OpenJDK JDK 19 프로젝트 페이지](https://openjdk.org/projects/jdk/19/)
- [InfoQ: Java 19 Delivers Features for Projects Loom, Panama and Amber](https://www.infoq.com/news/2022/09/java19-released/)
