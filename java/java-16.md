# Java 16 (2021년 3월)

> record와 instanceof 패턴 매칭을 동시에 정식화하고, JDK 내부를 기본적으로 강하게 캡슐화하며, Vector·외부 메모리 API를 incubator로 밀어붙인 "정식화 러시" 릴리스.

## 릴리스 정보
- 정식 출시일: 2021년 3월 16일
- LTS 여부: 아니오 (단기 지원 릴리스, LTS인 17 직전 버전)

## 시대적 배경

Java 16은 다음 LTS(Java 17)를 6개월 앞둔 버전이다. 그동안 여러 차례 preview를 거치며 다듬어 온 핵심 언어 기능들을 정식화해, LTS에서 안정적으로 제공할 채비를 갖추는 성격이 강하다. record와 instanceof 패턴 매칭이 이 버전에서 동시에 정식 기능이 된 것이 그 증거다.

또한 모듈 시스템(Java 9) 이후 오래 끌어온 과제인 **JDK 내부 API 캡슐화**를 이 버전에서 기본 정책으로 전환했다. 빌드 인프라를 Mercurial에서 Git/GitHub로 옮긴 것도 이 시기다.

## 주요 추가 기능

### record 정식화 (JEP 395)

14·15의 두 차례 preview를 거쳐 정식 기능으로 확정됐다. 불변 데이터 운반 객체를 간결하게 선언하며, 생성자·접근자·`equals`/`hashCode`/`toString`이 자동 생성된다.

```java
record Range(int lo, int hi) {
    // 컴팩트 생성자로 검증 로직 추가 가능
    Range {
        if (lo > hi) throw new IllegalArgumentException("lo > hi");
    }
}

var r = new Range(1, 10);
System.out.println(r.lo() + ".." + r.hi()); // 1..10
```

### instanceof 패턴 매칭 정식화 (JEP 394)

14·15의 preview를 거쳐 정식 기능이 됐다. `instanceof` 검사와 캐스팅, 바인딩 변수 선언을 한 번에 처리한다.

```java
// 패턴 변수를 조건식에서 바로 활용 가능
if (obj instanceof String s && s.length() > 5) {
    System.out.println(s.toUpperCase());
}
```

### sealed 클래스 2차 preview (JEP 397)

15의 1차 preview에 이어 다듬어진 2차 preview. 봉인된 타입과 record를 결합하는 미래(switch 패턴 매칭)를 준비하는 단계로, 17에서 정식화된다.

### Vector API (JEP 338, incubator)

SIMD(Single Instruction Multiple Data) 하드웨어 명령으로 벡터 연산을 표현하는 API가 incubator로 처음 등장. CPU의 벡터 유닛을 활용해 수치 계산 성능을 끌어올린다.

```java
// incubator 모듈: jdk.incubator.vector
var va = FloatVector.fromArray(SPECIES, a, 0);
var vb = FloatVector.fromArray(SPECIES, b, 0);
var vc = va.mul(vb);          // 벡터 곱
vc.intoArray(c, 0);
```

### 외부 메모리/링커 API (JEP 393, 389, incubator)

힙 밖 네이티브 메모리에 안전하게 접근하는 **Foreign-Memory Access API**가 3차 incubator(JEP 393)로, 네이티브 함수를 호출하는 **Foreign Linker API**가 incubator(JEP 389)로 제공됐다. 후일 17의 통합 FFM API(JEP 412)로 합쳐진다.

### jpackage 정식화 (JEP 392)

14에서 incubator로 나온 패키징 도구가 정식 기능이 됐다. Java 애플리케이션을 msi·dmg·deb 등 플랫폼별 네이티브 설치 패키지로 묶는다.

## 그 외 변경

- **JDK 내부의 강한 캡슐화 (JEP 396)**: `sun.misc.Unsafe` 등 일부 예외를 빼고, JDK 내부 API에 대한 리플렉션 접근을 **기본적으로 차단**(strong encapsulation)하도록 정책을 전환. `--illegal-access` 기본값이 `permit`에서 `deny`로 바뀌어, 내부 API에 의존하던 레거시 코드의 마이그레이션을 압박했다.
- **JEP 380**: Unix 도메인 소켓 채널 지원.
- **JEP 387**: Elastic Metaspace — 미사용 메타스페이스 메모리를 OS에 더 신속히 반환.
- **JEP 376**: ZGC의 스레드 스택 처리를 동시(concurrent) 단계로 이동해 정지 시간 단축.
- **JEP 390**: 값 기반 클래스(value-based class)에 대한 경고 도입.
- **JEP 347**: C++14 언어 기능을 JDK 소스에 허용.
- **JEP 357**: 소스 코드 관리 시스템을 Mercurial에서 Git으로 이전.
- **이식성**: Alpine Linux 포트(JEP 386), Windows/AArch64 포트(JEP 388).

## 영향과 의의

Java 16은 LTS 직전의 "마지막 다듬기" 릴리스로서 의미가 크다. record와 instanceof 패턴 매칭이라는 두 핵심 문법이 동시에 정식화되면서, 곧 나올 17의 sealed 정식화·switch 패턴 매칭과 맞물려 Java의 데이터 중심·패턴 중심 프로그래밍이 완성형에 다가섰다. JDK 내부의 강한 캡슐화 기본화는 단기적으로 호환성 통증을 유발했지만, 모듈 시스템이 약속한 캡슐화를 현실로 만든 중요한 전환이었다. incubator 단계의 Vector·외부 메모리 API는 Java의 고성능·네이티브 상호운용 로드맵을 예고했다.

## 참고 출처
- [OpenJDK: JDK 16](https://openjdk.org/projects/jdk/16/)
- [JEP 395: Records](https://openjdk.org/jeps/395)
- [JEP 394: Pattern Matching for instanceof](https://openjdk.org/jeps/394)
- [JEP 397: Sealed Classes (Second Preview)](https://openjdk.org/jeps/397)
- [JEP 338: Vector API (Incubator)](https://openjdk.org/jeps/338)
- [JEP 396: Strongly Encapsulate JDK Internals by Default](https://openjdk.org/jeps/396)
- [JEP 392: Packaging Tool](https://openjdk.org/jeps/392)
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
