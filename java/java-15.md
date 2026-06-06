# Java 15 (2020년 9월)

> 텍스트 블록을 정식화하고, 봉인 클래스(sealed)를 처음 선보이며, ZGC·Shenandoah를 정식 GC로 승격하고 Nashorn을 제거한 정리·확장의 릴리스.

## 릴리스 정보
- 정식 출시일: 2020년 9월 15일
- LTS 여부: 아니오 (단기 지원 릴리스)

## 시대적 배경

Java 11(LTS, 2018) 이후 6개월 주기 릴리스가 안정적으로 반복되며, 각 버전은 "새 기능 preview 도입 → 다듬기 → 정식화"와 "낡은 기능 정리"를 병행했다. Java 15는 그 양면을 모두 보여준다. 텍스트 블록을 정식 기능으로 끌어올리는 동시에, 더 이상 쓸모가 적어진 Nashorn JavaScript 엔진을 제거했다. 또한 experimental 상태였던 GC(ZGC·Shenandoah)와 14에서 preview였던 언어 기능들을 한 단계씩 더 성숙시켰다.

이 시기 Java 진영은 마이크로서비스·클라우드 환경에서의 저지연 GC 수요에 대응하느라, 실험 단계였던 ZGC와 Shenandoah를 production-ready로 끌어올리는 데 공을 들였다.

## 주요 추가 기능

### 텍스트 블록 (JEP 378, 정식)

13(1차 preview)·14(2차 preview)를 거쳐 정식 기능으로 확정됐다. 여러 줄 문자열을 `"""`로 감싸 JSON·HTML·SQL 같은 내장 텍스트를 escape 지옥 없이 작성할 수 있다.

```java
String json = """
        {
            "name": "Java",
            "version": 15
        }
        """;
```

### sealed 클래스 (JEP 360, preview)

클래스/인터페이스를 **상속·구현할 수 있는 대상을 명시적으로 제한**하는 기능. `permits` 절로 허용 목록을 지정해, 타입 계층을 설계자가 통제할 수 있다. 이 버전에서 처음 preview로 등장했다.

```java
public sealed interface Shape
        permits Circle, Rectangle, Triangle { }

public final class Circle implements Shape { }
public final class Rectangle implements Shape { }
public final class Triangle implements Shape { }
// 위 셋 외에는 Shape를 구현할 수 없다
```

### Hidden Class (JEP 371, 정식)

프레임워크가 런타임에 생성해 쓰는 클래스를, 다른 클래스에서 직접 참조할 수 없는 **숨겨진 클래스**로 정의하는 표준 API. 프록시·람다·동적 언어 구현체 등에 유용하며, 기존의 비표준 `Unsafe.defineAnonymousClass`를 대체한다.

### ZGC 정식화 (JEP 377)

저지연 확장형 가비지 컬렉터 ZGC가 experimental 딱지를 떼고 production 기능이 됐다. 대용량 힙에서도 정지 시간을 밀리초 단위로 유지한다.

### Shenandoah 정식화 (JEP 379)

또 다른 저지연 GC인 Shenandoah도 experimental에서 production 기능으로 승격됐다.

### EdDSA (JEP 339, 정식)

Edwards-Curve Digital Signature Algorithm(Ed25519/Ed448) 서명 알고리즘을 표준 지원. 성능과 보안성이 좋아 널리 쓰이는 현대 서명 방식을 플랫폼에 내장했다.

## 그 외 변경

- **Nashorn JavaScript 엔진 제거 (JEP 372)**: 11에서 deprecated됐던 Nashorn 엔진·API·도구가 완전히 제거됐다.
- **instanceof 패턴 매칭 2차 preview (JEP 375)**: 14의 1차 preview에 이어 변경 없이 한 번 더 preview.
- **record 2차 preview (JEP 384)**: record가 sealed 타입, 지역(local) record 등과 함께 다듬어진 2차 preview.
- **Foreign-Memory Access API 2차 incubator (JEP 383)**.
- **JEP 373**: 레거시 DatagramSocket API를 현대적으로 재구현.
- **JEP 374**: 편향 잠금(biased locking) 비활성화 및 지원 중단.
- **제거/지원 중단**: Solaris/SPARC 포트 제거(JEP 381), RMI Activation 지원 중단(JEP 385).

## 영향과 의의

Java 15는 14에서 뿌린 씨앗을 키운 "징검다리" 릴리스다. 텍스트 블록을 정식화해 일상 코드의 가독성을 끌어올렸고, sealed 클래스를 도입해 record·패턴 매칭과 함께 대수적 데이터 타입(ADT) 스타일과 향후의 switch 패턴 매칭을 위한 기반을 놓았다. ZGC·Shenandoah 정식화는 Java가 클라우드·대용량 서비스의 저지연 요구에 진지하게 대응하고 있음을 보여주는 신호였다. 동시에 Nashorn 제거처럼 군더더기를 걷어내며 플랫폼을 가볍게 유지했다.

## 참고 출처
- [OpenJDK: JDK 15](https://openjdk.org/projects/jdk/15/)
- [JEP 378: Text Blocks](https://openjdk.org/jeps/378)
- [JEP 360: Sealed Classes (Preview)](https://openjdk.org/jeps/360)
- [JEP 371: Hidden Classes](https://openjdk.org/jeps/371)
- [JEP 377: ZGC: A Scalable Low-Latency Garbage Collector](https://openjdk.org/jeps/377)
- [JEP 372: Remove the Nashorn JavaScript Engine](https://openjdk.org/jeps/372)
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
