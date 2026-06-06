# Java 변천사 — JDK 1.0부터 Java 25까지

> 자바 언어와 플랫폼의 역사를 버전별로 정리한 "책". 각 장(章)은 한 버전이며, 어떤 기능이 언제 추가되었는지, 그 배경과 의의를 다룬다.

---

## 이 책의 구성

각 버전은 독립된 마크다운 문서로 작성되어 있다. 문서마다 **릴리스 정보 → 시대적 배경 → 주요 추가 기능(코드 예시 포함) → 그 외 변경 → 영향과 의의 → 참고 출처** 구조를 따른다.

---

## 전체 타임라인

| 버전 | 출시 | 코드네임 | LTS | 한 줄 요약 | 문서 |
|------|------|----------|:---:|------------|------|
| JDK 1.0 | 1996-01 | — | | 자바 최초 정식 릴리스, JVM/바이트코드 모델 확립, AWT·애플릿 | [jdk-1.0.md](jdk-1.0.md) |
| JDK 1.1 | 1997-02 | — | | 내부 클래스, JavaBeans, JDBC, RMI, 리플렉션, 위임 이벤트 모델 | [jdk-1.1.md](jdk-1.1.md) |
| J2SE 1.2 | 1998-12 | Playground | | "Java 2" 브랜딩, Swing, Collections Framework, JIT, 플랫폼 분화 | [jdk-1.2.md](jdk-1.2.md) |
| J2SE 1.3 | 2000-05 | Kestrel | | HotSpot JVM 기본 탑재, JNDI, JPDA | [jdk-1.3.md](jdk-1.3.md) |
| J2SE 1.4 | 2002-02 | Merlin | | assert, NIO, 정규식, logging, XML(JAXP), 예외 체이닝 | [jdk-1.4.md](jdk-1.4.md) |
| J2SE 5.0 | 2004-09 | Tiger | | 제네릭·어노테이션·enum·for-each·오토박싱 — 언어 대변혁 | [java-5.md](java-5.md) |
| Java SE 6 | 2006-12 | Mustang | | 성능 중심, 스크립팅 API, 컴파일러 API, 웹서비스 스택 내장 | [java-6.md](java-6.md) |
| Java SE 7 | 2011-07 | Dolphin | | Project Coin(try-with-resources, diamond, multi-catch), NIO.2 | [java-7.md](java-7.md) |
| Java SE 8 | 2014-03 | — | | **람다·Stream·Optional·java.time — 함수형 혁명** | [java-8.md](java-8.md) |
| Java SE 9 | 2017-09 | — | | 모듈 시스템(Jigsaw), jshell, 6개월 릴리스 모델 전환 | [java-9.md](java-9.md) |
| Java SE 10 | 2018-03 | — | | `var` 지역변수 타입 추론 | [java-10.md](java-10.md) |
| Java SE 11 | 2018-09 | — | ✅ | 표준 HTTP 클라이언트, 단일 파일 실행, 첫 6개월모델 LTS | [java-11.md](java-11.md) |
| Java SE 12 | 2019-03 | — | | switch 표현식(preview), Shenandoah GC | [java-12.md](java-12.md) |
| Java SE 13 | 2019-09 | — | | 텍스트 블록(preview), switch 표현식 `yield` | [java-13.md](java-13.md) |
| Java SE 14 | 2020-03 | — | | switch 표현식 정식, record(preview), 유용한 NPE 메시지 | [java-14.md](java-14.md) |
| Java SE 15 | 2020-09 | — | | 텍스트 블록 정식, sealed 클래스(preview), Nashorn 제거 | [java-15.md](java-15.md) |
| Java SE 16 | 2021-03 | — | | record 정식, instanceof 패턴 매칭 정식 | [java-16.md](java-16.md) |
| Java SE 17 | 2021-09 | — | ✅ | sealed 클래스 정식, switch 패턴 매칭(preview) | [java-17.md](java-17.md) |
| Java SE 18 | 2022-03 | — | | UTF-8 기본 charset, jwebserver | [java-18.md](java-18.md) |
| Java SE 19 | 2022-09 | — | | **가상 스레드(preview)**, record 패턴(preview), 구조적 동시성 | [java-19.md](java-19.md) |
| Java SE 20 | 2023-03 | — | | 가상 스레드·record 패턴·switch 패턴 매칭 추가 preview | [java-20.md](java-20.md) |
| Java SE 21 | 2023-09 | — | ✅ | **가상 스레드 정식, record 패턴 정식, switch 패턴 매칭 정식** | [java-21.md](java-21.md) |
| Java SE 22 | 2024-03 | — | | FFM API 정식, 미명명 변수/패턴 정식, Stream Gatherers(preview) | [java-22.md](java-22.md) |
| Java SE 23 | 2024-09 | — | | 마크다운 Javadoc, 세대별 ZGC 기본화, String Templates 철회 | [java-23.md](java-23.md) |
| Java SE 24 | 2025-03 | — | | 역대 최다(24 JEP), Class-File API·Stream Gatherers 정식, 양자내성 암호 | [java-24.md](java-24.md) |
| Java SE 25 | 2025-09 | — | ✅ | Scoped Values·Module Import·Compact Source Files·Flexible Constructors 정식 | [java-25.md](java-25.md) |

> LTS(Long-Term Support): Java 11, 17, 21, 25 (Java 8도 사실상 장기 지원). 6개월 케이던스 시대에는 짝수 9월 릴리스(11, 17, 21, 25...)가 LTS다.

---

## 시대 구분으로 읽기

### 1부. 태동기 (1996~2002) — JDK 1.0 ~ 1.4
자바의 기본 골격이 잡힌 시기. JVM/바이트코드, AWT/Swing GUI, Collections, NIO 등 플랫폼의 토대가 마련됐다. 언어 문법은 거의 고정적이었다.

### 2부. 현대 자바의 기반 (2004~2011) — Java 5, 6, 7
**Java 5의 제네릭·어노테이션**이 언어를 바꿨고, 이는 Spring·Hibernate·JUnit 등 어노테이션 기반 생태계의 토대가 됐다. Java 7은 Oracle 인수 후 첫 릴리스로 Project Coin과 NIO.2를 도입했다.

### 3부. 함수형 혁명과 모듈화 (2014~2017) — Java 8, 9
**Java 8의 람다·Stream**은 자바 작성 방식을 근본적으로 바꿨다. Java 9는 모듈 시스템과 함께 6개월 릴리스 케이던스 시대를 열었다.

### 4부. 빠른 진화 (2018~2021) — Java 10 ~ 17
6개월마다 릴리스되며 `var`, switch 표현식, 텍스트 블록, record, sealed class, 패턴 매칭이 preview → 정식 경로를 밟았다. Java 11·17이 LTS.

### 5부. Loom과 패턴 매칭의 완성 (2022~2025) — Java 18 ~ 25
**가상 스레드(Project Loom)**, record 패턴, switch 패턴 매칭이 정식화(Java 21)되고, FFM API(Project Panama), Scoped Values, 모듈 임포트 등이 자리잡았다. Java 21·25가 LTS.

---

## 릴리스 모델의 변천

| 시기 | 모델 | 특징 |
|------|------|------|
| 1996~2017 (JDK 1.0 ~ Java 9) | 비정기 (수년 간격) | 기능이 모일 때마다 릴리스. Java 6→7 사이 약 5년 공백 |
| 2017~ (Java 10 이후) | **6개월 정기 케이던스** | 매년 3월/9월 릴리스. 기능은 preview → 정식으로 점진 도입 |
| 2018~ | **LTS 도입** | 처음엔 3년 주기(11, 17), 2023부터 2년 주기(21, 25)로 단축 |

- **개발 주체:** JDK 1.0~1.4 = Sun Microsystems / Java 5~6 = Sun (JCP 주도) / Java 7~ = Oracle (2010년 Sun 인수)
- **변경 절차:** Java 5 이전은 비공식 → Java 1.4부터 JCP(JSR) → Java 9 전후부터 **JEP**(JDK Enhancement Proposal)가 중심

---

## 용어 메모

- **JSR** (Java Specification Request): JCP의 표준 명세 단위. 예) 제네릭=JSR 14, 람다=JSR 335.
- **JEP** (JDK Enhancement Proposal): OpenJDK의 기능 제안 단위. 예) `var`=JEP 286, 가상 스레드=JEP 444.
- **preview / incubator / experimental:** 정식 채택 전 단계. preview는 언어/API 기능, incubator는 모듈 단위 API, experimental은 주로 JVM/GC 기능을 시험 배포한다.
