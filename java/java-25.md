# Java 25 (2025.09) — LTS

> Java 21에 이은 차세대 LTS. 스코프드 값·모듈 임포트 선언·유연한 생성자 본문·컴팩트 소스 파일과 인스턴스 main을 정식화하고, 컴팩트 객체 헤더·세대별 Shenandoah를 안정화한 집대성 릴리스.

## 릴리스 정보
- 정식 출시일: 2025년 9월 16일
- LTS 여부: **예** (장기 지원 / Oracle 기준 최소 8년 지원). 직전 LTS는 Java 21(2023.09)
- 포함 JEP 목록 (총 18개):
  - JEP 470: PEM Encodings of Cryptographic Objects (프리뷰)
  - JEP 502: Stable Values (프리뷰)
  - JEP 503: Remove the 32-bit x86 Port (정식)
  - JEP 505: Structured Concurrency (5차 프리뷰)
  - JEP 506: Scoped Values (정식)
  - JEP 507: Primitive Types in Patterns, instanceof, and switch (3차 프리뷰)
  - JEP 508: Vector API (10차 인큐베이터)
  - JEP 509: JFR CPU-Time Profiling (실험적)
  - JEP 510: Key Derivation Function API (정식)
  - JEP 511: Module Import Declarations (정식)
  - JEP 512: Compact Source Files and Instance Main Methods (정식)
  - JEP 513: Flexible Constructor Bodies (정식)
  - JEP 514: Ahead-of-Time Command-Line Ergonomics (정식)
  - JEP 515: Ahead-of-Time Method Profiling (정식)
  - JEP 518: JFR Cooperative Sampling (정식)
  - JEP 519: Compact Object Headers (정식)
  - JEP 520: JFR Method Timing & Tracing (정식)
  - JEP 521: Generational Shenandoah (정식)

## 시대적 배경

Java 21 이후 2년간(22·23·24) 프리뷰·인큐베이터·실험 단계를 거쳐 온 기능들이 마침내 정식 안착하는 LTS 릴리스다. 기업·프레임워크가 실제로 장기간 채택할 기준 버전이므로, "여러 프로젝트의 결실을 안정화해 모은다"는 성격이 강하다. **Project Loom**의 스코프드 값(정식)과 구조적 동시성(여전히 프리뷰), **Project Amber**의 모듈 임포트·유연한 생성자 본문·컴팩트 소스 파일(모두 정식), **Project Leyden**의 AOT 메서드 프로파일링·명령행 인체공학(정식), **Project Valhalla**로 가는 길목의 컴팩트 객체 헤더(정식) 등이 한자리에 모였다. 또한 32비트 x86 포트가 완전히 제거되며 한 시대를 마감했다.

## 주요 추가 기능

### Scoped Values (JEP 506, 정식)
- 스레드(특히 가상 스레드) 간 불변 데이터를 `ThreadLocal`보다 안전하고 효율적으로 공유한다. 22~24의 여러 프리뷰를 거쳐 LTS에서 정식화되었다. 값은 명시된 동적 범위(`run`/`call`) 안에서만 유효하며 불변이다.

```java
final static ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

ScopedValue.where(CURRENT_USER, user).run(() -> {
    handleRequest();          // 이 범위 안에서만 CURRENT_USER.get() 가능
});
```

### Module Import Declarations (JEP 511, 정식)
- 모듈이 export하는 모든 패키지를 한 줄로 임포트한다. 23 프리뷰 → 24 2차 프리뷰 → 25 정식.

```java
import module java.base;
import module java.sql;

void main() {
    var list = List.of(1, 2, 3);     // java.util.* 자동 가용
}
```

### Compact Source Files and Instance Main Methods (JEP 512, 정식)
- 초보자 친화 진입점이 정식화되었다(여러 차례 프리뷰를 거쳐 명칭이 "Compact Source Files"로 확정). 클래스 선언과 `String[] args` 없이 인스턴스 `main` 메서드만으로 프로그램을 작성할 수 있고, 콘솔 입출력 헬퍼(`java.lang.IO`)를 제공한다.

```java
// 클래스/메서드 시그니처 보일러플레이트 없이 실행 가능
void main() {
    IO.println("Hello, Java 25!");
}
```

### Flexible Constructor Bodies (JEP 513, 정식)
- 명시적 생성자 호출(`super(...)`/`this(...)`) 이전에 인자 검증·필드 준비 등의 문장을 둘 수 있다. JDK 22의 JEP 447에서 출발해 LTS에서 정식화되었다.

```java
class Temperature {
    final double celsius;
    Temperature(double value) {
        if (value < -273.15)                      // super 이전 검증
            throw new IllegalArgumentException("절대영도 미만");
        this.celsius = value;
    }
}
```

### Key Derivation Function API (JEP 510, 정식)
- HKDF 등 키 유도 함수(KDF)를 위한 표준 API가 정식화되었다(24 프리뷰 → 25 정식). 포스트양자 암호 등 현대 프로토콜에 필요한 키 파생을 표준화한다.

```java
KDF hkdf = KDF.getInstance("HKDF-SHA256");
AlgorithmParameterSpec params = HKDFParameterSpec.ofExtract()
        .addIKM(secretKey).addSalt(salt).thenExpand(info, 32);
SecretKey derived = hkdf.deriveKey("AES", params);
```

### Compact Object Headers (JEP 519, 정식)
- 객체 헤더를 (보통) 12바이트에서 8바이트로 줄여 힙 메모리 사용량을 절감한다. 24의 실험적 기능에서 LTS에서 제품(정식) 기능으로 승격되었다. 수백만 객체를 다루는 애플리케이션에서 메모리·캐시 효율이 향상된다.

```bash
java -XX:+UseCompactObjectHeaders -jar app.jar
```

### Generational Shenandoah (JEP 521, 정식)
- Shenandoah GC의 세대별 모드가 정식화되었다(24 실험적 → 25 정식). 짧게 사는 객체가 많은 워크로드에서 처리량과 지연을 개선한다.

## 그 외 변경
- **Ahead-of-Time Method Profiling (JEP 515)** / **Ahead-of-Time Command-Line Ergonomics (JEP 514)**: Project Leyden 후속. 메서드 프로파일을 미리 수집해 워밍업을 단축하고, AOT 캐시 사용을 위한 명령행 옵션을 단순화한다.
- **JFR 강화**: CPU-Time Profiling(JEP 509, 실험적, 리눅스), Cooperative Sampling(JEP 518), Method Timing & Tracing(JEP 520)으로 Flight Recorder의 프로파일링·진단 능력을 확장한다.
- **Stable Values (JEP 502, 프리뷰)**: `final`의 불변성과 지연 초기화의 유연성을 결합한 "안정값" API의 첫 프리뷰. 한 번만 설정되며 JIT가 상수처럼 최적화할 수 있다.
- **PEM Encodings of Cryptographic Objects (JEP 470, 프리뷰)**: 키·인증서 등 암호 객체를 PEM 텍스트로 인코딩/디코딩하는 표준 API의 첫 프리뷰.
- **Structured Concurrency (JEP 505, 5차 프리뷰)**: 아직 정식화되지 않고 API를 다듬는 5차 프리뷰로 유지된다.
- **Primitive Types in Patterns, instanceof, and switch (JEP 507, 3차 프리뷰)**: 원시 타입 패턴 매칭은 LTS에서도 여전히 프리뷰(3차)다.
- **Vector API (JEP 508, 10차 인큐베이터)**: Valhalla 의존성으로 인해 LTS에서도 여전히 인큐베이터(10차)에 머문다.
- **Remove the 32-bit x86 Port (JEP 503, 정식)**: 32비트 x86 포트와 관련 빌드를 완전히 제거.

## 영향과 의의

Java 25는 Java 21 이후 2년간 축적된 차세대 기능들을 정식화해 모은 LTS로, 기업의 차기 표준 채택 기준점이 된다. 스코프드 값·유연한 생성자 본문·모듈 임포트의 정식화는 Loom·Amber 시대의 언어/라이브러리 모델을 굳혔고, 컴팩트 객체 헤더와 세대별 Shenandoah, AOT 프로파일링은 메모리·시작 성능을 LTS 수준으로 안정화했다. 다만 구조적 동시성·원시 타입 패턴·Vector API·String Templates(보류) 등은 여전히 미완으로 남아, 다음 LTS(예정상 Java 29)로 숙제를 넘긴다.

## 참고 출처
- [JDK 25 - OpenJDK 프로젝트 페이지](https://openjdk.org/projects/jdk/25/)
- [Oracle Releases Java 25](https://www.oracle.com/news/announcement/oracle-releases-java-25-2025-09-16/)
- [The Arrival of Java 25 - Oracle Java Blog](https://blogs.oracle.com/java/the-arrival-of-java-25)
- [New Features in Java 25 - Baeldung](https://www.baeldung.com/java-25-features)
- [JDK 25 and JDK 26: What We Know So Far - InfoQ](https://www.infoq.com/news/2025/08/java-25-so-far/)
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
