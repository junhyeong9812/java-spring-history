# JDK 1.4 / J2SE 1.4 (코드네임 Merlin, 2002년 2월)

> 언어와 라이브러리 양면에서 풍성한 기능을 더한 버전. `assert` 키워드로 언어가 처음 확장되었고, NIO·정규식·로깅·XML 파서·예외 체이닝 등 오늘날 자바 개발자가 매일 쓰는 API가 표준에 대거 편입되었다.

## 릴리스 정보
- 정식 출시일: 2002년 2월 6일
- 개발 주체: Sun Microsystems
- 공식 명칭: J2SE 1.4 (Java 2 Platform, Standard Edition v1.4)
- 코드네임: Merlin (멀린, 쇠황조롱이)
- 참고: JСР(Java Community Process)를 거쳐 개발된 최초의 자바 플랫폼 릴리스(JSR 59).

## 시대적 배경
J2SE 1.3이 성능·안정성을 다졌다면, 1.4는 다시 기능 확장으로 방향을 잡았다. 당시 자바 애플리케이션은 점점 대형화·서버화되었고, 고성능 I/O, 표준 로깅, XML 처리, 정규식 같은 기능을 그동안 서드파티 라이브러리(예: Apache Log4j, Jakarta ORO, JDOM)에 의존해 왔다. 1.4는 이런 사실상 표준(de facto standard)들을 플랫폼 자체에 흡수하여, 외부 의존 없이도 견고한 애플리케이션을 만들 수 있게 했다.

## 주요 추가 기능

### assert 키워드
- 자바 언어에 처음으로 추가된 새 키워드. 프로그램의 가정(invariant)을 코드로 명시하고, 실행 시 검증할 수 있다.
- 기본적으로 비활성화되어 있으며 `-ea` 옵션으로 켠다(운영 성능에 영향 없음).

```java
public int sqrt(int x) {
    assert x >= 0 : "음수는 허용되지 않음: " + x;
    return (int) Math.sqrt(x);
}
// 실행: java -ea Main  (assertion 활성화)
```

### NIO (New I/O) — java.nio
- 채널(Channel), 버퍼(Buffer), 셀렉터(Selector) 기반의 고성능·논블로킹 I/O. 대규모 동시 연결을 효율적으로 처리할 수 있게 되어 서버 프로그래밍에 큰 영향을 주었다.

```java
import java.nio.*;
import java.nio.channels.*;
import java.io.RandomAccessFile;

FileChannel ch = new RandomAccessFile("data.txt", "r").getChannel();
ByteBuffer buf = ByteBuffer.allocate(1024);
ch.read(buf);
buf.flip();
while (buf.hasRemaining()) {
    System.out.print((char) buf.get());
}
ch.close();
```

### 정규 표현식 — java.util.regex
- Perl 스타일 정규식을 표준 라이브러리로 제공. `Pattern`과 `Matcher`로 문자열 매칭·치환·추출을 수행한다.

```java
import java.util.regex.*;

Pattern p = Pattern.compile("(\\d{4})-(\\d{2})-(\\d{2})");
Matcher m = p.matcher("오늘은 2002-02-06 입니다");
if (m.find()) {
    System.out.println("연도: " + m.group(1)); // 2002
}
```

### 로깅 API — java.util.logging
- 표준 로깅 프레임워크. 로거(Logger), 핸들러(Handler), 레벨(Level) 개념으로 애플리케이션 로그를 체계적으로 남길 수 있다.

```java
import java.util.logging.*;

Logger log = Logger.getLogger("app");
log.info("애플리케이션 시작");
log.warning("설정 파일이 비어 있음");
```

### 예외 체이닝(Exception Chaining)
- 한 예외가 다른 예외로 인해 발생했음을 "원인(cause)"으로 연결해 보존하는 기능. `Throwable`에 `getCause()`와 cause 생성자가 추가되어, 저수준 예외를 감싸 던지면서도 원래 스택 트레이스를 잃지 않는다.

```java
try {
    loadConfig();
} catch (IOException e) {
    throw new RuntimeException("설정 로딩 실패", e); // e를 cause로 연결
}
```

### XML 처리 — JAXP
- JAXP(Java API for XML Processing)가 코어에 포함되어, DOM·SAX 파서와 XSLT 변환을 표준으로 사용할 수 있게 되었다.

```java
import javax.xml.parsers.*;
import org.w3c.dom.Document;

DocumentBuilder db = DocumentBuilderFactory.newInstance().newDocumentBuilder();
Document doc = db.parse("config.xml");
System.out.println(doc.getDocumentElement().getNodeName());
```

### IPv6 지원
- `java.net`에 IPv6 주소 지원이 추가되어, 차세대 인터넷 프로토콜 환경에서도 동작하게 되었다.

### Java Web Start
- 웹 링크 클릭만으로 데스크톱 자바 애플리케이션을 다운로드·실행·자동 업데이트하는 배포 기술(JNLP 기반)이다.
- 엄밀히는 Web Start 1.0이 2001년 3월에 별도 제품으로 먼저 출시되었고, J2SE 1.4부터 JRE에 기본 번들로 통합되었다. (1.4에서 처음 만들어진 것이 아니라 1.4에서 플랫폼에 포함된 것)

## 그 외 변경 / API 추가
- Image I/O API: 다양한 이미지 포맷의 읽기/쓰기 표준화.
- Preferences API(`java.util.prefs`): 사용자/시스템 환경설정 저장.
- JAAS, JSSE, JCE 등 보안·암호화 확장이 코어에 통합.
- `LinkedHashMap`, `LinkedHashSet` 등 컬렉션 보강, `assert` 외 정수 비트연산 유틸 등.

## 영향과 의의
J2SE 1.4는 "외부 라이브러리에 기대던 핵심 기능들을 표준으로 끌어들인" 버전이다. NIO는 고성능 서버·네트워킹 프레임워크의 토대가 되었고, 정규식·로깅·XML·예외 체이닝은 지금도 거의 모든 자바 프로젝트에서 사용된다. 또한 JCP를 통해 개발된 최초의 릴리스로서, 이후 자바 발전이 커뮤니티 표준 프로세스로 운영되는 관행을 정착시켰다. 이 버전을 끝으로 다음 릴리스(2004)에서는 "Java 2"의 1.x 번호 체계를 버리고 J2SE 5.0으로 도약하며, 제네릭·오토박싱·애너테이션 등 대규모 언어 개편이 이어진다.

## 참고 출처
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [Java 1.4 - javaalmanac.io](https://javaalmanac.io/jdk/1.4/)
- [JDK release dates - Java Glossary (mindprod)](https://www.mindprod.com/jgloss/jdkreleasedates.html)
