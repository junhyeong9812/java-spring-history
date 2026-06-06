# Java 7 (Java SE 7, Dolphin, 2011년 7월)

> Oracle의 Sun 인수 후 첫 메이저 릴리스. "Project Coin"이라는 소소한 언어 개선 묶음과 NIO.2, Fork/Join, invokedynamic을 담아 5년 만에 Java를 다시 진전시킨 버전.

## 릴리스 정보
- 정식 출시일: 2011년 7월 28일 (General Availability) — 발표는 7월 7일
- 개발 주체: Oracle (2010년 Sun Microsystems 인수 완료 후 첫 릴리스), OpenJDK 커뮤니티 협업
- 공식 명칭: Java SE 7 (내부 버전 1.7)
- 코드네임: Dolphin
- 플랫폼 스펙: JSR 336 (Java SE 7 Release Contents)
- LTS 여부: 해당 시대엔 LTS 개념 없음

## 시대적 배경
Java SE 6(2006) 이후 무려 약 5년의 공백이 있었다. Sun의 경영난, JCP 내부 갈등, 그리고 2009~2010년 Oracle의 Sun 인수 절차가 겹치며 Java 7의 출시가 크게 지연되었다. Oracle은 인수 후 "기능을 다 채우고 늦게 내기보다, 준비된 것부터 내자"는 **Plan B**를 채택해 람다·모듈 같은 대형 기능은 Java 8로 미루고, 완성된 기능들로 Java 7을 먼저 출시했다.

언어 측면에서는 Joshua Bloch가 제안한 **Project Coin** — 큰 부담 없이 일상 코드를 간결하게 만드는 작은 문법 개선들의 묶음(JSR 334) — 이 핵심이었다.

## 주요 추가 기능

### try-with-resources (Project Coin, JSR 334)
`AutoCloseable`을 구현한 자원을 try 괄호에서 선언하면 블록 종료 시 자동으로 `close()`가 호출된다. 자원 누수와 장황한 finally를 제거한다.

도입 전:
```java
BufferedReader br = new BufferedReader(new FileReader("a.txt"));
try {
    return br.readLine();
} finally {
    if (br != null) br.close(); // 수동 닫기, 예외 처리 번거로움
}
```

도입 후:
```java
try (BufferedReader br = new BufferedReader(new FileReader("a.txt"))) {
    return br.readLine();
} // 자동으로 br.close() 호출 (예외가 나도 보장)
```

### 다이아몬드 연산자 (Diamond Operator, `<>`)
제네릭 인스턴스 생성 시 우변의 타입 인자를 생략할 수 있다.

도입 전:
```java
Map<String, List<Integer>> map = new HashMap<String, List<Integer>>();
```

도입 후:
```java
Map<String, List<Integer>> map = new HashMap<>(); // 타입 추론으로 간결
```

### switch문에서 String 사용
도입 전(정수/enum만 가능)에는 if-else 체인을 써야 했다.

도입 후:
```java
String cmd = "start";
switch (cmd) {
    case "start" -> System.out.println("시작");
    case "stop"  -> System.out.println("정지");
    default      -> System.out.println("알 수 없음");
}
```

### 멀티 catch (Multi-catch)
여러 예외를 하나의 catch 블록에서 처리한다.

도입 전:
```java
try {
    doWork();
} catch (IOException e) {
    log(e);
} catch (SQLException e) {
    log(e); // 동일 처리인데 중복
}
```

도입 후:
```java
try {
    doWork();
} catch (IOException | SQLException e) {
    log(e); // 한 번에 처리
}
```

### 숫자 리터럴 개선 (언더스코어 / 바이너리 리터럴)
가독성을 위해 숫자에 `_`를 넣을 수 있고, `0b` 접두사로 2진수 리터럴을 쓸 수 있다.

```java
int million   = 1_000_000;       // 언더스코어로 자릿수 구분
long card     = 1234_5678_9012L;
int binary    = 0b1010_0001;     // 바이너리 리터럴
```

### NIO.2 — 새 파일 시스템 API (JSR 203)
`java.nio.file` 패키지로 파일 I/O가 현대화되었다. `Path`/`Files`, 심볼릭 링크, 파일 속성, 디렉터리 변경 감시(`WatchService`)를 지원한다.

도입 전:
```java
File file = new File("data.txt");
boolean ok = file.delete(); // 실패 사유를 알기 어려움
```

도입 후:
```java
import java.nio.file.*;

Path path = Paths.get("data.txt");
Files.delete(path); // 실패 시 구체적 예외(NoSuchFileException 등)
List<String> lines = Files.readAllLines(path);
Files.copy(path, Paths.get("backup.txt"));
```

### Fork/Join 프레임워크 (JSR 166y)
분할 정복(divide-and-conquer) 작업을 멀티코어에서 병렬 실행하는 프레임워크(`ForkJoinPool`, `RecursiveTask`)다. 작업 훔치기(work-stealing) 스케줄링을 사용하며, 후일 Java 8 병렬 스트림의 엔진이 된다.

```java
class SumTask extends RecursiveTask<Long> {
    final long[] arr; final int lo, hi;
    SumTask(long[] a, int lo, int hi) { this.arr = a; this.lo = lo; this.hi = hi; }
    protected Long compute() {
        if (hi - lo <= 1000) {
            long s = 0; for (int i = lo; i < hi; i++) s += arr[i]; return s;
        }
        int mid = (lo + hi) >>> 1;
        SumTask left = new SumTask(arr, lo, mid);
        left.fork();
        SumTask right = new SumTask(arr, mid, hi);
        return right.compute() + left.join();
    }
}
long total = new ForkJoinPool().invoke(new SumTask(data, 0, data.length));
```

### invokedynamic (JSR 292)
JVM 바이트코드에 동적 메서드 호출을 위한 새 명령 `invokedynamic`과 `java.lang.invoke`(MethodHandle) API가 추가되었다. 정적 타입에 묶이지 않은 호출을 효율적으로 지원해 JRuby, Groovy 등 JVM 동적 언어의 성능을 끌어올렸고, Java 8 람다 구현의 기반이 되었다.

## 그 외 변경 / API 추가
- G1 (Garbage-First) 가비지 컬렉터 도입 (CMS를 잇는 차세대 GC)
- 동시성 유틸리티 보강: `Phaser`, `ThreadLocalRandom`, `ConcurrentLinkedDeque`
- `java.util.Objects` 유틸리티 클래스 (`requireNonNull`, `equals`, `hash` 등)
- 향상된 타입 추론 및 가변인자 경고(`@SafeVarargs`)
- Unicode 6.0 지원, 새 `Locale`/통화 API 개선
- 클라이언트 측 RIA(Rich Internet Application) 관련 개선 (Java FX 별도 발전)

## 영향과 의의
Java SE 7은 Oracle 체제에서 Java가 정상적으로 발전을 재개했음을 보여준 릴리스다. Project Coin의 작은 문법 개선들(try-with-resources, diamond, multi-catch, switch-on-String)은 일상 코드의 장황함을 눈에 띄게 줄였고, NIO.2는 노후한 `java.io.File` API를 대체했다. 특히 Fork/Join과 invokedynamic은 그 자체로도 유용했지만, 1년 뒤 Java 역사상 또 한 번의 분수령이 되는 **Java 8의 람다와 스트림**을 떠받치는 인프라로서 더 큰 의미를 가진다.

## 참고 출처
- [Java version history — Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [JSR 336: Java SE 7 Release Contents — JCP](https://jcp.org/en/jsr/detail?id=336)
- [Java 7 — WikiChip](https://en.wikichip.org/wiki/Java_7)
- [JSR 334: Small Enhancements to the Java Programming Language (Project Coin) — JCP](https://jcp.org/en/jsr/detail?id=334)
