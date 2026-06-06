# Java 18 (2022년 3월)

> UTF-8을 기본 문자셋으로 표준화하고, 의존성 없이 쓸 수 있는 간단한 웹서버와 JavaDoc 코드 스니펫 태그를 도입한 비-LTS 릴리스. 동시에 Loom/Panama/Amber 프로젝트의 프리뷰·인큐베이터 기능들이 다음 단계로 진행되었다.

## 릴리스 정보
- 정식 출시일: 2022년 3월 22일
- LTS 여부: 아니오 (단기 지원, 6개월 주기 릴리스)
- 포함 JEP 수: 9개

## 시대적 배경
Java 17(2021년 9월)이 LTS로 출시된 직후의 첫 번째 비-LTS 릴리스다. 6개월 단위 릴리스 케이던스가 안정적으로 자리잡으면서, 거대한 신기능을 한 번에 터뜨리기보다 **프리뷰(preview)와 인큐베이터(incubator) 단계를 거쳐 점진적으로 성숙시키는** 개발 방식이 확립된 시기다.

당시 OpenJDK는 세 개의 큰 프로젝트를 동시에 진행하고 있었다.
- **Project Amber**: 언어 생산성 개선 (패턴 매칭, 레코드 등)
- **Project Panama**: 네이티브 코드/메모리 상호운용 (Foreign Function & Memory API, Vector API)
- **Project Loom**: 경량 동시성 (가상 스레드, 구조적 동시성) — 18에서는 아직 직접적인 기능이 들어오지 않았지만, 다음 릴리스를 향한 준비가 한창이었다.

Java 18은 이 중 Amber와 Panama의 진행, 그리고 플랫폼 기본값 정비(UTF-8)와 개발 편의 기능(웹서버, 스니펫)에 초점을 맞췄다.

## 주요 추가 기능

### UTF-8을 기본 문자셋으로 (JEP 400, Final/정식)
- 표준 Java API들이 사용하는 **기본 문자셋(default charset)을 운영체제·로케일과 무관하게 UTF-8로 고정**했다.
- 기존에는 `file.encoding`이 플랫폼에 따라 달라져(Windows는 흔히 windows-1252/MS949, Linux는 UTF-8), 같은 코드가 환경마다 다르게 동작하는 고질적 버그의 원인이었다.
- 이제 `Charset.defaultCharset()`이 기본적으로 UTF-8을 반환한다. 콘솔 입출력 등은 여전히 `Console`의 별도 인코딩을 따를 수 있다.
- 과거 동작이 필요하면 `-Dfile.encoding=COMPAT`로 되돌릴 수 있다.

```java
// JEP 400의 영향을 받는 것은 default charset에 의존하던 API들이다.
import java.io.*;

// new String(byte[]), FileReader/FileWriter, InputStreamReader, PrintStream 등
// charset을 명시하지 않는 API가 18부터 일관되게 UTF-8을 사용한다.
String text = new String(bytes);          // 18 이전: 플랫폼 charset → 18부터: UTF-8
try (var reader = new FileReader("note.txt")) { /* UTF-8로 읽음 */ }
System.out.println(java.nio.charset.Charset.defaultCharset()); // UTF-8

// 참고: Files.readString/writeString(charset 미지정)은 JEP 400 이전(JDK 11)부터
//       이미 UTF-8 고정이라 이번 변경 대상이 아니다.
```

### 간단한 웹서버 — jwebserver (JEP 408, Final/정식)
- 정적 파일을 서빙하는 **최소 기능 HTTP/1.1 서버**를 JDK에 내장했다. 프로토타이핑, 임시 테스트, 교육 용도가 목적이다.
- 커맨드라인 도구 `jwebserver`와 프로그래밍 API(`com.sun.net.httpserver.SimpleFileServer`)를 모두 제공한다.
- 기본은 현재 디렉토리를 `127.0.0.1:8000`에 읽기 전용으로 서빙한다. **GET과 HEAD 요청만 처리**하며, 그 외 메서드는 405(Method Not Allowed)/501(Not Implemented)로 응답한다. CGI나 동적 콘텐츠는 지원하지 않는다.

```bash
# 현재 디렉토리를 즉시 서빙
$ jwebserver
# 포트/바인드 주소/디렉토리 지정
$ jwebserver -p 9000 -b 0.0.0.0 -d /var/www
```

```java
// 프로그래밍 방식
import com.sun.net.httpserver.SimpleFileServer;
import java.net.InetSocketAddress;
import java.nio.file.Path;

var server = SimpleFileServer.createFileServer(
        new InetSocketAddress(8000),
        Path.of("/var/www"),
        SimpleFileServer.OutputLevel.VERBOSE);
server.start();
```

### JavaDoc 코드 스니펫 — @snippet (JEP 413, Final/정식)
- JavaDoc 표준 Doclet에 **`@snippet` 태그**를 추가하여, API 문서에 들어가는 예제 코드를 더 안전하고 검증 가능하게 작성하도록 했다.
- 기존 `<pre>{@code ...}</pre>` 방식의 한계(컴파일·검증 불가, 하이라이팅·링크 부족)를 보완한다.
- 인라인 스니펫과 외부 파일 참조 스니펫을 모두 지원하며, 영역 강조·정규식 하이라이팅·링크 마크업이 가능하다.

```java
/**
 * 사용 예시:
 * {@snippet :
 *   var list = List.of(1, 2, 3);
 *   list.forEach(System.out::println);
 * }
 */
public void example() { }
```

### Pattern Matching for switch (JEP 420, Second Preview/2차 프리뷰)
- Java 17의 1차 프리뷰(JEP 406)에 이어 **두 번째 프리뷰**다. switch에서 타입 패턴과 가드를 사용할 수 있게 하는 기능을 다듬는 단계다.
- 18에서의 주요 변경: **가드 패턴 표기 정련**(이후 19에서 `when` 키워드로 발전), `null`·전체성(exhaustiveness) 처리 개선, 패턴의 지배 관계(dominance) 검사 강화.
- 정식화는 Java 21(JEP 441)에서 이루어진다.

```java
// --enable-preview 필요
static String describe(Object obj) {
    return switch (obj) {
        case Integer i && i > 0 -> "양의 정수: " + i;
        case Integer i           -> "0 이하 정수: " + i;
        case String s            -> "문자열: " + s;
        case null                -> "널";
        default                  -> "기타";
    };
}
```

### Foreign Function & Memory API (JEP 419, Second Incubator/2차 인큐베이터)
- 네이티브 라이브러리 함수 호출과 JVM 힙 밖 메모리 접근을 안전하게 다루는 API의 **두 번째 인큐베이터**다(`jdk.incubator.foreign`).
- 기존 JNI를 대체하기 위한 Project Panama의 핵심이다. 18에서는 API 사용성과 안전성을 다듬었다.
- 이후 19에서 `java.lang.foreign` 정식 패키지의 **프리뷰**(JEP 424)로 승격된다.

### Vector API (JEP 417, Third Incubator/3차 인큐베이터)
- SIMD 벡터 연산을 하드웨어 벡터 명령으로 안정적으로 컴파일하는 API의 **세 번째 인큐베이터**다(`jdk.incubator.vector`).
- 머신러닝·암호·금융 등 수치 연산 가속이 목적이다. 18에서는 ARM Scalable Vector Extension 등 추가 아키텍처 대응과 성능 개선이 이루어졌다.

## 그 외 변경
- **JEP 416: 코어 리플렉션을 Method Handle로 재구현** — `java.lang.reflect.Method`/`Constructor`/`Field`의 내부 구현을 `java.lang.invoke` MethodHandle 기반으로 교체했다. 동작은 동일하되 유지보수성과 일관성이 개선되었다.
- **JEP 418: 인터넷 주소 해석 SPI** — 호스트명·주소 해석에 플랫폼 내장 리졸버 대신 커스텀 리졸버를 끼울 수 있는 서비스 제공자 인터페이스(SPI)를 도입했다. 비동기 DNS, 테스트용 가짜 리졸버 등을 구현할 수 있다.
- `finalize()` 메서드의 deprecation 진행, 작은 라이브러리 API 추가 등 점진적 정리가 이어졌다.

## 영향과 의의
Java 18 자체는 "조용한" 릴리스에 가깝지만, 그 안에 담긴 변화는 실무에 의미가 컸다.
- **UTF-8 기본화(JEP 400)**는 수년간 개발자를 괴롭히던 "환경마다 다른 인코딩" 문제를 플랫폼 차원에서 종결지은 결정적 변경이다.
- **jwebserver**는 "정적 파일 잠깐 서빙"에 외부 도구를 깔 필요를 없앴다.
- Panama(FFM, Vector)와 Amber(switch 패턴)의 단계적 진행은 Java 19~21에서 폭발할 가상 스레드·패턴 매칭·네이티브 상호운용의 토대를 다졌다.

## 참고 출처
- [JEP 400: UTF-8 by Default](https://openjdk.org/jeps/400)
- [JEP 408: Simple Web Server](https://openjdk.org/jeps/408)
- [JEP 413: Code Snippets in Java API Documentation](https://openjdk.org/jeps/413)
- [JEP 420: Pattern Matching for switch (Second Preview)](https://openjdk.org/jeps/420)
- [JEP 419: Foreign Function & Memory API (Second Incubator)](https://openjdk.org/jeps/419)
- [JEP 417: Vector API (Third Incubator)](https://openjdk.org/jeps/417)
- [JEP 416: Reimplement Core Reflection with Method Handles](https://openjdk.org/jeps/416)
- [JEP 418: Internet-Address Resolution SPI](https://openjdk.org/jeps/418)
- [OpenJDK JDK 18 프로젝트 페이지](https://openjdk.org/projects/jdk/18/)
- [InfoQ: Oracle Releases Java 18](https://www.infoq.com/news/2022/03/java-18-so-far/)
