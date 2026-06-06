# Spring 변천사 — Framework / Boot / Kotlin

> Spring Framework와 Spring Boot의 역사를 대버전별로 정리한 "책". 설정 방식의 변천(XML → 어노테이션 → Java Config → 함수형 DSL)과 Kotlin의 도입사를 함께 다룬다.

---

## 이 책의 구성

Spring은 두 축으로 진화했다.
- **Spring Framework** — IoC/DI 컨테이너를 핵심으로 한 본체 (1.x ~ 7.x. 본 문서는 6.x까지 상세, 7.0은 최신 동향으로 표기)
- **Spring Boot** — 자동 설정으로 Spring 사용을 간소화한 상위 레이어 (1.x ~ 4.x. 본 문서는 3.x까지 상세, 4.0은 최신 동향으로 표기)

여기에 사용자가 특별히 요청한 **Kotlin과 Spring의 통합사**를 별도 문서로 정리했다.

---

## 전체 타임라인

| 시기 | Spring Framework | Spring Boot | 핵심 전환점 |
|------|------------------|-------------|-------------|
| 2004 | **1.0** (3월) | — | IoC/DI 컨테이너, AOP, XML 설정, EJB 대안 |
| 2006~07 | **2.0**(10월) / 2.5(07.11) | — | XML 네임스페이스, 어노테이션(`@Autowired`/`@Component`) 도입 |
| 2009~12 | **3.0**(12월) / 3.1 / 3.2 | — | Java 5 baseline, **Java Config**, SpEL, REST, `@Profile` |
| 2013~16 | **4.0**(13.12) / 4.3 | **1.0**(14.04) | Java 8 지원, `@RestController` / Boot 등장(자동 설정·스타터) |
| 2017~20 | **5.0**(17.09) / 5.2 / 5.3 | **2.0**(18.03) | **리액티브(WebFlux) + Kotlin 1급 지원** / Boot 2.0 Kotlin 정식 지원, 5.2 코루틴 |
| 2022~ | **6.0**(22.11) / 6.1 / 6.2 | **3.0**(22.11) / 3.1~3.4 | **Java 17 + Jakarta EE 9(javax→jakarta)**, GraalVM 네이티브/AOT, 가상 스레드 |
| 2025~ | **7.0**(25.11.13 GA) | **4.0**(25.11.20 GA) | **현재 최신 세대** — Java 17 baseline(Java 25 수용), Jakarta EE 11, JSpecify 널 안정성, API 버저닝, Boot 코드베이스 모듈화 |

> 본 문서의 상세 설명은 Framework 6.x / Boot 3.x까지 다룬다. 7.0/4.0은 위 타임라인에 최신 동향으로만 표기하며 별도 상세 문서는 두지 않는다.

---

## 문서 목차

### Spring Framework (본체)
| 대버전 | 최초 출시 | 최소 자바 | 핵심 | 문서 |
|--------|-----------|-----------|------|------|
| 1.x | 2004-03 | Java 1.3 | IoC/DI, AOP, XML 설정 | [framework-1.x.md](framework-1.x.md) |
| 2.x | 2006-10 | Java 1.3+ (2.5는 1.4.2+) | XML 네임스페이스, 어노테이션 시작 | [framework-2.x.md](framework-2.x.md) |
| 3.x | 2009-12 | Java 5 | Java Config, SpEL, REST | [framework-3.x.md](framework-3.x.md) |
| 4.x | 2013-12 | Java 6 (Java 8은 지원 기능) | Java 8 지원, `@RestController`, WebSocket | [framework-4.x.md](framework-4.x.md) |
| 5.x | 2017-09 | Java 8 | **리액티브(WebFlux) + Kotlin 1급** | [framework-5.x.md](framework-5.x.md) |
| 6.x | 2022-11 | Java 17 | **Jakarta EE 9+, AOT/네이티브** | [framework-6.x.md](framework-6.x.md) |

### Spring Boot (자동 설정 레이어)
| 대버전 | 최초 출시 | 기반 Framework | 최소 자바 | 핵심 | 문서 |
|--------|-----------|----------------|-----------|------|------|
| 1.x | 2014-04 | 4.x | Java 6 | 자동 설정, 스타터, 내장 톰캣, Actuator | [boot-1.x.md](boot-1.x.md) |
| 2.x | 2018-03 | 5.x | Java 8 | WebFlux, Micrometer, **Kotlin 공식 지원** | [boot-2.x.md](boot-2.x.md) |
| 3.x | 2022-11 | 6.x | Java 17 | Jakarta, GraalVM 네이티브, 가상 스레드 | [boot-3.x.md](boot-3.x.md) |

### Kotlin
- [kotlin-and-spring.md](kotlin-and-spring.md) — 코틀린이 언제·어떻게 Spring에 들어왔는가, 자바와 코틀린의 공존 방식

---

## 설정 스타일의 변천 (한눈에)

```
XML BeanFactory     어노테이션         Java Config        Boot 자동설정              함수형/Kotlin DSL
(2004, 1.x)    →    (2007, 2.5)   →    (2009, 3.0)   →    (2014, Boot)         →    (2017, Framework 5.0)
<bean .../>         @Component         @Configuration     @SpringBootApplication    beans { } / router { }
config.xml          @Autowired         @Bean             (auto-configuration)
```

자바 진영의 흐름:
1. **XML 시대 (1.x~2.x)** — 모든 빈을 XML로 선언. 장황하지만 설정과 코드 분리.
2. **어노테이션 시대 (2.5~)** — `@Component`/`@Autowired`로 컴포넌트 스캔. XML 대폭 축소.
3. **Java Config 시대 (3.0~)** — `@Configuration`/`@Bean`으로 타입 안전한 설정.
4. **Boot 시대 (2014~)** — 자동 설정(`auto-configuration`)으로 설정 자체를 최소화.
5. **함수형/Kotlin DSL (2017~)** — `beans { }`, `router { }`로 코드로서의 설정.

---

## Kotlin 도입 타임라인 (요약)

| 시기 | 사건 |
|------|------|
| 2011-07 | JetBrains, Kotlin 공개 |
| 2016-02 | Kotlin 1.0 출시 |
| 2016 (경) | start.spring.io(프로젝트 생성기)에 Kotlin 옵션 실험적 추가 |
| 2017-05 | Google, 안드로이드 1급 언어로 Kotlin 채택 |
| **2017-09** | **Spring Framework 5.0 — Kotlin 1급 지원 정식 포함** (널 안정성, 확장 함수, `beans`/`router` DSL) |
| **2018-03** | **Spring Boot 2.0 — Kotlin 정식 지원** (컴파일러 플러그인·스타터, start.spring.io 정식 옵션) |
| **2019-09** | **Spring Framework 5.2 — 코루틴(`suspend`/`Flow`) 정식 지원** |

자세한 내용과 코드 예시(자바 vs 코틀린 비교)는 [kotlin-and-spring.md](kotlin-and-spring.md) 참조.
