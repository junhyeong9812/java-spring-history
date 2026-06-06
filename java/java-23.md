# Java 23 (2024.09)

> 마크다운 Javadoc과 세대별 ZGC 기본화를 정식화하고, 원시 타입 패턴 매칭·모듈 임포트 선언을 새로 프리뷰한 비-LTS 릴리스. 문자열 템플릿은 재설계를 위해 전면 보류되었다.

## 릴리스 정보
- 정식 출시일: 2024년 9월 17일
- LTS 여부: 아니오 (단기 지원 / Java 24로 대체)
- 포함 JEP 목록 (총 12개):
  - JEP 455: Primitive Types in Patterns, instanceof, and switch (프리뷰)
  - JEP 466: Class-File API (2차 프리뷰)
  - JEP 467: Markdown Documentation Comments (정식)
  - JEP 469: Vector API (8차 인큐베이터)
  - JEP 471: Deprecate the Memory-Access Methods in sun.misc.Unsafe for Removal
  - JEP 473: Stream Gatherers (2차 프리뷰)
  - JEP 474: ZGC: Generational Mode by Default (정식)
  - JEP 476: Module Import Declarations (프리뷰)
  - JEP 477: Implicitly Declared Classes and Instance Main Methods (3차 프리뷰)
  - JEP 480: Structured Concurrency (3차 프리뷰)
  - JEP 481: Scoped Values (3차 프리뷰)
  - JEP 482: Flexible Constructor Bodies (2차 프리뷰)

## 시대적 배경

Java 22에 이은 두 번째 비-LTS 릴리스로, 1년 뒤 출시될 Java 25 LTS를 향한 기능 성숙의 중간 기점이다. 가장 주목할 사건은 **문자열 템플릿(JEP 459)의 전면 보류**였다. 21·22에서 프리뷰되었으나, 프로세서 중심 설계가 사용자에게 혼란을 주고 조합성이 떨어진다는 피드백에 따라 23에서는 프리뷰에서조차 제거되었다 — 5년 만에 처음으로 정식화에 이르지 못한 프리뷰 기능이 되었다. 한편 문서화(마크다운 Javadoc)와 GC(세대별 ZGC) 영역에서 의미 있는 정식 기능이 들어왔고, `sun.misc.Unsafe`의 메모리 접근 메서드 폐기 예고로 FFM/VarHandle로의 이전 신호를 명확히 했다.

## 주요 추가 기능

### Markdown Documentation Comments (JEP 467, 정식)
- Javadoc 주석을 HTML과 `@`-태그 혼합 대신 마크다운으로 작성할 수 있게 한다. 소스 형태에서 훨씬 읽기 쉽다. `///`로 시작하는 새 주석 형식을 사용한다.

```java
/// 두 수를 더한다.
///
/// - `a` 첫 번째 값
/// - `b` 두 번째 값
///
/// 자세한 내용은 [java.lang.Math]를 참고.
int add(int a, int b) {
    return a + b;
}
```

### ZGC: Generational Mode by Default (JEP 474, 정식)
- Z 가비지 컬렉터의 기본 모드를 **세대별(generational)** 로 전환한다. 대부분의 워크로드에서 비-세대 모드보다 성능이 크게 우수함이 확인되었다. (비-세대 모드는 이후 Java 24의 JEP 490에서 제거된다.)

```bash
# -XX:+UseZGC 만으로 세대별 ZGC가 활성화됨 (-XX:+ZGenerational 불필요)
java -XX:+UseZGC -jar app.jar
```

### Primitive Types in Patterns, instanceof, and switch (JEP 455, 프리뷰)
- 패턴 매칭, `instanceof`, `switch`를 모든 원시 타입에 대해 사용할 수 있게 확장한다.

```java
Object obj = 42;
if (obj instanceof int i) {           // 원시 타입 패턴
    System.out.println(i + 1);
}

switch (x) {                          // 원시 타입에 대한 패턴 switch
    case 0 -> "zero";
    case int i when i > 0 -> "양수";
    default -> "음수";
}
```

### Module Import Declarations (JEP 476, 프리뷰)
- 한 모듈이 export하는 모든 패키지를 한 줄로 임포트한다. 임포트하는 코드가 모듈일 필요는 없다.

```java
import module java.base;   // java.base 모듈의 모든 export 패키지를 임포트

void main() {
    List<String> list = List.of("a", "b");   // java.util 등 별도 import 불필요
}
```

### Stream Gatherers (JEP 473, 2차 프리뷰)
- Java 22에서 프리뷰된 사용자 정의 중간 연산 API를 큰 변경 없이 2차 프리뷰로 이어간다. (Java 24에서 정식화됨.)

### Flexible Constructor Bodies (JEP 482, 2차 프리뷰)
- "Statements before super(...)"(JEP 447)에서 이름이 바뀐 기능의 2차 프리뷰. 명시적 생성자 호출 이전에 문장을 둘 수 있다.

```java
class Range {
    final int lo, hi;
    Range(int lo, int hi) {
        if (lo > hi)                              // this(...) 호출 전 검증
            throw new IllegalArgumentException();
        this.lo = lo;
        this.hi = hi;
    }
}
```

## 그 외 변경
- **Deprecate Memory-Access Methods in sun.misc.Unsafe (JEP 471)**: `sun.misc.Unsafe`의 메모리 접근 메서드를 향후 제거 대상으로 폐기 예고. 대체 수단은 VarHandle(JDK 9)과 FFM API(JDK 22)다.
- **Class-File API (JEP 466, 2차 프리뷰)**: 클래스 파일 처리 표준 API의 2차 프리뷰.
- **Implicitly Declared Classes and Instance Main Methods (JEP 477, 3차 프리뷰)**: 초보자 친화적 진입점 계속 다듬기.
- **Structured Concurrency (JEP 480, 3차 프리뷰)** / **Scoped Values (JEP 481, 3차 프리뷰)**: Project Loom 동시성 기능 지속.
- **Vector API (JEP 469, 8차 인큐베이터)**: SIMD 벡터 API 계속 인큐베이팅.

## 영향과 의의

Java 23은 "정식화 2건 + 다수의 프리뷰 갱신"이라는 전형적인 비-LTS 릴리스 패턴을 보여준다. 마크다운 Javadoc은 라이브러리 문서화 경험을 즉각 개선했고, 세대별 ZGC 기본화는 대용량 힙·저지연 서비스의 운영 기본값을 바꾸었다. 무엇보다 문자열 템플릿의 보류는 "프리뷰는 폐기될 수 있다"는 프리뷰 제도의 취지를 실증한 사례로 남았다.

## 참고 출처
- [JDK 23 - OpenJDK 프로젝트 페이지](https://openjdk.org/projects/jdk/23/)
- [Oracle Releases Java 23](https://www.oracle.com/news/announcement/oracle-releases-java-23-2024-09-17/)
- [Java 23 Delivers Markdown Documentation, ZGC Generational Mode, Deprecate sun.misc.Unsafe - InfoQ](https://www.infoq.com/news/2024/09/java23-released/)
- [Update on String Templates (JEP 459) - OpenJDK amber-spec-experts](https://mail.openjdk.org/pipermail/amber-spec-experts/2024-April/004106.html)
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
