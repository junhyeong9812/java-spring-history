# Java 20 (2023년 3월)

> Java 21 LTS를 6개월 앞두고, 21에 담길 핵심 기능들(가상 스레드, 레코드 패턴, switch 패턴 매칭, FFM API, 구조적 동시성, scoped values)을 마지막으로 다듬은 "징검다리" 릴리스. 정식화된 새 기능은 없고, 전부 다음 단계의 프리뷰·인큐베이터로 진행되었다.

## 릴리스 정보
- 정식 출시일: 2023년 3월 21일
- LTS 여부: 아니오 (단기 지원)
- 포함 JEP 수: 7개

## 시대적 배경
Java 20은 **다음 LTS(Java 21)를 위한 최종 점검 릴리스**의 성격이 강하다. 7개 JEP 모두가 기존 기능의 새로운 프리뷰 라운드이거나 인큐베이터 단계로, 신규 정식 기능은 없었다.

이는 6개월 케이던스 모델의 의도된 작동 방식이다. LTS 직전 비-LTS 릴리스에서 프리뷰 기능을 한 번 더 검증·정련해 피드백을 흡수한 뒤, LTS에서 안정적으로 정식화하는 전략이다. 실제로 Java 20에서 다듬어진 가상 스레드·레코드 패턴·switch 패턴 매칭이 모두 Java 21에서 정식이 된다.

## 주요 추가 기능

### 가상 스레드 (JEP 436, Second Preview/2차 프리뷰) ⭐
- Java 19(JEP 425)의 1차 프리뷰에 이은 **두 번째 프리뷰**. 큰 API 변경 없이 안정화에 집중했다.
- 19 대비 변경: `Thread.Builder`를 통한 스레드 생성, `ThreadLocal` 동작, 스케줄러 관련 세부사항 등이 피드백을 반영해 정련되었다.
- 정식화는 Java 21(JEP 444)에서 이루어진다.

```java
// --enable-preview 필요 (JDK 20)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    var futures = IntStream.range(0, 100_000)
        .mapToObj(i -> executor.submit(() -> {
            Thread.sleep(Duration.ofMillis(100)); // 블로킹 I/O를 흉내
            return i;
        }))
        .toList();
    // 10만 개 작업이 적은 수의 OS 스레드 위에서 실행된다
}
```

### 레코드 패턴 (JEP 432, Second Preview/2차 프리뷰)
- Java 19(JEP 405)에 이은 **두 번째 프리뷰**. 주요 변경: `for` 향상 루프 헤더에서의 레코드 패턴 지원을 제거하는 등 범위를 정리하고, 제네릭 레코드의 타입 추론을 다듬었다.
- 정식화는 Java 21(JEP 440).

```java
// --enable-preview 필요
sealed interface Shape permits Circle, Rectangle {}
record Circle(double r) implements Shape {}
record Rectangle(double w, double h) implements Shape {}

static double area(Shape shape) {
    return switch (shape) {
        case Circle(double r)             -> Math.PI * r * r;
        case Rectangle(double w, double h) -> w * h;
    }; // sealed 덕분에 default 불필요 (전체성 보장)
}
```

### Pattern Matching for switch (JEP 433, Fourth Preview/4차 프리뷰)
- Java 19(JEP 427)의 3차 프리뷰에 이은 **네 번째 프리뷰**. 레코드 패턴(JEP 432)과의 공진화를 위해 한 라운드 더 프리뷰로 유지되었다.
- 전체성(exhaustiveness) 검사 방식과 패턴이 적용되는 타입 처리 등이 정련되었다. 정식화는 Java 21(JEP 441).

```java
// when 가드 + null 케이스 (4차 프리뷰)
static String describe(Object obj) {
    return switch (obj) {
        case null               -> "널";
        case Integer i when i<0 -> "음수";
        case Integer i          -> "정수";
        case String s           -> "문자열(길이 " + s.length() + ")";
        default                 -> "기타";
    };
}
```

### Foreign Function & Memory API (JEP 434, Second Preview/2차 프리뷰)
- Java 19(JEP 424)에 이은 **두 번째 프리뷰**. 의미 있는 API 정리가 이루어졌다.
- `MemorySegment`와 `MemoryAddress` 추상화를 **하나로 통합**, `MemoryLayout` 계층을 sealed로 만들어 패턴 매칭과 잘 어울리게 했고, `MemorySession`을 `Arena`와 `SegmentScope`로 분리했다.
- 정식화는 Java 22(JEP 454)에서 이루어진다(21에서는 3차 프리뷰).

### Scoped Values (JEP 429, Incubator/1차 인큐베이터)
- 스레드(특히 가상 스레드) 내부와 자식 스레드로 **불변 데이터를 안전하게 공유**하는 메커니즘. `ThreadLocal`의 단점을 개선하는 것이 목표다.
- `ThreadLocal`은 가변·상속 비용·생명주기 관리가 문제인데, 가상 스레드가 수천~수만 개 존재하는 환경에서 특히 부담이 크다. Scoped Value는 정해진 동적 범위(바운드 영역) 안에서만 값이 유효하고 불변이라 가볍고 안전하다.

```java
// 인큐베이터 API (JDK 20)
final static ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

ScopedValue.where(CURRENT_USER, loggedInUser)
           .run(() -> handleRequest()); // 이 범위 안에서만 CURRENT_USER.get() 유효

void handleRequest() {
    User u = CURRENT_USER.get(); // 명시적 인자 전달 없이 접근
}
```

### 구조적 동시성 (JEP 437, Second Incubator/2차 인큐베이터)
- Java 19(JEP 428)에 이은 **두 번째 인큐베이터**. API는 거의 그대로 유지하되, `StructuredTaskScope`가 scoped values를 상속·전파하도록 업데이트했다.

### Vector API (JEP 438, Fifth Incubator/5차 인큐베이터)
- SIMD 벡터 연산 API의 **다섯 번째 인큐베이터**. FFM API 변경에 맞춰 보조를 맞췄다.

## 그 외 변경
- Java 20에는 정식화된 신규 언어/플랫폼 기능이 없다. 모든 핵심 변경이 프리뷰·인큐베이터 단계 진전에 집중되었다.
- 다수의 성능·안정성·버그 수정과 JDK 내부 정리 작업이 병행되었다.

## 영향과 의의
Java 20은 단독으로 보면 화려하지 않지만, **Java 21 LTS의 품질을 떠받친 핵심 릴리스**다.
- 가상 스레드·레코드 패턴·switch 패턴 매칭이 여기서 마지막 프리뷰 라운드를 거치며 다듬어졌고, 그 결과 21에서 안심하고 정식화될 수 있었다.
- Scoped Values라는 새 개념이 처음 등장해, 가상 스레드 시대에 맞는 컨텍스트 전달 방식을 제시했다.
- 6개월 케이던스 + 프리뷰 제도가 "큰 기능을 점진적으로, 그러나 LTS에서는 안정적으로" 내보내는 데 어떻게 기여하는지를 보여준 모범 사례다.

## 참고 출처
- [JEP 436: Virtual Threads (Second Preview)](https://openjdk.org/jeps/436)
- [JEP 432: Record Patterns (Second Preview)](https://openjdk.org/jeps/432)
- [JEP 433: Pattern Matching for switch (Fourth Preview)](https://openjdk.org/jeps/433)
- [JEP 434: Foreign Function & Memory API (Second Preview)](https://openjdk.org/jeps/434)
- [JEP 429: Scoped Values (Incubator)](https://openjdk.org/jeps/429)
- [JEP 437: Structured Concurrency (Second Incubator)](https://openjdk.org/jeps/437)
- [JEP 438: Vector API (Fifth Incubator)](https://openjdk.org/jeps/438)
- [OpenJDK JDK 20 프로젝트 페이지](https://openjdk.org/projects/jdk/20/)
- [InfoQ: Java 20 Delivers Features for Projects Amber, Loom and Panama](https://www.infoq.com/news/2023/03/java20-released/)
