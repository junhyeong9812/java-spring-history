# Java 13 (2019년 9월)

> 텍스트 블록을 미리 선보이고(preview), switch 표현식에 `yield`를 도입해 언어 현대화를 이어간 버전.

## 릴리스 정보
- 정식 출시일: 2019년 9월 17일
- 개발 주체: Oracle (OpenJDK)
- LTS 여부: 아니오 (단기 지원)
- 포함 JEP 수: 5개

## 시대적 배경
Java 12에서 시작된 preview 기반 언어 진화가 본격적으로 굴러가는 릴리스다. Java 12에서 처음 선보인 switch 표현식이 커뮤니티 피드백을 반영해 2차 preview로 다듬어졌고, 오랫동안 자바 개발자들이 불편해하던 **여러 줄 문자열 리터럴**이 텍스트 블록이라는 형태로 처음 등장했다. "한 릴리스에 큰 기능을 preview로 넣고, 다음 릴리스에서 개선하며, 그다음에 정식화한다"는 리듬이 자리 잡았다.

## 주요 추가 기능

### 텍스트 블록 (JEP 355)
- 상태: **미리보기(Preview)** — `--enable-preview --release 13` 필요. (이후 Java 14에서 2차 preview, Java 15에서 정식화)
- `"""`로 둘러싸는 여러 줄 문자열 리터럴. HTML·JSON·SQL 등 여러 줄 문자열을 escape(`\n`, `\"`)와 연결(`+`) 없이 자연스럽게 작성할 수 있다. 닫는 구분자의 들여쓰기를 기준으로 공통 들여쓰기(incidental whitespace)가 자동 제거된다.

```java
// 기존 방식: escape와 연결로 가독성 저하
String html = "<html>\n" +
              "    <body>\n" +
              "        <p>Hello, Java 13</p>\n" +
              "    </body>\n" +
              "</html>\n";

// Java 13 텍스트 블록 (preview)
String html = """
        <html>
            <body>
                <p>Hello, Java 13</p>
            </body>
        </html>
        """;

String json = """
        {
            "name": "Java",
            "version": 13
        }
        """;

String query = """
        SELECT id, name
        FROM users
        WHERE active = true
        """;
```

### switch 표현식 — 2차 미리보기 (JEP 354)
- 상태: **미리보기(Preview, 2차)** — `--enable-preview --release 13` 필요. (Java 14에서 정식화)
- Java 12의 1차 preview에서 값을 돌려줄 때 쓰던 `break <값>;` 문법을 **`yield` 문**으로 교체했다. 화살표(`->`) 케이스 내부에서 블록을 쓰거나, 전통적 `case ... :` 레이블에서 값을 반환할 때 `yield`를 사용한다.

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    default -> {
        int len = day.toString().length();   // 블록 안에서 계산 후
        yield len;                            // yield로 값 반환
    }
};

// 전통적 콜론 레이블에서도 yield 사용 가능
int result = switch (code) {
    case 1: yield 100;
    case 2: yield 200;
    default: yield 0;
};
```

### ZGC: 미사용 메모리 OS 반환 (JEP 351)
- 상태: 실험적(ZGC 자체가 이 시점에 실험적)
- ZGC가 그동안 사용하지 않는 힙 메모리를 OS에 돌려주지 않던 한계를 개선했다. 일정 시간 유휴 상태의 힙 메모리를 OS로 반환해, 컨테이너·클라우드 환경에서 메모리 효율을 높인다(`-XX:ZUncommitDelay`로 지연 시간 조정).

### 레거시 소켓 API 재구현 (JEP 353)
- 상태: 정식(내부 구현 교체)
- JDK 1.0 시절부터 이어져 유지보수가 어렵던 `java.net.Socket`/`java.net.ServerSocket`의 내부 구현을, 더 단순하고 현대적이며 디버깅하기 쉬운 새 구현(`NioSocketImpl`)으로 교체했다. 이후 Project Loom의 가상 스레드 친화적 I/O를 위한 기반이기도 하다.

## 그 외 변경 / API 추가
- **JEP 350 — 동적 CDS 아카이브(Dynamic CDS Archives)**: 애플리케이션 실행이 끝나는 시점에 로드된 클래스들을 동적으로 아카이브할 수 있게 해, AppCDS 사용성을 크게 개선(사전 클래스 리스트 작성 불필요).
- **API**: `String`에 텍스트 블록 지원용 메서드 `stripIndent()`, `translateEscapes()`, `formatted(Object...)` 추가(preview 연계). `FileSystems.newFileSystem(Path, Map)` 추가. 유니코드 12.1 지원.

## 영향과 의의
Java 13은 단일한 "킬러 기능"보다 **언어를 점진적으로 다듬어 가는 과정**을 잘 보여주는 릴리스다. 텍스트 블록은 자바 개발자들이 수십 년간 바라던 기능으로, preview 단계에서부터 큰 환영을 받았고 Java 15에서 정식화되어 오늘날 SQL/JSON/HTML 작성의 표준이 되었다. switch 표현식의 `yield` 도입은 1차 preview의 어색했던 `break <값>` 문법을 깔끔하게 정리해 Java 14 정식화의 길을 닦았다. ZGC 메모리 반환과 소켓 API 재구현은 클라우드 시대와 이후 가상 스레드(Project Loom)를 향한 준비 작업이라는 의미가 있다.

## 참고 출처
- [JDK 13 — OpenJDK Project](https://openjdk.org/projects/jdk/13/)
- [JEP 355: Text Blocks (Preview)](https://openjdk.org/jeps/355)
- [JEP 354: Switch Expressions (Second Preview)](https://openjdk.org/jeps/354)
- [The arrival of Java 13! — Oracle Blog](https://blogs.oracle.com/java-platform-group/the-arrival-of-java-13)
- [Significant Changes in JDK 13 Release — Oracle Docs](https://docs.oracle.com/en/java/javase/24/migrate/significant-changes-jdk-13.html)
- [Java version history — Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
