# Java 14 (2020년 3월)

> switch 표현식을 정식 기능으로 확정하고, record·패턴 매칭·텍스트 블록 같은 현대 Java 문법의 씨앗을 preview로 대거 심은 릴리스.

## 릴리스 정보
- 정식 출시일: 2020년 3월 17일
- LTS 여부: 아니오 (단기 지원 릴리스, 다음 버전 출시까지만 지원)

## 시대적 배경

Java는 9 버전부터 6개월 주기 릴리스 모델로 전환했고, Java 14는 그 흐름 속의 한 단계였다. 짧은 주기 덕분에 새 문법을 **preview 기능**으로 먼저 내보내 커뮤니티 피드백을 받고, 다듬어 정식화하는 점진적 전략이 자리를 잡던 시기다.

Java 14는 이 전략의 대표 사례다. switch 표현식은 12·13에서 두 차례 preview를 거쳐 이번에 정식화됐고, record·instanceof 패턴 매칭은 처음 preview로 등장했으며, 텍스트 블록은 두 번째 preview에 들어갔다. "한 번에 큰 변화"가 아니라 "여러 번에 걸친 안전한 진화"라는 모던 Java의 개발 문화가 뚜렷이 드러난다.

## 주요 추가 기능

### switch 표현식 (JEP 361, 정식)

12·13의 두 차례 preview를 거쳐 정식 기능으로 확정됐다. switch를 **문장(statement)** 뿐 아니라 **값을 반환하는 표현식(expression)** 으로 쓸 수 있다. 화살표(`->`) 문법으로 fall-through와 break 누락 버그를 제거하고, `yield`로 블록에서 값을 반환한다.

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
    default -> {
        int len = day.toString().length();
        yield len; // 블록에서 값 반환
    }
};
```

### record (JEP 359, preview)

불변 데이터를 담는 클래스를 한 줄로 선언하는 새 타입. 생성자, 접근자, `equals`/`hashCode`/`toString`이 자동 생성된다. 보일러플레이트 제거가 목표이며, 이 버전에서 처음 preview로 등장했다.

```java
record Point(int x, int y) { }

var p = new Point(3, 4);
System.out.println(p.x());      // 3 (자동 생성된 접근자)
System.out.println(p);          // Point[x=3, y=4]
```

### instanceof 패턴 매칭 (JEP 305, preview)

`instanceof` 검사와 형 변환을 한 번에 처리한다. 검사에 성공하면 바인딩 변수에 캐스팅된 값이 자동으로 들어가, 별도 캐스팅 코드가 사라진다.

```java
if (obj instanceof String s) {
    // s는 이미 String으로 캐스팅됨
    System.out.println(s.length());
}
```

### 유용한 NullPointerException (JEP 358, 정식)

NPE 발생 시 **정확히 어떤 변수/표현식이 null이었는지**를 메시지에 담아 알려준다. 디버깅 시간을 크게 줄여준다. 다만 14에서는 이 상세 메시지가 **기본 비활성화**라 `-XX:+ShowCodeDetailsInExceptionMessages` 옵션을 켜야 하며, 기본 활성화는 JDK 15부터다.

```text
Cannot invoke "String.toLowerCase()" because "<local1>.name" is null
```

### 텍스트 블록 (JEP 368, 2차 preview)

여러 줄 문자열을 `"""`로 감싸 가독성 있게 작성한다. 13에서 1차 preview로 나왔고, 14에서 `\`(줄 이음), `\s`(공백 유지) 이스케이프가 추가된 2차 preview가 됐다.

```java
String html = """
        <html>
            <body>
                <p>Hello</p>
            </body>
        </html>
        """;
```

### JFR 이벤트 스트리밍 (JEP 349, 정식)

JDK Flight Recorder의 데이터를 파일로 덤프하지 않고 **실시간 스트림**으로 소비할 수 있게 했다. 모니터링·관측(observability) 도구가 애플리케이션 성능 데이터를 즉시 받아볼 수 있다.

## 그 외 변경

- **패키징 도구 jpackage (JEP 343, incubator)**: 플랫폼별 네이티브 설치 패키지(msi, dmg, deb 등)를 만드는 도구가 incubator로 도입.
- **Foreign-Memory Access API (JEP 370, incubator)**: 힙 밖 네이티브 메모리에 안전하게 접근하는 API가 incubator로 시작.
- **JEP 345**: G1 GC의 NUMA 인식 메모리 할당.
- **JEP 352**: 비휘발성(NVM) 매핑 ByteBuffer 지원.
- **JEP 364**: ZGC를 macOS로 포팅 (JEP 365는 Windows로 포팅).
- **제거/지원 중단**: CMS 가비지 컬렉터 제거(JEP 363), Pack200 도구·API 제거(JEP 367), Solaris/SPARC 포트 지원 중단(JEP 362), ParallelScavenge+SerialOld GC 조합 지원 중단(JEP 366).

## 영향과 의의

Java 14는 "모던 Java 문법"의 토대를 놓은 분기점이다. 이후 LTS인 Java 17을 화려하게 만든 record·패턴 매칭·텍스트 블록·sealed 같은 기능들의 출발선이 대부분 이 버전(혹은 직전)에 있다. switch 표현식 정식화로 함수형 스타일이 표준 문법에 편입됐고, 유용한 NPE 메시지는 일상적인 개발 경험을 즉시 개선했다. preview 제도를 적극 활용해 큰 언어 변화를 안전하게 굴려가는 모던 Java 개발 모델을 가장 잘 보여준 릴리스이기도 하다.

## 참고 출처
- [OpenJDK: JDK 14](https://openjdk.org/projects/jdk/14/)
- [JEP 361: Switch Expressions](https://openjdk.org/jeps/361)
- [JEP 359: Records (Preview)](https://openjdk.org/jeps/359)
- [JEP 305: Pattern Matching for instanceof (Preview)](https://openjdk.org/jeps/305)
- [JEP 358: Helpful NullPointerExceptions](https://openjdk.org/jeps/358)
- [JEP 368: Text Blocks (Second Preview)](https://openjdk.org/jeps/368)
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
