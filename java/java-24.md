# Java 24 (2025.03)

> 무려 24개의 JEP를 담은 대형 비-LTS 릴리스. 클래스 파일 API와 스트림 Gatherers를 정식화하고, 양자내성 암호(ML-KEM·ML-DSA)와 AOT 클래스 로딩을 도입했으며, 32비트 x86 포트를 정리하기 시작했다.

## 릴리스 정보
- 정식 출시일: 2025년 3월 18일
- LTS 여부: 아니오 (단기 지원 / Java 25 LTS로 대체)
- 포함 JEP 목록 (총 24개):
  - JEP 404: Generational Shenandoah (실험적)
  - JEP 450: Compact Object Headers (실험적)
  - JEP 472: Prepare to Restrict the Use of JNI
  - JEP 475: Late Barrier Expansion for G1
  - JEP 478: Key Derivation Function API (프리뷰)
  - JEP 479: Remove the Windows 32-bit x86 Port (정식)
  - JEP 483: Ahead-of-Time Class Loading & Linking (정식)
  - JEP 484: Class-File API (정식)
  - JEP 485: Stream Gatherers (정식)
  - JEP 486: Permanently Disable the Security Manager (정식)
  - JEP 487: Scoped Values (4차 프리뷰)
  - JEP 488: Primitive Types in Patterns, instanceof, and switch (2차 프리뷰)
  - JEP 489: Vector API (9차 인큐베이터)
  - JEP 490: ZGC: Remove the Non-Generational Mode (정식)
  - JEP 491: Synchronize Virtual Threads without Pinning (정식)
  - JEP 492: Flexible Constructor Bodies (3차 프리뷰)
  - JEP 493: Linking Run-Time Images without JMODs (정식)
  - JEP 494: Module Import Declarations (2차 프리뷰)
  - JEP 495: Simple Source Files and Instance Main Methods (4차 프리뷰)
  - JEP 496: Quantum-Resistant Module-Lattice-Based Key Encapsulation Mechanism (정식)
  - JEP 497: Quantum-Resistant Module-Lattice-Based Digital Signature Algorithm (정식)
  - JEP 498: Warn upon Use of Memory-Access Methods in sun.misc.Unsafe
  - JEP 499: Structured Concurrency (4차 프리뷰)
  - JEP 501: Deprecate the 32-bit x86 Port for Removal

## 시대적 배경

Java 25 LTS 직전의 마지막 비-LTS 릴리스로, JEP 24개라는 역대급 분량을 담았다. LTS를 6개월 앞두고 여러 기능을 정식화·안정화하여 LTS의 완성도를 끌어올리려는 의도가 뚜렷하다. 시대적으로는 **양자컴퓨팅에 대비한 포스트양자 암호(PQC)** 의 표준 도입(ML-KEM, ML-DSA)이 가장 상징적이다. 미국 NIST가 2024년 PQC 표준을 확정한 직후 JDK에 곧바로 반영된 것으로, 자바 플랫폼의 보안 대응 속도를 보여준다. 또한 가상 스레드의 핀닝(pinning) 문제 해결, AOT 클래스 로딩(Project Leyden의 첫 결실) 등 운영 성능 개선도 두드러진다.

## 주요 추가 기능

### Class-File API (JEP 484, 정식)
- 클래스 파일을 파싱·생성·변환하는 표준 API가 정식화되었다(22 프리뷰 → 23 2차 프리뷰 → 24 정식). JDK가 내부적으로 두고 있던 ASM 사본을 이 표준 API로 대체할 수 있게 하는 것이 목표이며, 외부 ASM 등 서드파티 바이트코드 라이브러리를 폐기 대상으로 삼는 것은 목표가 아니다.

```java
import java.lang.classfile.*;

ClassModel cm = ClassFile.of().parse(bytes);
for (MethodModel m : cm.methods()) {
    System.out.println(m.methodName().stringValue());
}
```

### Stream Gatherers (JEP 485, 정식)
- 사용자 정의 중간 연산 API가 정식화되었다(22 프리뷰 → 23 2차 프리뷰 → 24 정식). `Stream.gather(...)`와 `Gatherers` 팩토리를 표준으로 사용할 수 있다.

```java
List<List<Integer>> windows = Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.windowSliding(2))
    .toList(); // [[1, 2], [2, 3], [3, 4], [4, 5]]
```

### 양자내성 암호 — ML-KEM (JEP 496) / ML-DSA (JEP 497, 정식)
- NIST가 표준화한 격자 기반 포스트양자 암호 알고리즘을 JDK 표준 API로 제공한다. **ML-KEM**(FIPS 203, 키 캡슐화)과 **ML-DSA**(FIPS 204, 디지털 서명)이며, 양자컴퓨터 등장 시에도 안전한 키 교환·서명을 가능케 한다.

```java
// ML-KEM 키 쌍 생성 (키 캡슐화용)
KeyPairGenerator kemKpg = KeyPairGenerator.getInstance("ML-KEM");
KeyPair kemKp = kemKpg.generateKeyPair();

// ML-DSA 키 쌍 생성 후 디지털 서명
KeyPairGenerator dsaKpg = KeyPairGenerator.getInstance("ML-DSA");
KeyPair dsaKp = dsaKpg.generateKeyPair();

Signature sig = Signature.getInstance("ML-DSA");
sig.initSign(dsaKp.getPrivate());
sig.update("hello".getBytes());
byte[] signature = sig.sign();
```

### Ahead-of-Time Class Loading & Linking (JEP 483, 정식)
- 애플리케이션이 사용할 클래스를 미리 로드·링크한 캐시를 만들어 시작 시간을 크게 단축한다. **Project Leyden**의 첫 정식 산출물로, 특히 단명(short-lived) 프로그램과 마이크로서비스 콜드 스타트에 효과적이다.

```bash
# 1) 학습 단계: AOT 설정 기록
java -XX:AOTMode=record -XX:AOTConfiguration=app.aotconf -cp app.jar App
# 2) 생성 단계: AOT 캐시 생성
java -XX:AOTMode=create -XX:AOTConfiguration=app.aotconf -XX:AOTCache=app.aot -cp app.jar
# 3) 실행: 캐시 사용
java -XX:AOTCache=app.aot -cp app.jar App
```

### Synchronize Virtual Threads without Pinning (JEP 491, 정식)
- 가상 스레드가 `synchronized` 블록/메서드 안에서 블로킹될 때 캐리어 스레드에 고정(pinning)되던 문제를 해결한다. 이제 동기화 코드 안에서도 가상 스레드가 자유롭게 언마운트되어, Loom의 확장성 제약이 크게 완화되었다.

### ZGC: Remove the Non-Generational Mode (JEP 490, 정식)
- Java 23에서 세대별 ZGC가 기본이 된 뒤, 이 버전에서 비-세대 모드를 완전히 제거했다. `-XX:-ZGenerational` 옵션은 무시되고 향후 제거된다.

## 그 외 변경
- **Prepare to Restrict the Use of JNI (JEP 472)**: FFM API 정식화에 이어, JNI 사용 시 경고를 발생시켜 향후 제한을 예고. 네이티브 상호운용을 더 안전한 FFM으로 유도한다.
- **Permanently Disable the Security Manager (JEP 486, 정식)**: 오래전 폐기 예고된 SecurityManager를 영구 비활성화(런타임에서 활성화 불가).
- **Key Derivation Function API (JEP 478, 프리뷰)**: HKDF 등 키 유도 함수 표준 API의 첫 프리뷰. (Java 25에서 정식화됨.)
- **Generational Shenandoah (JEP 404, 실험적)** / **Compact Object Headers (JEP 450, 실험적)**: GC·메모리 풋프린트 개선을 실험 단계로 도입.
- **Late Barrier Expansion for G1 (JEP 475)**: G1 GC의 배리어 코드를 컴파일 후반에 확장해 C2 컴파일 비용을 줄인다.
- **Linking Run-Time Images without JMODs (JEP 493, 정식)**: JMOD 파일 없이 `jlink`로 런타임 이미지를 생성할 수 있게 한다. 단 이는 `--enable-linkable-runtime`으로 빌드한 JDK에서만 가능하며, 기본 빌드는 여전히 JMOD를 포함한다. 이를 통해 JDK 설치 크기를 줄일 수 있다.
- **Remove the Windows 32-bit x86 Port (JEP 479)** / **Deprecate the 32-bit x86 Port for Removal (JEP 501)**: 32비트 x86 지원을 단계적으로 종료(윈도우는 제거, 리눅스는 폐기 예고). Java 25에서 완전 제거된다.
- **Warn upon Use of sun.misc.Unsafe Memory-Access Methods (JEP 498)**: 23에서 폐기 예고한 메서드 사용 시 런타임 경고 발생.
- **계속 진행 중인 프리뷰**: Scoped Values(JEP 487, 4차), Structured Concurrency(JEP 499, 4차), Flexible Constructor Bodies(JEP 492, 3차), Module Import Declarations(JEP 494, 2차), Primitive Types in Patterns(JEP 488, 2차), Simple Source Files and Instance Main Methods(JEP 495, 4차), Vector API(JEP 489, 9차 인큐베이터).

## 영향과 의의

Java 24는 LTS 직전 릴리스답게 "정식화 러시"를 보여준다. 클래스 파일 API·스트림 Gatherers의 정식화는 라이브러리/프레임워크 생태계의 기반을 바꾸었고, 가상 스레드 핀닝 해결은 Loom 채택의 마지막 큰 장애물을 제거했다. 양자내성 암호의 신속한 표준 도입은 자바가 보안 패러다임 전환에 선제 대응함을 보여준다. 여기서 안정화된 기능들이 6개월 뒤 Java 25 LTS에서 그대로 정식 기반이 된다.

## 참고 출처
- [JDK 24 - OpenJDK 프로젝트 페이지](https://openjdk.org/projects/jdk/24/)
- [Java 24 Delivers New Experimental and Many Final Features - InfoQ](https://www.infoq.com/news/2025/03/java24-released/)
- [Java 24 Features (with Examples) - HappyCoders](https://www.happycoders.eu/java/java-24-features/)
- [Six JDK 24 Features You Should Know About - foojay.io](https://foojay.io/today/six-jdk-24-features-you-should-know-about/)
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
