# JDK 1.2 / J2SE 1.2 — "Java 2" (코드네임 Playground, 1998년 12월)

> 자바 역사상 가장 큰 분수령 중 하나. Swing과 Collections Framework가 등장하고, "Java 2" 브랜딩과 함께 J2SE/J2EE/J2ME로 플랫폼이 분화한 버전이다.

## 릴리스 정보
- 정식 출시일: 1998년 12월 8일
- 개발 주체: Sun Microsystems
- 공식 명칭: J2SE 1.2 (Java 2 Platform, Standard Edition) — 버전 번호는 1.2지만 "Java 2"로 마케팅됨
- 코드네임: Playground (일부 출처에서만 쓰이는 비공식 명칭. Sun의 공식 코드네임 관행은 1.3 Kestrel부터 본격화됨)

## 시대적 배경
JDK 1.1까지 자바는 빠르게 성장했지만, GUI(AWT)는 여전히 무겁고 플랫폼 의존적이었으며, 자료구조 API는 `Vector`/`Hashtable` 수준으로 일관성이 떨어졌다. 동시에 자바의 적용 범위가 데스크톱·서버·임베디드로 넓어지면서 하나의 JDK로 모든 영역을 감당하기 어려워졌다.

Sun은 이 버전을 "Java 2"로 새롭게 브랜딩하며, 플랫폼을 용도별로 나눴다:
- **J2SE** (Standard Edition) — 데스크톱/일반 애플리케이션
- **J2EE** (Enterprise Edition) — 서버/엔터프라이즈
- **J2ME** (Micro Edition) — 모바일/임베디드

## 주요 추가 기능

### Swing
- 순수 자바로 그려지는 경량(lightweight) GUI 툴킷. AWT의 네이티브 피어 의존을 벗어나, 모든 플랫폼에서 동일하게 동작하고 Look & Feel을 교체할 수 있다.
- 컴포넌트 이름 앞에 `J`가 붙는다(`JButton`, `JFrame`, `JTable` 등). 이후 10여 년간 자바 데스크톱 GUI의 표준이 되었다.

```java
import javax.swing.*;

public class Demo {
    public static void main(String[] args) {
        JFrame f = new JFrame("Swing 예제");
        f.add(new JButton("클릭"));
        f.setSize(200, 100);
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        f.setVisible(true);
    }
}
```

### 컬렉션 프레임워크(Collections Framework)
- `List`, `Set`, `Map` 인터페이스와 `ArrayList`, `HashMap`, `TreeMap`, `LinkedList` 등 구현체, 그리고 `Collections`/`Arrays` 유틸리티, `Iterator`를 통합 제공.
- 기존의 산발적인 `Vector`/`Hashtable`을 일관된 체계로 대체하여, 자바 프로그래밍의 데이터 처리 방식을 표준화했다.

```java
import java.util.*;

List<String> list = new ArrayList<>(); // (제네릭은 J2SE 5.0부터, 여기선 개념 예시)
list.add("a");
list.add("b");
Collections.sort(list);

Map<String, Integer> map = new HashMap<>();
map.put("one", 1);
for (Map.Entry<String, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + "=" + e.getValue());
}
```

### JIT 컴파일러 기본 탑재
- Sun JVM에 JIT(Just-In-Time) 컴파일러가 기본 포함되어, 바이트코드를 실행 시점에 네이티브 코드로 변환함으로써 성능이 크게 향상되었다.

### strictfp 키워드
- 부동소수점 연산이 플랫폼과 무관하게 IEEE 754 규약대로 동일한 결과를 내도록 강제하는 키워드. 플랫폼 독립성을 수치 연산 영역까지 확장했다.

```java
public strictfp class Calc {
    double compute(double a, double b) { return a * b; }
}
```

## 그 외 변경 / API 추가
- Java Plug-in: 브라우저에서 Sun JRE로 애플릿을 실행하게 해주는 플러그인.
- Java IDL: CORBA와의 연동을 위한 IDL 지원.
- 정책 기반 보안 모델(Policy/Permission) 강화 — 세분화된 권한 제어.
- `Collator`, 접근성(Accessibility) API 등 추가.

## 영향과 의의
J2SE 1.2는 "Java 2"라는 이름이 보여주듯 사실상 자바의 2세대를 연 버전이다. Swing은 데스크톱 자바의 표준이 되었고, 컬렉션 프레임워크는 오늘날까지 모든 자바 코드의 기본 도구로 쓰인다. 플랫폼의 SE/EE/ME 분화는 자바가 임베디드부터 대형 서버까지 전 영역을 아우르는 생태계로 확장되는 출발점이었다. "Java 2" 브랜딩은 이후 J2SE 5.0(2004)까지 약 6년간 유지되었다.

## 참고 출처
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [Java 1.2 - javaalmanac.io](https://javaalmanac.io/jdk/1.2/)
- [JDK release dates - Java Glossary (mindprod)](https://www.mindprod.com/jgloss/jdkreleasedates.html)
