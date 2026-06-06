# Java 6 (Java SE 6, Mustang, 2006년 12월)

> 언어 문법 변화는 거의 없지만 JVM 성능을 대폭 끌어올리고 스크립팅·컴파일러 API, 웹서비스 스택을 플랫폼에 내장한 "성능과 통합"의 릴리스.

## 릴리스 정보
- 정식 출시일: 2006년 12월 11일
- 개발 주체: Sun Microsystems (JCP 표준화)
- 공식 명칭: Java SE 6 (내부 버전 1.6) — 이 버전부터 "J2SE"에서 "Java SE"로 브랜드 변경
- 코드네임: Mustang
- 플랫폼 스펙: JSR 270 (Java SE 6 Release Contents)
- LTS 여부: 해당 시대엔 LTS 개념 없음

## 시대적 배경
J2SE 5.0이 언어 문법을 한꺼번에 갈아엎은 직후라, Sun은 Java SE 6에서 의도적으로 언어 변경을 최소화하고 **JVM 성능, 진단 도구, 라이브러리 통합**에 집중했다. 또한 SOAP/WSDL 기반 웹서비스가 엔터프라이즈 표준으로 자리 잡던 시기라, 그동안 별도 다운로드로 쓰던 JAXB·JAX-WS 같은 웹서비스 스택을 JDK 본체에 포함했다.

Mustang은 개발 과정을 오픈으로 공개해 정기 빌드를 배포한 첫 릴리스이기도 하며, 직후인 2006~2007년 Sun이 Java를 GPL로 오픈소스화(OpenJDK)하는 흐름과 맞물린다.

## 주요 추가 기능

### 스크립팅 API (Scripting for the Java Platform, JSR 223)
JVM 위에서 스크립트 언어를 실행하는 표준 프레임워크다. Mozilla Rhino 기반 JavaScript 엔진이 기본 탑재되어, 별도 라이브러리 없이 Java에서 스크립트를 평가할 수 있게 되었다.

```java
import javax.script.*;

ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("JavaScript");
Object result = engine.eval("var x = 10; x * 2;");
System.out.println(result); // 20.0 (Rhino는 JS 숫자를 Double로 반환)

// Java 변수를 스크립트로 바인딩
engine.put("name", "Mustang");
engine.eval("print('Hello ' + name)");
```

### 컴파일러 API (Java Compiler API, JSR 199)
런타임에 Java 소스를 프로그래밍 방식으로 컴파일할 수 있다. 이전에는 임시 `.java` 파일을 만들고 `javac`를 외부 프로세스로 호출하거나 내부 API를 해킹해야 했다.

```java
import javax.tools.*;

JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
int result = compiler.run(null, null, null, "Hello.java");
System.out.println(result == 0 ? "컴파일 성공" : "실패");
```
이 API는 동적 코드 생성, JSP 컨테이너, 어노테이션 처리 도구의 기반이 되었다.

### 플러그인 가능한 어노테이션 처리 (Pluggable Annotation Processing, JSR 269)
컴파일 시점에 어노테이션을 처리하는 표준 API(`javax.annotation.processing`, `javax.lang.model`)가 도입되었다. 별도 도구였던 `apt`를 대체하며, 이후 Lombok·Dagger·MapStruct 같은 코드 생성 도구의 토대가 된다.

### JDBC 4.0 (JSR 221)
데이터베이스 접근이 한결 간결해졌다. 드라이버 자동 로딩(`Class.forName` 불필요), `SQLException` 계층 개선(`SQLException`이 `Iterable`을 구현, 원인별 서브클래스 추가), `SQLXML`·`RowId`·`NClob` 등 새 SQL 타입 지원이 추가되었다.

도입 전 (JDBC 3.0):
```java
Class.forName("com.mysql.jdbc.Driver"); // 수동 드라이버 로딩 필요
Connection conn = DriverManager.getConnection(url, user, pw);
```

도입 후 (JDBC 4.0):
```java
// 드라이버가 클래스패스에 있으면 자동 로딩됨
Connection conn = DriverManager.getConnection(url, user, pw);
```

### JAXB 2.0 (JSR 222) 및 웹서비스 스택
XML 바인딩(JAXB 2.0)과 SOAP 웹서비스(JAX-WS 2.0, JSR 224)가 JDK에 내장되어 어노테이션만으로 웹서비스를 만들 수 있게 되었다.

```java
import javax.jws.WebService;

@WebService
public class Calculator {
    public int add(int a, int b) { return a + b; }
}
```
함께 StAX(Streaming API for XML, JSR 173)도 표준 라이브러리에 편입되었다.

### GUI / 데스크톱 개선
- Swing의 `GroupLayout` 레이아웃 매니저 (NetBeans Matisse 디자이너 지원)
- `SwingWorker` 표준 편입 (백그라운드 작업과 EDT 분리)
- 테이블 정렬·필터링, 향상된 드래그앤드롭
- `java.awt.Desktop` API (기본 브라우저/메일/파일 연결 실행)
- 시스템 트레이 지원 (`SystemTray`, `TrayIcon`)

## 그 외 변경 / API 추가
- JVM 성능 대폭 개선 (서버/클라이언트 HotSpot 최적화, 동기화·가비지 컬렉션 튜닝) — Java 6은 "체감 성능" 향상으로 특히 유명하다
- `java.lang.management` 기반 모니터링·진단 강화, JConsole 개선
- Common Annotations (JSR 250) 편입 (`@PostConstruct`, `@Resource` 등)
- `Console` 클래스 (`System.console()`)로 비밀번호 입력 등 콘솔 처리 개선
- 다양한 `NavigableMap`/`NavigableSet` 등 컬렉션 보강

## 영향과 의의
Java SE 6은 "조용하지만 강력한" 릴리스로 평가된다. 새 문법으로 개발자를 흥분시키진 않았지만, 눈에 띄는 성능 향상과 웹서비스·스크립팅·컴파일러 API의 플랫폼 통합으로 엔터프라이즈 현장에서 매우 오래 사랑받았다. 실제로 Java 7 출시가 Sun의 경영난과 Oracle 인수로 지연되면서, Java 6은 여러 해 동안 사실상의 산업 표준 JDK로 장기 군림했다.

## 참고 출처
- [Java version history — Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [Java Platform, Standard Edition 6 — Oracle](https://www.oracle.com/java/technologies/se6-jsp.html)
- [JSR 270: Java SE 6 Release Contents — JCP](https://jcp.org/en/jsr/detail?id=270)
- [Java SE 6 (December 11, 2006) — Liquisearch](https://www.liquisearch.com/java_version_history/java_se_6_december_11_2006)
