# 모델 교차 검증 기록 (codex)

문서의 사실관계(날짜·JEP/JSR 번호·preview/정식 상태·기능 귀속)를 외부 LLM `codex`(GPT 계열)로 교차검증한 기록이다. 오케스트레이션 규칙 5.2 / discuss 3.7에 따른다.

## 호출 1

- 호출 ID: `java-spring-history/cross-validation/001`
- 도구: `codex exec --skip-git-repo-check -s read-only --ephemeral`
- 입력 해시(sha256 앞 12자): `9775bf7ef818`
- 입력: 전체 문서의 핵심 사실 주장 요약(버전 타임라인 + JEP/JSR 매핑 + Kotlin 타임라인)
- 보안 게이트: 패턴 스캔 매칭 0건 통과 (공개 역사 사실만, 시크릿/PII 없음)
- 종료 코드: 0 (정상)

### codex 지적 6건과 처리 결과

| # | 지적 | 판정 | 처리 |
|---|------|------|------|
| 1 | JDK 1.1 출시일 출처 충돌(1997-02-19 vs java.com 1997-03-28) | **채택** | `jdk-1.1.md`에 출처별 차이 명시 |
| 2 | J2SE 1.2 "Playground"는 비공식 코드네임 | **채택** | `jdk-1.2.md`에 "일부 출처의 비공식 명칭" 주석 |
| 3 | 1.2의 JIT 기본 탑재 귀속 애매 | **기각** | 문서는 "JIT 기본 포함"만 서술, HotSpot 기본 탑재는 1.3 문서에서 별도 정확히 기술 |
| 4 | Web Start를 1.4 신규 기능처럼 서술 | **채택** | `jdk-1.4.md`에 "2001년 별도 출시 → 1.4에서 번들 통합"으로 정정 |
| 5 | Java 25 JEP 목록에서 518/520 누락 | **기각** | 실제 `java-25.md`에는 18개(518/520 포함) 모두 존재. codex에 보낸 *요약*만 누락된 것 |
| 6 | "Boot 2.0에서 Kotlin 완성" 표현 부정확 | **채택** | `spring/README.md`·`kotlin-and-spring.md`에서 "정식 지원"으로 완화, 코루틴은 5.2에서 추가됨을 명시 |

### 메모
- 지적 5는 codex가 옳게 짚었으나 대상은 "내가 codex에 보낸 요약"이었고 실제 문서는 정확했다 → 실제 파일과 대조해 기각. (요약본만 보고 판단하면 안 된다는 교훈)
- 1차 작성 시 서브 에이전트들은 "환경에 codex 없음"으로 잘못 판단해 교차검증을 스킵했으나, 실제로는 `codex` v0.137.0이 설치되어 있어 사후 검증을 수행했다.

## 호출 2 (정밀 검증 — 파일 본문 전체)

요약이 아니라 **각 문서 본문 전체**를 파일별로 codex에 넣어 정밀 사실검증을 수행했다.

- 호출 ID: `java-spring-history/cross-validation/002`
- 대상: 버전 문서 36개 + 인덱스 2개 = 38개 (파일별 개별 호출, 병렬 6)
- 도구: `codex exec --skip-git-repo-check -s read-only --ephemeral` (codex는 웹 출처 접근 가능 상태)
- 보안 게이트: 통과. 단 `boot-3.x.md`의 코드 예시 `alice@example.com`이 이메일 패턴에 걸렸으나 예약 예시 도메인(example.com) 플레이스홀더로 무해 판정.
- 결과: 3개 파일 `NO_ISSUES`(java-10/13/16), 나머지에서 총 ~85건 지적.

### 처리 요약 (지적 → 거의 전부 채택, 파일별 반영)

대표 사실 정정:
- **Nashorn**: Java 11 "제거" → 실제는 deprecate(JEP 335), 제거는 Java 15(JEP 372).
- **Helpful NPE**: Java 14에서 기본 OFF(`-XX:+ShowCodeDetailsInExceptionMessages` 필요), 기본 활성화는 Java 15.
- **`when` 가드**: Java 17/18 예시에서 잘못 사용 → 해당 시대는 `&&` guarded pattern. `when`은 Java 19 도입, 21 정식.
- **sealed**: Java 21 정식화 목록에서 제거(정식화는 Java 17 JEP 409).
- **Stream Gatherers**: "Java 25 정식" → 실제 Java 24(JEP 485) 정식.
- **레코드 패턴 enhanced-for(JEP 432)**: 방향 반대 서술 정정(헤더 지원 "추가", named record patterns "제거").
- **JDK 15 GA**: 9월 16일 → 9월 15일.
- **Spring Framework 7.0(2025-11-13)/Boot 4.0(2025-11-20)**: 출시 사실을 인덱스 타임라인에 반영(상세는 6.x/3.x까지).
- **Spring 기능 귀속 다수**: `@Repository`(2.0), `@ConstructorBinding`(Boot 2.2), `@ConfigurationProperties` 스캔(2.2), SpEL 컴파일러(4.1), Boot 1.2 GA(2014-12) 등.
- **시대정확 코드 예시**: 구버전 문서들의 람다·제네릭·for-each·diamond·arrow switch 등 후대 문법을, 그 버전에서 실제 동작하는 문법으로 전면 교체(JDK 1.0 resize()/show(), 1.1 익명 내부 클래스, 1.2 raw type + getContentPane().add() 등). 컴파일 불가 예시들도 동작하도록 수정.

기각/보정:
- codex가 "kotlin-spring(all-open)이 `@Entity`를 open으로 만든다"고 전제한 부분은 **틀림** → 웹 재검증 후 "`@Entity`는 all-open 기본 대상이 아니며, `open` 또는 `allOpen` 직접 지정 필요"로 정확히 보정.
- `java-10/13/16`은 지적 없음.

## 호출 3 (다이어그램 검증 — Mermaid 문법 + 기술 정확성)

각 문서에 추가한 Mermaid 다이어그램(총 36개, 19개 파일)을 codex로 검증했다.

- 호출 ID: `java-spring-history/cross-validation/003`
- 대상: 다이어그램이 추가된 19개 파일 (파일별 개별 호출, 병렬 6)
- 점검 항목: ① Mermaid 문법 유효성(렌더 가능 여부) ② 묘사된 의존성/흐름의 기술적 정확성
- 사전 기계 검증: Python으로 펜스 균형·타입 키워드·subgraph/alt-end 균형·특수문자 라벨 따옴표 처리 확인(36블록 통과)
- 결과: **문법 오류 0건**. 14개 파일 `NO_ISSUES`, 5개 파일에서 기술 정확성 7건 지적 → 전부 채택.

### 채택한 정확성 정정
- `java-11.md`: `thenApply` 콜백은 원래 호출 스레드가 아니라 완료 스레드에서 실행됨 → 별도 콜백 스레드로 표현 + 설명 보강.
- `boot-2.x.md`: ① MVC 스택 의존 방향 정정(Spring MVC → Servlet API → Tomcat). ② Prometheus는 push가 아니라 `/actuator/prometheus`를 **scrape(pull)** → 화살표 라벨로 pull/push 구분.
- `framework-3.x.md`: ① DispatcherServlet이 Controller를 **HandlerAdapter**를 통해 호출하도록 정정. ② REST(`@ResponseBody`)는 ViewResolver를 거치지 않고 HttpMessageConverter로 직렬화되는 분기를 `alt`로 추가.
- `framework-4.x.md`: `@ResponseBody` 직렬화는 DispatcherServlet이 직접 컨버터를 호출하는 게 아니라 HandlerAdapter의 ReturnValueHandler가 호출 → 시퀀스에 HandlerAdapter 추가.
- `kotlin-and-spring.md`: `CoroutineCrudRepository <--> ReactiveCrudRepository`는 양방향 타입 변환이 아님 → 코루틴 API 추상화가 리액티브 인프라 위에서 동작하는 단방향 관계로 정정(suspend↔Mono, Flow↔Flux는 양방향 유지).
