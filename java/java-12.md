# Java 12 (2019년 3월)

> switch 표현식을 처음 선보이며(preview) 자바 언어 문법의 현대화를 시작한 버전.

## 릴리스 정보
- 정식 출시일: 2019년 3월 19일
- 개발 주체: Oracle (OpenJDK)
- LTS 여부: 아니오 (단기 지원)
- 포함 JEP 수: 8개

## 시대적 배경
Java 11(LTS) 이후 첫 단기 릴리스다. 6개월 케이던스가 안정적으로 자리 잡으면서, 이제 Oracle은 큰 언어 기능을 **preview(미리보기)** 형태로 먼저 출시해 커뮤니티 피드백을 받고, 이후 릴리스에서 다듬어 정식화하는 패턴을 도입했다. Java 12의 switch 표현식이 그 첫 사례로, 이 "preview → 안정화 → 정식" 프로세스는 이후 텍스트 블록, records, sealed classes, pattern matching 등 자바 언어 진화의 표준 절차가 된다.

## 주요 추가 기능

### switch 표현식 (JEP 325)
- 상태: **미리보기(Preview)** — 컴파일·실행 시 `--enable-preview --release 12` 필요.
- 기존 `switch`는 문(statement)일 뿐이고 fall-through(break 누락 시 다음 case로 흘러내림) 버그가 잦았다. JEP 325는 switch를 **값을 돌려주는 표현식**으로도 쓸 수 있게 하고, 화살표(`->`) 문법을 도입해 fall-through를 없애고 여러 레이블을 콤마로 묶을 수 있게 했다.

```java
// 기존 switch 문 (fall-through 위험, break 필요)
int numLetters;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        numLetters = 6;
        break;
    case TUESDAY:
        numLetters = 7;
        break;
    default:
        throw new IllegalStateException();
}

// Java 12 switch 표현식 (preview): 화살표 문법, 값 반환
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```
> 참고: 이 시점에는 블록에서 값을 반환할 때 `break <값>;` 문법을 썼다. 이후 Java 13의 2차 preview에서 `yield`로 대체된다.

### Shenandoah GC (JEP 189)
- 상태: **실험적(Experimental)** — Red Hat이 주도한 저지연 GC.
- 힙 크기와 무관하게 짧고 일정한 일시정지를 목표로, 애플리케이션 스레드와 **동시에(concurrent)** 객체를 정리(compaction 포함)한다. ZGC와 함께 자바 저지연 GC 시대를 여는 또 다른 축이다.

### 마이크로벤치마크 스위트 (JEP 230)
- 상태: 정식(빌드/툴링)
- JMH(Java Microbenchmark Harness) 기반의 마이크로벤치마크 모음을 JDK 소스에 포함시켜, JDK 자체 성능 회귀를 쉽게 측정하고 새 벤치마크를 추가할 수 있게 했다.

## 그 외 변경 / API 추가
- **컴팩트 숫자 포맷(Compact Number Formatting)** — API 추가: `NumberFormat.getCompactNumberInstance()`로 큰 숫자를 "1K", "1M", "1만", "1000만" 같은 짧은 로케일 인지 형식으로 표현.
  ```java
  NumberFormat fmt = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);
  fmt.format(1_000);      // "1K"
  fmt.format(1_000_000);  // "1M"
  ```
- **JEP 341 — 기본 CDS 아카이브**: JDK 빌드 시 기본 클래스 리스트로 CDS 아카이브를 미리 생성해 기본 제공. 시작 시간 개선.
- **JEP 344 — G1의 중단 가능한 혼합 컬렉션(Abortable Mixed Collections)**: 일시정지 목표를 초과할 것 같으면 혼합 GC를 중단해 지연 목표를 더 잘 지킴.
- **JEP 346 — G1의 미사용 메모리 즉시 반환**: 유휴 시점에 커밋된 힙 메모리를 OS에 더 신속히 돌려줌.
- **JEP 334 — JVM Constants API**: `java.lang.constant` 패키지 도입. class-file/런타임 아티팩트(상수 풀에 로드 가능한 상수)를 명목상으로 기술하는 API.
- **JEP 340 — 하나의 AArch64 포트만 유지**: 중복된 64비트 ARM 포트 중 하나를 제거.
- **API**: `String.indent(int)`, `String.transform(Function)`, `Collectors.teeing(...)`(두 컬렉터 결과를 합치는 다운스트림), `Files.mismatch(Path, Path)` 등 추가.

## 영향과 의의
Java 12는 기능 수는 적지만 **언어 진화의 방법론을 확립한** 릴리스다. switch 표현식을 preview로 내놓아 "정식화 전에 실험하고 피드백받는" 절차를 처음 적용했고, 이는 이후 자바가 안전하게 빠른 문법 혁신을 이어가는 틀이 되었다. GC 측면에서는 Shenandoah가 합류하면서 ZGC와 함께 저지연 GC 선택지가 넓어졌다. 컴팩트 숫자 포맷, `Collectors.teeing` 같은 작지만 실용적인 API도 더해졌다.

## 참고 출처
- [JDK 12 — OpenJDK Project](https://openjdk.org/projects/jdk/12/)
- [JEP 325: Switch Expressions (Preview)](https://openjdk.org/jeps/325)
- [JEP 230: Microbenchmark Suite](https://openjdk.org/jeps/230)
- [Java 12 Released with Experimental Switch Expressions and Shenandoah GC — InfoQ](https://www.infoq.com/news/2019/03/java12-released/)
- [Java 12 Features — DigitalOcean](https://www.digitalocean.com/community/tutorials/java-12-features)
- [Java version history — Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
