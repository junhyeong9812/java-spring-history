# Java 17 (2021년 9월) — LTS

> Java 11 이후 3년 만의 LTS. 여러 버전에 걸쳐 다듬어 온 모던 문법(record·sealed·패턴 매칭)을 한데 모아 안정적으로 제공하는, 사실상 "모던 Java의 새 기준점" 릴리스.

## 릴리스 정보
- 정식 출시일: 2021년 9월 14일
- LTS 여부: **예 (Long-Term Support)**. 직전 LTS는 Java 11(2018), 다음 LTS는 Java 21(2023).
- 지원: Oracle 및 주요 벤더가 수년간 장기 지원·보안 패치를 제공하는 기준 버전.

## 시대적 배경

Java 17은 Java 11 이후 3년 만에 등장한 LTS다. 6개월 주기 릴리스(12~16)가 차곡차곡 쌓아 올린 언어 기능들이 17에서 정식 형태로 수렴했다는 점이 핵심이다. switch 표현식(14), 텍스트 블록(15), record(16), instanceof 패턴 매칭(16), sealed 클래스(17)까지 — 모던 Java의 핵심 문법 대부분이 17 시점에 정식 기능으로 완비됐다.

그래서 많은 기업과 프레임워크가 Java 8/11에서 곧장 17로 점프하는 마이그레이션을 택했다. Spring Framework 6 / Spring Boot 3가 **Java 17을 최소 요구 버전**으로 채택하면서, 17은 새 시대 엔터프라이즈 Java의 기준선이 됐다.

이 버전은 LTS인 만큼, 새 기능의 양 자체는 많지 않지만 **장기 운영에 필요한 안정성·보안·정리 작업**에 무게를 둔 것이 특징이다.

## 주요 추가 기능

### sealed 클래스 정식화 (JEP 409)

15(1차)·16(2차) preview를 거쳐 **정식 기능**으로 확정됐다. 클래스·인터페이스의 상속/구현 대상을 `permits`로 제한해, 타입 계층을 설계자가 닫힌 집합으로 통제할 수 있다. record·패턴 매칭과 결합하면 대수적 데이터 타입(ADT) 스타일을 표현할 수 있다.

```java
public sealed interface Shape
        permits Circle, Rectangle { }

public record Circle(double radius)              implements Shape { }
public record Rectangle(double w, double h)      implements Shape { }
```

봉인 타입의 하위 타입은 다음 셋 중 하나여야 한다:
- `final` — 더 이상 확장 불가
- `sealed` — 다시 제한된 확장 허용
- `non-sealed` — 봉인을 풀어 자유 확장 허용

### switch 패턴 매칭 (JEP 406, preview)

switch에서 **타입 패턴**으로 분기할 수 있게 하는 기능이 preview로 등장했다. sealed 타입과 결합하면, 컴파일러가 모든 경우를 다뤘는지(exhaustiveness) 검사해줘 안전한 분기를 작성할 수 있다. (정식화는 Java 21의 JEP 441에서 이뤄진다.)

```java
// preview 기능 (--enable-preview 필요)
static double area(Shape shape) {
    return switch (shape) {
        case Circle c    -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.w() * r.h();
        // Shape가 sealed이고 모든 하위 타입을 다뤘다면 default 불필요
    };
}
```

null 처리와 가드(`when`/guarded pattern)도 함께 실험됐다.

```java
static String describe(Object obj) {
    return switch (obj) {
        case null      -> "널";
        case Integer i -> "정수 " + i;
        case String s  -> "문자열 길이 " + s.length();
        default        -> "기타";
    };
}
```

### 새 의사난수 생성기 API (JEP 356, 정식)

`RandomGenerator` 인터페이스를 새로 도입하고, 다양한 PRNG 알고리즘(예: Xoshiro, LXM 계열)을 통합된 API로 제공한다. 기존 `Random`/`SplittableRandom`/`ThreadLocalRandom`을 이 인터페이스 아래로 통합해, 알고리즘을 이름으로 선택하고 일관되게 사용할 수 있다. 스트림 기반 생성과 점프(jump)·분할(split) 기능도 표준화했다.

```java
RandomGenerator gen = RandomGenerator.of("Xoshiro256PlusPlus");
int n = gen.nextInt(100);

// 팩토리로 알고리즘 탐색·생성
RandomGeneratorFactory.of("L64X128MixRandom")
        .create()
        .ints(5)
        .forEach(System.out::println);
```

### 강한 캡슐화의 완성 (JEP 403)

16에서 기본값으로 전환했던 JDK 내부 캡슐화를 한 단계 더 밀어붙여, `--illegal-access` 옵션으로 캡슐화를 **풀 수 있는 길 자체를 없앴다**(`sun.misc.Unsafe` 등 일부 중요한 내부 API는 여전히 접근 가능). 내부 API에 의존하던 코드는 표준 대안으로의 이전이 사실상 강제됐다.

### macOS/AArch64 포트 (JEP 391)

Apple Silicon(M1 등 AArch64 기반 Mac)을 네이티브로 지원. Apple이 자체 칩으로 전환하던 시점에 맞춰 Java가 신속히 대응했다.

### 새 macOS 렌더링 파이프라인 (JEP 382)

Java 2D 내부 렌더링을, 2018년부터 deprecated된 Apple OpenGL 대신 **Apple Metal API** 기반으로 다시 구현했다.

## 그 외 변경

- **Foreign Function & Memory API (JEP 412, incubator)**: 16의 외부 메모리(JEP 393)와 외부 링커(JEP 389) API를 하나로 통합한 incubator. 네이티브 코드 호출과 힙 밖 메모리 접근을 JNI보다 안전·간결하게 다룬다. (Java 22의 JEP 454에서 정식화.)
- **Vector API 2차 incubator (JEP 414)**: 16의 1차 incubator(JEP 338)를 개선.
- **항상 엄격한 부동소수점 (JEP 306)**: `strictfp` 의미를 기본으로 되돌려, 모든 플랫폼에서 동일한 부동소수점 결과를 보장.
- **컨텍스트별 역직렬화 필터 (JEP 415)**: 역직렬화 필터를 컨텍스트 단위로 동적 선택·구성해 보안을 강화.

### 지원 중단(deprecation)·제거

- **Applet API 지원 중단(제거 예정) (JEP 398)**: 브라우저 플러그인 시대의 유물인 `java.applet.Applet`을 제거 예정으로 표시. 대부분의 브라우저가 이미 플러그인 지원을 끊은 상황을 반영.
- **Security Manager 지원 중단(제거 예정) (JEP 411)**: `SecurityManager`와 관련 API 전반을 제거 예정으로 표시. 25년 가까이 쓰였지만 실제 활용도가 낮고 유지 비용이 컸던 권한 기반 보안 모델을 단계적으로 퇴출하기로 했다.
- **실험적 AOT/JIT 컴파일러 제거 (JEP 410)**: GraalVM 기반 실험적 AOT·JIT 컴파일러(jaotc 등)를 JDK에서 제거. AOT 컴파일은 별도 GraalVM 프로젝트로 일원화.
- **RMI Activation 제거 (JEP 407)**: 15에서 지원 중단했던 RMI Activation 메커니즘을 완전히 제거.

## 영향과 의의

Java 17은 단순한 6개월 릴리스가 아니라 **모던 Java의 새 표준선**이다.

1. **문법의 완성**: switch 표현식·텍스트 블록·record·instanceof 패턴 매칭·sealed가 모두 정식 기능으로 모이면서, 12~16에 걸친 점진적 진화가 17에서 하나의 일관된 모습으로 수렴했다. 여기에 switch 패턴 매칭(preview)이 더해져, record + sealed + switch 패턴 매칭이라는 데이터 중심 프로그래밍의 큰 그림이 윤곽을 드러냈다.

2. **마이그레이션의 기준점**: 8/11에 머물던 수많은 프로젝트가 17로 이동했다. Spring Boot 3가 Java 17을 최소 버전으로 요구하면서, 17은 엔터프라이즈 생태계의 사실상 기본값이 됐다.

3. **장기 운영을 위한 정리**: Applet·Security Manager·RMI Activation·실험적 AOT 제거, 강한 캡슐화 완성 등 "오래된 짐을 덜어내는" 작업이 집중됐다. LTS답게, 향후 수년의 안정적 운영을 위한 토대를 다졌다.

4. **현대 하드웨어 대응**: Apple Silicon 네이티브 포트와 Metal 렌더링 파이프라인으로, 변화하는 하드웨어 환경에 발 빠르게 적응했다.

종합하면 Java 17은 "모던 Java를 한 번에 받아들이기 좋은 안정적 LTS"이며, 이후 Java 21 LTS로 이어지는 진화의 든든한 중간 거점이다.

## 참고 출처
- [OpenJDK: JDK 17](https://openjdk.org/projects/jdk/17/)
- [JEPs in JDK 17 integrated since JDK 11](https://openjdk.org/projects/jdk/17/jeps-since-jdk-11)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
- [JEP 406: Pattern Matching for switch (Preview)](https://openjdk.org/jeps/406)
- [JEP 356: Enhanced Pseudo-Random Number Generators](https://openjdk.org/jeps/356)
- [JEP 411: Deprecate the Security Manager for Removal](https://openjdk.org/jeps/411)
- [JEP 398: Deprecate the Applet API for Removal](https://openjdk.org/jeps/398)
- [JEP 412: Foreign Function & Memory API (Incubator)](https://openjdk.org/jeps/412)
- [New Features in Java 17 - Baeldung](https://www.baeldung.com/java-17-new-features)
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
