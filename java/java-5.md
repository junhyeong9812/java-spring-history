# Java 5 (J2SE 5.0, Tiger, 2004년 9월)

> 제네릭, 어노테이션, enum, 향상된 for문 등을 한꺼번에 도입해 Java 언어 문법 자체를 가장 크게 바꾼 대변혁 릴리스.

## 릴리스 정보
- 정식 출시일: 2004년 9월 30일
- 개발 주체: Sun Microsystems (JCP, Java Community Process를 통한 표준화)
- 공식 명칭: J2SE 5.0 (내부 버전 1.5) — 마케팅상 버전 번호를 1.5에서 5.0으로 점프
- 코드네임: Tiger
- 플랫폼 스펙: JSR 176 (J2SE 5.0 Release Contents)
- LTS 여부: 해당 시대엔 LTS 개념 자체가 없었음 (LTS는 Java 8 이후 도입)

## 시대적 배경
J2SE 5.0 이전의 Java는 컬렉션을 다룰 때 모든 요소를 `Object`로 취급하고 꺼낼 때마다 캐스팅해야 했으며, 런타임에야 `ClassCastException`으로 타입 오류가 드러났다. 상수 집합은 `int` 상수나 어색한 typesafe enum 패턴으로 흉내 내야 했고, 메타데이터는 XML 설정 파일이나 주석 규약(XDoclet 등)에 의존했다.

당시 C#이 등장하며 언어 차원의 생산성을 강조하던 경쟁 환경에서, Sun은 "개발자가 더 적은 코드로 더 안전하게" 쓸 수 있도록 언어 문법을 대대적으로 손봤다. Tiger는 1.0 이후 Java 언어에 가해진 가장 큰 변화로 평가된다.

## 주요 추가 기능

### 제네릭 (Generics, JSR 14)
컴파일 타임 타입 안정성을 제공하고 대부분의 명시적 캐스팅을 제거한다.

도입 전:
```java
List list = new ArrayList();
list.add("hello");
// 잘못된 타입을 넣어도 컴파일러가 막지 못함
list.add(42);
String s = (String) list.get(0); // 매번 캐스팅 필요, 런타임 위험
```

도입 후:
```java
List<String> list = new ArrayList<String>();
list.add("hello");
// list.add(42); // 컴파일 에러 — 잘못된 타입을 컴파일 시점에 차단
String s = list.get(0); // 캐스팅 불필요
```

제네릭 메서드와 와일드카드도 함께 도입되었다:
```java
public static <T> T firstOf(List<T> list) {
    return list.get(0);
}

void printAll(List<? extends Number> nums) {
    for (Number n : nums) System.out.println(n);
}
```

### 어노테이션 / 메타데이터 (Annotations, JSR 175)
클래스, 메서드, 필드 등 언어 구성요소에 메타데이터를 부착할 수 있게 되었다. 프레임워크 설정이 XML에서 코드 내 어노테이션으로 이동하는 흐름의 출발점이다.

```java
@Override
public String toString() {
    return "tiger";
}

@Deprecated
public void oldApi() { }

// 사용자 정의 어노테이션
@interface Author {
    String name();
}

@Author(name = "Duke")
class Sample { }
```
표준 내장 어노테이션으로 `@Override`, `@Deprecated`, `@SuppressWarnings`가 추가되었다.

### 열거형 (enum, JSR 201)
타입 안전한 상수 집합을 언어 차원에서 지원한다.

도입 전:
```java
public static final int SEASON_SPRING = 0;
public static final int SEASON_SUMMER = 1;
// 단순 int라 타입 안전성 없음, 잘못된 값 전달 가능
```

도입 후:
```java
public enum Season { SPRING, SUMMER, FALL, WINTER }

Season s = Season.SPRING;
switch (s) {
    case SPRING:
        break;
    default:
        break;
}

// enum은 필드와 메서드도 가질 수 있는 완전한 클래스
public enum Planet {
    EARTH(9.8), MARS(3.7);
    private final double gravity;
    Planet(double g) { this.gravity = g; }
    public double gravity() { return gravity; }
}
```

### 향상된 for문 (for-each, JSR 201)
컬렉션과 배열 순회를 간결하게 한다.

도입 전:
```java
for (Iterator it = list.iterator(); it.hasNext(); ) {
    String s = (String) it.next();
    System.out.println(s);
}
```

도입 후:
```java
for (String s : list) {
    System.out.println(s);
}
```

### 오토박싱 / 언박싱 (Autoboxing/Unboxing, JSR 201)
기본형과 래퍼 타입 간 변환을 자동화한다.

도입 전:
```java
List<Integer> nums = new ArrayList<Integer>();
nums.add(Integer.valueOf(10));      // 수동 박싱
int x = nums.get(0).intValue();     // 수동 언박싱
```

도입 후:
```java
List<Integer> nums = new ArrayList<Integer>();
nums.add(10);          // 자동 박싱 (int -> Integer)
int x = nums.get(0);   // 자동 언박싱 (Integer -> int)
```

### 가변인자 (Varargs, JSR 201)
개수가 정해지지 않은 인자를 배열처럼 받을 수 있다.

```java
static int sum(int... values) {
    int total = 0;
    for (int v : values) total += v;
    return total;
}
sum(1, 2, 3, 4); // 호출 시 개수 자유
```
`printf`/`String.format`도 이 기능 위에서 동작한다.

### 정적 임포트 (Static Import)
정적 멤버를 클래스명 없이 사용할 수 있다.

```java
import static java.lang.Math.*;

double r = sqrt(PI * 2); // Math.sqrt, Math.PI 대신
```

### 동시성 유틸리티 (java.util.concurrent, JSR 166)
Doug Lea가 주도한 고수준 동시성 라이브러리가 표준에 편입되었다. `ExecutorService`, `ConcurrentHashMap`, `CountDownLatch`, `BlockingQueue`, 원자적 변수(`AtomicInteger` 등)를 제공한다.

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> f = pool.submit(new Callable<Integer>() {
    public Integer call() { return 1 + 1; }
});
Integer result = f.get();
pool.shutdown();
```

### Scanner (java.util.Scanner)
콘솔/스트림 입력을 간단히 파싱한다.

```java
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
String line = sc.next();
```

## 그 외 변경 / API 추가
- `printf` / `format` 스타일 포매팅 (`java.util.Formatter`)
- `StringBuilder` 추가 (`StringBuffer`의 동기화하지 않는(non-synchronized) 버전으로 단일 스레드에서 더 빠름)
- `java.lang.instrument` 패키지 (자바 에이전트 기반 계측)
- JVM Tool Interface(JVMTI), JPDA 개선
- 공유 클래스 데이터(Class Data Sharing)로 시작 시간 단축

## 영향과 의의
J2SE 5.0은 Java를 "장황하지만 안전한" 언어에서 "현대적 타입 시스템을 갖춘" 언어로 끌어올렸다. 제네릭과 어노테이션은 이후 Spring, Hibernate, JUnit 4 같은 어노테이션 기반 프레임워크 생태계의 토대가 되었고, `java.util.concurrent`는 멀티코어 시대의 표준 동시성 모델을 제시했다. 오늘날 우리가 쓰는 Java 문법의 상당 부분이 이때 형태를 갖췄으며, Tiger는 Java 역사상 가장 영향력 큰 단일 릴리스로 꼽힌다.

## 참고 출처
- [Java version history — Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [J2SE 5.0 — Oracle](https://www.oracle.com/java/technologies/javase/j2se-1-5.html)
- [JSR 176: J2SE 5.0 Release Contents — JCP](https://jcp.org/en/jsr/detail?id=176)
- [J2SE 5.0 (September 30, 2004) — Liquisearch](https://www.liquisearch.com/java_version_history/j2se_50_september_30_2004)
