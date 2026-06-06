# JDK 1.3 / J2SE 1.3 (코드네임 Kestrel, 2000년 5월)

> 성능과 안정성에 집중한 버전. HotSpot JVM을 기본 탑재하여 자바의 고질적 약점이던 실행 속도를 크게 개선하고, JNDI·JPDA·Java Sound 등 플랫폼 인프라를 보강했다.

## 릴리스 정보
- 정식 출시일: 2000년 5월 8일
- 개발 주체: Sun Microsystems
- 공식 명칭: J2SE 1.3 (Java 2 Platform, Standard Edition v1.3)
- 코드네임: Kestrel (황조롱이)

## 시대적 배경
J2SE 1.2가 기능을 대폭 확장했다면, 1.3은 그 기능들을 "실제 운영 환경에서 빠르고 안정적으로" 돌아가게 만드는 데 초점을 맞춘 점진적·성숙화 버전이다. 당시 자바는 여전히 "느리다"는 비판을 받았는데, Sun이 인수한 기술을 바탕으로 한 HotSpot JVM의 기본 탑재가 이 인식을 바꾸는 전환점이 되었다. 같은 시기 J2EE 기반 서버 애플리케이션이 본격 확산되면서, JNDI 같은 엔터프라이즈 연계 API의 표준 포함도 중요했다.

## 주요 추가 기능

### HotSpot JVM 기본 탑재
- 적응형 최적화(adaptive optimization)를 수행하는 HotSpot 가상 머신이 기본 JVM이 되었다.
- 자주 실행되는 "핫스팟" 코드를 런타임에 분석하여 집중적으로 네이티브 컴파일·최적화함으로써, 단순 JIT보다 뛰어난 성능을 냈다. 이로써 자바의 "느리다"는 평판이 본격적으로 개선되기 시작했다.

### JNDI (Java Naming and Directory Interface)
- 이전에는 별도 확장으로 제공되던 JNDI가 코어 플랫폼에 통합되었다.
- LDAP, DNS, RMI 레지스트리 등 다양한 네이밍·디렉터리 서비스를 통일된 API로 조회·바인딩할 수 있게 했다. J2EE에서 데이터소스·EJB 룩업의 핵심 메커니즘이 되었다.

```java
import javax.naming.*;

Context ctx = new InitialContext();
Object obj = ctx.lookup("java:comp/env/jdbc/MyDB");
```

### JPDA (Java Platform Debugger Architecture)
- 표준 디버깅 인프라. JVM TI, JDWP(디버그 와이어 프로토콜), JDI(디버그 인터페이스)로 구성되어, IDE와 도구들이 일관된 방식으로 자바 프로그램을 원격 디버깅할 수 있게 했다.

### Java Sound API
- 오디오(샘플 기반 사운드, MIDI) 재생·녹음·합성을 위한 표준 API가 코어에 포함되었다.

### 동적 프록시(Dynamic Proxy)
- `java.lang.reflect.Proxy`를 통해 런타임에 인터페이스 구현 객체를 동적으로 생성하는 기능. AOP, 원격 호출 스텁, 프레임워크의 인터셉터 등에 폭넓게 활용되었다.

```java
MyService proxy = (MyService) Proxy.newProxyInstance(
    MyService.class.getClassLoader(),
    new Class<?>[]{ MyService.class },
    (p, method, args) -> {
        System.out.println("호출: " + method.getName());
        return null;
    });
```

## 그 외 변경 / API 추가
- RMI를 CORBA(IIOP) 위에서 동작시키는 RMI-IIOP 지원 — J2EE 상호운용성 강화.
- JavaSound, Java Naming 통합 외에 `Timer`/`TimerTask` 등 유틸리티 추가.
- 디버깅·프로파일링·성능 관련 개선이 전반적으로 이루어짐.

## 영향과 의의
J2SE 1.3은 화려한 신기능보다 내실(성능·안정성·운영 인프라)을 다진 버전으로 평가된다. 특히 HotSpot의 기본 탑재는 자바를 서버 사이드 주류 언어로 끌어올리는 결정적 계기가 되었고, JNDI·JPDA는 이후 엔터프라이즈 자바와 개발 도구 생태계의 표준 토대가 되었다. "Kestrel" 이후 자바 릴리스에는 새(bird) 코드네임 전통이 자리잡았다.

## 참고 출처
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [Java 1.3 - javaalmanac.io](https://javaalmanac.io/jdk/1.3/)
- [JDK release dates - Java Glossary (mindprod)](https://www.mindprod.com/jgloss/jdkreleasedates.html)
