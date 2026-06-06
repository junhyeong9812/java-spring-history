# Java 10 (2018년 3월)

> 6개월 릴리스 케이던스의 첫 결과물. `var` 지역변수 타입 추론으로 자바 코드의 장황함을 줄인 버전.

## 릴리스 정보
- 정식 출시일: 2018년 3월 20일
- 개발 주체: Oracle (OpenJDK)
- LTS 여부: 아니오 (단기 지원, 다음 정식 릴리스 출시까지 지원)
- 포함 JEP 수: 12개

## 시대적 배경
Java 9까지 자바는 수 년에 한 번씩 대형 릴리스를 내놓는 방식이었다. Java 9는 모듈 시스템(Jigsaw) 때문에 여러 차례 연기되며 약 3년 반이 걸렸다. 이런 지연을 끝내기 위해 Oracle은 **6개월 주기의 시간 기반(time-based) 릴리스 케이던스**를 도입했고, Java 10이 그 첫 결과물이다. 이제 기능이 준비되면 그 시점의 릴리스 열차에 실리고, 준비되지 않으면 6개월 뒤 다음 열차를 탄다. 이 변화는 JEP 322(Time-Based Release Versioning)로 버전 체계에도 반영되어, Java 10은 `18.3`(2018년 3월)이라는 내부 버전 표기를 함께 가진다.

## 주요 추가 기능

### 지역변수 타입 추론 `var` (JEP 286)
- 상태: **정식 기능(standard)**
- 지역변수 선언 시 타입 자리에 `var`를 쓰면 컴파일러가 우변(initializer)으로부터 타입을 추론한다. 동적 타입이 아니라 여전히 정적 타입이며, 단지 명시적 타입 표기를 생략하는 문법적 편의다.
- 사용 가능 위치: 초기화식이 있는 지역변수, for/for-each 루프 변수, try-with-resources 변수.
- 사용 불가 위치: 멤버 변수(필드), 메서드 파라미터, 메서드 반환 타입, 람다 파라미터, 초기화 없는 선언, `null`만 대입하는 경우.

```java
// 기존 방식
ArrayList<Map<String, Integer>> list = new ArrayList<Map<String, Integer>>();

// var 사용
var list = new ArrayList<Map<String, Integer>>();   // ArrayList<Map<String,Integer>>로 추론
var message = "Hello, Java 10";                       // String
var count = 10;                                       // int

for (var i = 0; i < list.size(); i++) { /* ... */ }   // int
for (var entry : someMap.entrySet()) { /* ... */ }    // Map.Entry<...>

try (var reader = Files.newBufferedReader(path)) {    // BufferedReader
    // ...
}
```

### G1을 위한 병렬 Full GC (JEP 307)
- 상태: 정식 기능
- Java 9부터 기본 GC가 된 G1은 그동안 Full GC를 **단일 스레드**로 수행해 최악의 경우 일시정지가 길었다. JEP 307은 Full GC도 멀티스레드로 병렬화해 최악 지연 시간을 개선했다.

### 애플리케이션 클래스 데이터 공유 / AppCDS (JEP 310)
- 상태: 정식 기능
- 기존 CDS(Class-Data Sharing)는 부트스트랩 클래스만 공유 아카이브에 담을 수 있었다. AppCDS는 **애플리케이션 클래스**까지 아카이브에 포함해 여러 JVM 프로세스가 메모리에 매핑된 클래스 메타데이터를 공유하도록 한다. 시작 시간 단축과 메모리 풋프린트 절감 효과가 있다.

### 가비지 컬렉터 인터페이스 (JEP 304)
- 상태: 정식 기능(내부 구조 개선)
- HotSpot 내부에 GC들이 구현해야 하는 깔끔한 **공통 인터페이스**를 도입했다. GC 코드가 여기저기 흩어져 있던 것을 정리해, 새로운 GC를 추가하거나 기존 GC를 제거하기 쉽게 만든 기반 작업이다. 이후 ZGC, Shenandoah 등 새 GC 도입의 토대가 되었다.

## 그 외 변경 / API 추가
- **JEP 322 — 시간 기반 릴리스 버전 체계**: `$FEATURE.$INTERIM.$UPDATE.$PATCH` 형식의 새 버전 문자열 도입.
- **JEP 312 — 스레드 로컬 핸드셰이크(Thread-Local Handshakes)**: 모든 스레드를 멈추는 전역 safepoint 없이 개별 스레드에 콜백을 실행하는 메커니즘. GC 지연 개선의 기반.
- **JEP 316 — 대체 메모리 장치에 힙 할당**: NV-DIMM 같은 대체 메모리에 힙을 할당할 수 있게 함.
- **JEP 317 — 실험적 자바 기반 JIT 컴파일러(Graal)**: Graal을 실험적 JIT 컴파일러로 사용할 수 있게 함(Linux/x64).
- **JEP 319 — 루트 인증서**: OpenJDK에 기본 루트 CA 인증서 세트를 포함시켜, 별도 설정 없이도 TLS 검증이 동작하도록 함.
- **JEP 314 — 추가 유니코드 언어 태그 확장**.
- **JEP 296 — JDK 포레스트를 단일 리포지토리로 통합**, **JEP 313 — `javah` 네이티브 헤더 생성 도구 제거**(내부/툴링 변경).
- **API**: `Optional.orElseThrow()`(인자 없는 버전), `List.copyOf`/`Set.copyOf`/`Map.copyOf` 등 변경 불가 컬렉션 복사 팩토리, `Collectors.toUnmodifiableList/Set/Map` 추가.

## 영향과 의의
Java 10은 기능 자체보다 **새로운 릴리스 리듬을 증명한** 릴리스다. "6개월마다 진짜로 출시된다"는 것을 보여주며, 이후 자바가 빠르게 진화할 수 있는 기반을 만들었다. 기능 면에서는 `var`가 단연 화제였다. Scala/Kotlin/C# 등에 익숙한 개발자들이 기대하던 타입 추론을 자바에 들여왔고, 가독성과 남용 사이의 균형에 대한 커뮤니티 토론을 촉발했다. GC 인터페이스(JEP 304)와 스레드 로컬 핸드셰이크(JEP 312)는 눈에 띄지 않지만, 이후 ZGC·Shenandoah 같은 저지연 GC 시대를 여는 중요한 토대가 되었다.

## 참고 출처
- [JDK 10 — OpenJDK Project](https://openjdk.org/projects/jdk/10/)
- [JEP 286: Local-Variable Type Inference](https://openjdk.org/jeps/286)
- [Java version history — Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [Simplify Local Variable Type Definition Using the Java 10 var Keyword — Red Hat Developer](https://developers.redhat.com/blog/2018/05/25/simplify-local-variable-type-definition-using-the-java-10-var-keyword)
- [Java 10 Features (with Examples) — HappyCoders](https://www.happycoders.eu/java/java-10-features/)
