# Java 21 (2023년 9월, LTS)

> Java 17 이후 첫 LTS. **가상 스레드(JEP 444)·레코드 패턴(JEP 440)·switch 패턴 매칭(JEP 441)이 모두 정식화**되어, 현대 Java의 동시성과 데이터 중심 프로그래밍 패러다임을 확정한 분기점이 된 릴리스. 총 15개 JEP를 담았다.

## 릴리스 정보
- 정식 출시일: 2023년 9월 19일
- LTS 여부: **예 (Long-Term Support)** — Java 17(2021), Java 25(2025)와 함께 장기 지원 라인
- 포함 JEP 수: **15개** (정식 9개, 프리뷰 5개, 인큐베이터 1개)

## 시대적 배경
Java 21은 2021년 9월 Java 17 이후 **2년 만의 LTS**다. 그 2년간 18·19·20을 거치며 프리뷰·인큐베이터로 검증된 기능들이 한꺼번에 정식으로 쏟아져 나오면서, "여러 해에 걸친 빅 프로젝트(Loom·Amber·Panama)의 수확기"가 되었다.

가장 큰 사건은 **Project Loom의 가상 스레드 정식화**다. 십수 년간 진행되어 온 경량 동시성 작업이 마침내 운영 환경에서 쓸 수 있는 정식 기능이 되었고, 이는 그동안 고동시성을 위해 리액티브 프로그래밍에 의존하던 Java 생태계의 흐름을 되돌렸다. Spring Boot 3.2가 곧바로 가상 스레드를 지원하는 등 프레임워크 채택이 빠르게 이어졌다.

동시에 Project Amber의 **패턴 매칭 3종 세트(레코드 패턴 + switch 패턴 매칭, 그리고 기반이 되는 sealed 타입)**가 정식화되면서, Java는 함수형·데이터 지향 스타일을 일급으로 표현할 수 있게 되었다. **Sequenced Collections**라는 오래 기다려 온 컬렉션 API 보강도 정식 포함됐다.

대부분의 기업이 LTS만 운영에 올린다는 점에서, Java 21은 "실무 Java의 다음 기준선"이 된 릴리스다.

---

## 주요 추가 기능 (정식)

### 가상 스레드 (JEP 444, Final/정식) ⭐⭐⭐
Java 19(JEP 425)·20(JEP 436)의 두 차례 프리뷰를 거쳐 **정식 기능**이 되었다. Java 21을 정의하는 단 하나의 기능을 꼽으라면 단연 이것이다.

**무엇인가**
- 가상 스레드는 **JVM이 관리하는 경량 스레드**다. OS 스레드와 1:1로 묶이지 않고, 다수의 가상 스레드가 소수의 플랫폼(OS) 스레드(**캐리어 스레드**) 위에서 M:N으로 다중화된다.
- 가상 스레드가 블로킹 작업(I/O, `sleep`, 락 대기 등)에 진입하면 JVM이 그 스레드를 캐리어에서 **언마운트**하고, 캐리어는 다른 가상 스레드를 실행한다. 블로킹이 끝나면 다시 마운트되어 이어서 실행된다.
- 결과적으로 **수십만~수백만 개의 가상 스레드**를 생성해도 OS 스레드는 적게 유지된다. 생성·전환 비용이 매우 저렴하다.

**왜 중요한가**
- 그동안 높은 처리량을 위해서는 스레드 풀을 아껴 쓰고 비동기·논블로킹(리액티브) 코드를 작성해야 했다. 이는 코드 복잡도·디버깅 난이도·스택트레이스 단절이라는 큰 비용을 동반했다.
- 가상 스레드는 **"요청 하나당 스레드 하나(thread-per-request)"**라는 단순하고 직관적인 모델을 유지하면서도 리액티브 수준의 확장성을 제공한다. 즉, **동기·블로킹 스타일의 읽기 쉬운 코드로 고동시성**을 달성한다.

**20 대비 정식화에서의 변경**
- 2차 프리뷰(JEP 436)와 비교한 가장 큰 변화는 가상 스레드가 **`ThreadLocal`을 완전히 지원**하게 된 것이다(thread-local 사용을 opt-out 하는 옵션 제거). 호환성을 위해 thread-local을 신뢰성 있게 동작시키는 방향으로 정리되었다.

**사용 예시**
```java
// 정식 기능 — --enable-preview 불필요

// 1) 단일 가상 스레드
Thread.startVirtualThread(() ->
        System.out.println("Running in " + Thread.currentThread()));

// 2) 빌더 API
Thread vt = Thread.ofVirtual()
        .name("worker-", 0)
        .start(() -> doWork());
vt.join();

// 3) 요청당 스레드 — 가상 스레드 ExecutorService
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 1_000_000).forEach(i ->
        executor.submit(() -> {
            // 블로킹 I/O를 마음껏 호출 — OS 스레드를 점유하지 않는다
            Thread.sleep(Duration.ofSeconds(1));
            return fetchFromDatabase(i);
        }));
} // close()가 모든 작업 완료까지 대기
```

**주의할 점 (실무)**
- 가상 스레드는 **풀링하지 않는다**. "필요할 때마다 새로 생성"이 올바른 사용법이다.
- `synchronized` 블록 안에서 블로킹하면 가상 스레드가 캐리어에 **고정(pinning)**되어 확장성이 떨어질 수 있다(이 한계는 이후 JDK 24의 JEP 491에서 해소된다). 21에서는 `ReentrantLock` 사용이 권장됐다.
- CPU 바운드 작업에는 이점이 적다. 가상 스레드의 강점은 **블로킹 I/O가 많은 워크로드**에서 나온다.

### 레코드 패턴 (JEP 440, Final/정식) ⭐
Java 19(JEP 405)·20(JEP 432)의 프리뷰를 거쳐 **정식**이 되었다.
- 패턴 매칭에서 **레코드를 분해(deconstruct)**하여 컴포넌트를 곧바로 바인딩한다. `instanceof`와 `switch` 모두에서 사용 가능하다.
- **중첩**이 가능해, 깊은 객체 구조를 한 줄로 풀어낸다. sealed 타입과 결합하면 컴파일러가 전체성을 검증한다.

```java
record Point(int x, int y) {}
record Line(Point from, Point to) {}

// instanceof + 중첩 레코드 패턴
static void print(Object obj) {
    if (obj instanceof Line(Point(var x1, var y1), Point(var x2, var y2))) {
        System.out.printf("(%d,%d) -> (%d,%d)%n", x1, y1, x2, y2);
    }
}

// switch + 레코드 패턴
sealed interface Shape permits Circle, Rectangle {}
record Circle(double r) implements Shape {}
record Rectangle(double w, double h) implements Shape {}

static double area(Shape s) {
    return switch (s) {
        case Circle(double r)              -> Math.PI * r * r;
        case Rectangle(double w, double h) -> w * h;
    }; // sealed → 컴파일러가 전체성 보장, default 불필요
}
```

### switch를 위한 패턴 매칭 (JEP 441, Final/정식) ⭐
Java 17부터 18·19·20까지 **네 차례의 프리뷰**를 거쳐 마침내 정식이 되었다.
- `switch`의 `case` 라벨에 **타입 패턴**을 쓸 수 있고, `when` 절로 **가드 조건**을 붙일 수 있다.
- `null`을 `case null`로 명시적으로 다룰 수 있다.
- 패턴 switch는 **모든 입력값을 빠짐없이 처리(전체성)**해야 한다. sealed 타입·enum과 결합하면 컴파일 타임에 누락이 잡힌다.

```java
sealed interface Animal permits Dog, Cat, Bird {}
record Dog(String name) implements Animal {}
record Cat(String name) implements Animal {}
record Bird(boolean canFly) implements Animal {}

static String sound(Animal a) {
    return switch (a) {
        case Dog d            -> d.name() + ": 멍멍";
        case Cat c            -> c.name() + ": 야옹";
        case Bird b when b.canFly() -> "짹짹 (날 수 있음)";
        case Bird b           -> "짹짹 (못 남)";
    }; // sealed → default 불필요
}

// null 통합 처리 + 타입 패턴
static String describe(Object obj) {
    return switch (obj) {
        case null       -> "널 값";
        case Integer i  -> "정수 " + i;
        case String s   -> "문자열 " + s;
        default         -> "기타 " + obj;
    };
}
```

### Sequenced Collections (JEP 431, Final/정식) ⭐
- **순서가 있는 컬렉션을 위한 통일된 인터페이스**를 새로 도입했다. 그동안 "첫/마지막 원소 접근"이나 "역순 순회"가 컬렉션 종류마다 제각각(예: `List.get(0)` vs `Deque.getFirst()` vs `LinkedHashSet`은 방법 없음)이었던 문제를 해결한다.
- 새 인터페이스: `SequencedCollection`, `SequencedSet`, `SequencedMap`.
- 공통 메서드: `getFirst()`, `getLast()`, `addFirst()`, `addLast()`, `removeFirst()`, `removeLast()`, 그리고 **역순 뷰** `reversed()`.

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3));
list.getFirst();   // 1  (기존 list.get(0))
list.getLast();    // 3  (기존 list.get(list.size()-1))
list.addFirst(0);  // [0, 1, 2, 3]
list.reversed();   // [3, 2, 1, 0] (뷰)

LinkedHashSet<String> set = new LinkedHashSet<>(List.of("a", "b", "c"));
set.getFirst();    // "a"  — 이전엔 깔끔한 방법이 없었다
set.getLast();     // "c"

SequencedMap<String,Integer> map = new LinkedHashMap<>();
map.putFirst("x", 1);
map.putLast("y", 2);
map.firstEntry();  // x=1
```

### Generational ZGC (JEP 439, Final/정식)
- 저지연 가비지 컬렉터 **ZGC에 세대(generational) 모드**를 추가했다. 객체를 젊은 세대와 오래된 세대로 나눠 관리한다.
- "대부분의 객체는 금방 죽는다"는 약한 세대 가설을 활용해, 젊은 세대를 더 자주·저렴하게 수집한다. 결과적으로 **할당 stall 감소, 더 적은 힙 여유로 동일 성능, CPU 오버헤드 감소**.
- 21에서는 `-XX:+UseZGC -XX:+ZGenerational`로 활성화(비세대 ZGC가 기본). 이후 세대 ZGC가 기본이 되고 비세대 모드는 폐기 수순을 밟는다.

### Key Encapsulation Mechanism API (JEP 452, Final/정식)
- **키 캡슐화 메커니즘(KEM)**을 위한 표준 API(`javax.crypto.KEM`)를 도입했다. KEM은 공개키 암호로 대칭키를 안전하게 전달하는 현대적 기법이다.
- **포스트 양자 암호(PQC)** 알고리즘 다수가 KEM 형태라는 점에서, 양자 내성 암호 시대를 대비한 기반 작업이다.

---

## 미리보기·인큐베이터 기능

### 문자열 템플릿 (JEP 430, Preview/1차 프리뷰)
- 문자열에 표현식을 끼워 넣되 **인젝션을 방지하고 검증 가능한** 보간 메커니즘. `STR`, `FMT`, `RAW` 등의 템플릿 프로세서를 제공한다.
- 백틱이 아니라 `\{...}` 구문을 쓴다.

```java
// --enable-preview 필요
String name = "Java";
int version = 21;
String msg = STR."Hello \{name} \{version}!"; // "Hello Java 21!"
```
> 주의: 문자열 템플릿은 이후 설계 재검토로 **22에서 2차 프리뷰**를 거친 뒤 제거(철회)되었다. Java 21 시점에는 프리뷰였다.

### 미명명 패턴과 변수 (JEP 443, Preview/1차 프리뷰)
- 쓰지 않는 패턴 컴포넌트나 변수를 **`_`(언더스코어)**로 표기한다. 의도적으로 무시함을 명확히 한다.

```java
// --enable-preview 필요
if (obj instanceof Point(int x, _)) { /* y는 무시 */ }

try { ... } catch (Exception _) { log("실패"); } // 예외 변수 미사용 명시

for (var _ : items) count++;
```

### 미명명 클래스와 인스턴스 main 메서드 (JEP 445, Preview/1차 프리뷰)
- 입문자를 위해 **보일러플레이트 없는 `main`**을 허용한다. `public static void`나 `String[] args`, 클래스 선언 없이도 프로그램을 작성할 수 있다.

```java
// --enable-preview 필요 — 파일 전체가 이게 전부일 수 있다
void main() {
    System.out.println("Hello, World!");
}
```

### Scoped Values (JEP 446, Preview/1차 프리뷰)
- Java 20(JEP 429) 인큐베이터에서 **프리뷰**로 승격. 가상 스레드 및 자식 스레드로 불변 데이터를 안전·경량으로 공유한다. `ThreadLocal`의 대안.

```java
// --enable-preview 필요
final static ScopedValue<User> USER = ScopedValue.newInstance();

ScopedValue.where(USER, currentUser).run(() -> handle()); // 이 범위에서만 USER.get() 유효
```

### 구조적 동시성 (JEP 453, Preview/1차 프리뷰)
- Java 19·20의 인큐베이터를 거쳐 **프리뷰**로 승격(패키지가 `java.util.concurrent`로 이동). 가상 스레드와 결합해 다중 작업을 하나의 단위로 다룬다.

```java
// --enable-preview 필요
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var user  = scope.fork(() -> findUser(id));
    var order = scope.fork(() -> fetchOrder(id));
    scope.join().throwIfFailed();          // 모두 대기 + 실패 시 전파
    return new Response(user.get(), order.get());
}
```

### Foreign Function & Memory API (JEP 442, Third Preview/3차 프리뷰)
- 19·20의 프리뷰에 이은 **세 번째 프리뷰**. JNI 없이 네이티브 함수 호출과 힙 밖 메모리 접근을 다룬다. 정식화는 Java 22(JEP 454).

### Vector API (JEP 448, Sixth Incubator/6차 인큐베이터)
- SIMD 벡터 연산 API의 **여섯 번째 인큐베이터**. FFM API와 보조를 맞춰 계속 인큐베이터로 유지되었다.

---

## 그 외 변경
- **JEP 449: Windows 32-bit x86 포트 폐기 예고** — 향후 제거를 위한 deprecation. 가상 스레드를 32비트에서 fallback 구현해야 하는 부담 등이 배경이다.
- **JEP 451: 에이전트의 동적 로딩 제한 준비** — 실행 중인 JVM에 자바 에이전트를 동적으로 붙일 때 경고를 내보내, 향후 기본 비허용으로 전환할 준비를 한다. 무결성·보안 강화 흐름의 일부.
- 다수의 라이브러리 API 추가, 성능·보안 개선, deprecation 정리.

## 영향과 의의
Java 21은 **Java 17 이후 가장 중요한 LTS**, 나아가 Java 8 이래 가장 큰 패러다임 전환을 담은 릴리스로 평가된다.

- **가상 스레드(JEP 444)**: Java 동시성 모델을 근본부터 바꿨다. 리액티브 프로그래밍의 복잡성 없이 고동시성을 달성하는 길을 열었고, Spring Boot 3.2, Helidon, Quarkus, Micronaut 등이 빠르게 지원에 나섰다. "thread-per-request의 부활"이라 불린다.
- **패턴 매칭 완성(JEP 440 + 441 + sealed)**: 레코드·sealed·패턴 매칭이 한데 모여, Java가 대수적 데이터 타입(ADT)과 데이터 지향 프로그래밍을 일급으로 표현할 수 있게 되었다. `switch` 한 표현식으로 타입 분기와 분해, 전체성 검사를 동시에 수행한다.
- **Sequenced Collections(JEP 431)**: 25년 묵은 컬렉션 프레임워크의 빈틈(통일된 순서 접근)을 메웠다.
- **Generational ZGC**: 대용량 힙에서 저지연을 유지하는 GC 기술이 한 단계 성숙했다.

LTS라는 점에서 이 모든 기능이 기업 운영 환경의 "기본값"이 되었다. 많은 조직이 Java 8/11/17에서 21로의 점프를 계획하면서, Java 21은 사실상 **현대 Java의 새 기준선**으로 자리 잡았다.

## 참고 출처
- [JEP 444: Virtual Threads](https://openjdk.org/jeps/444)
- [JEP 440: Record Patterns](https://openjdk.org/jeps/440)
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441)
- [JEP 431: Sequenced Collections](https://openjdk.org/jeps/431)
- [JEP 439: Generational ZGC](https://openjdk.org/jeps/439)
- [JEP 452: Key Encapsulation Mechanism API](https://openjdk.org/jeps/452)
- [JEP 430: String Templates (Preview)](https://openjdk.org/jeps/430)
- [JEP 443: Unnamed Patterns and Variables (Preview)](https://openjdk.org/jeps/443)
- [JEP 445: Unnamed Classes and Instance Main Methods (Preview)](https://openjdk.org/jeps/445)
- [JEP 446: Scoped Values (Preview)](https://openjdk.org/jeps/446)
- [JEP 453: Structured Concurrency (Preview)](https://openjdk.org/jeps/453)
- [JEP 442: Foreign Function & Memory API (Third Preview)](https://openjdk.org/jeps/442)
- [OpenJDK JDK 21 프로젝트 페이지](https://openjdk.org/projects/jdk/21/)
- [InfoQ: Java 21, the Next LTS Release, Delivers Virtual Threads, Record Patterns and Pattern Matching](https://www.infoq.com/news/2023/09/java21-released/)
- [Oracle Java Magazine: Java 21 is here](https://blogs.oracle.com/javamagazine/java-21-now-available/)
