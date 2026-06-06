# Java 22 (2024.03)

> 외부 함수 & 메모리 API와 미명명 변수/패턴을 정식화하고, 스트림 Gatherers·생성자 본문 유연화 등 차세대 기능을 대거 프리뷰로 선보인 비-LTS 릴리스.

## 릴리스 정보
- 정식 출시일: 2024년 3월 19일
- LTS 여부: 아니오 (단기 지원, 6개월 주기 / Java 23으로 대체)
- 포함 JEP 목록 (총 12개):
  - JEP 423: Region Pinning for G1 (정식)
  - JEP 447: Statements before super(...) (프리뷰)
  - JEP 454: Foreign Function & Memory API (정식)
  - JEP 456: Unnamed Variables & Patterns (정식)
  - JEP 457: Class-File API (프리뷰)
  - JEP 458: Launch Multi-File Source-Code Programs (정식)
  - JEP 459: String Templates (2차 프리뷰)
  - JEP 460: Vector API (7차 인큐베이터)
  - JEP 461: Stream Gatherers (프리뷰)
  - JEP 462: Structured Concurrency (2차 프리뷰)
  - JEP 463: Implicitly Declared Classes and Instance Main Methods (2차 프리뷰)
  - JEP 464: Scoped Values (2차 프리뷰)

## 시대적 배경

Java 21(2023.09, LTS) 직후의 첫 비-LTS 릴리스로, LTS에서 정식화된 기반 위에 차세대 프로젝트들의 결과물을 쏟아낸 버전이다. 특히 **Project Panama**의 핵심 산출물인 외부 함수 & 메모리(FFM) API가 수년간의 인큐베이팅/프리뷰를 거쳐 마침내 정식화되어, JNI를 대체할 안전한 네이티브 상호운용 수단이 표준 라이브러리에 자리 잡았다. 동시에 **Project Loom**(구조적 동시성, 스코프드 값), **Project Amber**(미명명 변수/패턴, 문자열 템플릿, 암시적 클래스) 등 여러 프로젝트가 병렬로 성숙해 가는 모습을 보여주었다.

## 주요 추가 기능

### Foreign Function & Memory API (JEP 454, 정식)
- 자바 코드가 JVM 밖의 네이티브 메모리에 접근하고 네이티브 함수를 직접 호출할 수 있게 하는 API. JDK 14부터 여러 차례 인큐베이터/프리뷰를 거쳐 이 버전에서 정식화되었다. 깨지기 쉽고 위험한 JNI를 대체한다.
- `MemorySegment`, `Arena`, `Linker`, `FunctionDescriptor` 등으로 구성된다.

```java
import java.lang.foreign.*;
import java.lang.invoke.MethodHandle;

try (Arena arena = Arena.ofConfined()) {
    // C 표준 라이브러리의 strlen 함수에 대한 핸들 확보
    Linker linker = Linker.nativeLinker();
    MethodHandle strlen = linker.downcallHandle(
        linker.defaultLookup().find("strlen").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_LONG, ValueLayout.ADDRESS));

    MemorySegment str = arena.allocateUtf8String("Hello FFM");
    long len = (long) strlen.invoke(str);
    System.out.println(len); // 9
}
```

### Unnamed Variables & Patterns (JEP 456, 정식)
- 사용하지 않는 변수나 패턴 컴포넌트를 밑줄(`_`)로 표기해 의도를 명확히 한다. 예외 변수, 람다 파라미터, for 루프 변수, 패턴 매칭 등에서 사용한다.

```java
// 사용하지 않는 예외 변수
try {
    int n = Integer.parseInt(s);
} catch (NumberFormatException _) {
    System.out.println("숫자가 아님");
}

// 레코드 패턴에서 일부 컴포넌트 무시
record Point(int x, int y) {}
if (obj instanceof Point(int x, _)) {
    System.out.println(x);
}
```

### Launch Multi-File Source-Code Programs (JEP 458, 정식)
- 단일 파일 소스 실행(JDK 11)을 확장하여, 컴파일 없이 여러 소스 파일로 구성된 프로그램을 `java` 명령으로 바로 실행할 수 있다.

```bash
# Main.java가 같은 디렉터리의 Helper.java를 참조해도 바로 실행 가능
java Main.java
```

### Stream Gatherers (JEP 461, 프리뷰)
- Stream API에 사용자 정의 **중간 연산**을 만들 수 있는 `Stream.gather(Gatherer)`를 추가한다. 기존에 부족했던 윈도잉, 폴드, 커스텀 변환 등을 표현할 수 있다.

```java
// 고정 크기 윈도우로 묶기 (built-in gatherer)
List<List<Integer>> windows = Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.windowFixed(2))
    .toList(); // [[1, 2], [3, 4], [5]]
```

### Statements before super(...) (JEP 447, 프리뷰)
- 생성자에서 `super(...)`/`this(...)` 호출 **이전에** 인자 검증·준비 등의 문장을 작성할 수 있게 한다(단, 생성 중인 인스턴스 참조는 불가).

```java
class PositiveBigInteger extends BigInteger {
    PositiveBigInteger(long value) {
        if (value <= 0)                         // super() 호출 전 검증
            throw new IllegalArgumentException("양수여야 함");
        super(Long.toString(value));
    }
}
```

### Class-File API (JEP 457, 프리뷰)
- 클래스 파일을 파싱·생성·변환하기 위한 표준 API를 도입한다. 기존에 JDK 내부적으로 의존하던 ASM 같은 외부 라이브러리를 대체할 표준 수단을 제공한다.

### String Templates (JEP 459, 2차 프리뷰)
- 문자열 보간을 안전하게 수행하는 기능의 2차 프리뷰. (참고: 이 기능은 설계상의 문제로 Java 23에서 프리뷰에서도 제거되어 재설계에 들어갔다.)

```java
// 2차 프리뷰 당시 문법 (이후 폐기됨)
String name = "Java";
String msg = STR."Hello \{name}, version \{22}";
```

## 그 외 변경
- **Structured Concurrency (JEP 462, 2차 프리뷰)** / **Scoped Values (JEP 464, 2차 프리뷰)**: Project Loom의 동시성 모델을 계속 다듬는다.
- **Implicitly Declared Classes and Instance Main Methods (JEP 463, 2차 프리뷰)**: 초보자 친화적인 간소화된 `main` 진입점.
- **Vector API (JEP 460, 7차 인큐베이터)**: SIMD 벡터 연산 API 계속 인큐베이팅.
- **Region Pinning for G1 (JEP 423, 정식)**: JNI 임계 영역(critical region) 동안에도 G1 GC가 멈추지 않도록 영역 단위 고정을 도입해 지연을 줄인다.

## 영향과 의의

FFM API의 정식화는 자바 생태계에서 오랜 숙원이던 안전한 네이티브 상호운용의 표준화를 의미하며, 향후 JNI 사용 제한(JDK 24의 JEP 472)으로 이어지는 출발점이 되었다. 또한 Stream Gatherers, 생성자 본문 유연화 등 이후 LTS(Java 25)에서 정식화될 기능들이 본격적으로 모습을 드러낸 릴리스로, "Java 25 LTS로 가는 길목"의 성격이 강하다.

## 참고 출처
- [JDK 22 - OpenJDK 프로젝트 페이지](https://openjdk.org/projects/jdk/22/)
- [Oracle Releases Java 22](https://www.oracle.com/news/announcement/oracle-releases-java-22-2024-03-19/)
- [The Arrival of Java 22! – Inside.java](https://inside.java/2024/03/19/the-arrival-of-java-22/)
- [Consolidated JDK 22 Release Notes](https://www.oracle.com/java/technologies/javase/22all-relnotes.html)
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
