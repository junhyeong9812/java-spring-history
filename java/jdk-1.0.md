# JDK 1.0 (1996년 1월)

> 자바의 최초 정식 릴리스. "한 번 작성하면 어디서나 실행된다(Write Once, Run Anywhere)"는 비전을 처음으로 세상에 내놓은 버전이다.

## 릴리스 정보
- 정식 출시일: 1996년 1월 23일
- 개발 주체: Sun Microsystems
- 공식 명칭: JDK 1.0 (안정판은 JDK 1.0.2)
- 코드네임: 없음

## 시대적 배경
1990년대 중반은 웹 브라우저가 막 대중화되던 시기였다. 당시 웹은 정적인 HTML 문서 중심이었고, 브라우저 안에서 동적인 프로그램을 실행할 방법이 마땅치 않았다. Sun Microsystems는 원래 가전기기용 임베디드 언어(Oak 프로젝트)로 출발했던 기술을 인터넷 시대에 맞게 재정비하여 "자바"로 발표했다.

JDK 1.0의 핵심 메시지는 플랫폼 독립성이었다. 소스를 바이트코드로 컴파일하고 각 운영체제에 맞는 JVM(자바 가상 머신)이 이를 실행함으로써, 윈도우/맥/유닉스를 가리지 않고 같은 프로그램이 돌아간다는 점이 당시로서는 혁신이었다. 특히 Netscape Navigator 브라우저가 자바 애플릿을 지원하면서 자바는 급속히 주목받았다.

## 주요 추가 기능

### 자바 언어와 가상 머신(JVM)
- 객체지향 언어의 기본 구조(클래스, 상속, 인터페이스, 예외 처리)와 자동 메모리 관리(가비지 컬렉션)를 갖춘 최초 버전이다.
- 소스(`.java`)를 바이트코드(`.class`)로 컴파일하고, JVM이 이를 해석 실행하는 모델을 확립했다.

```java
// 최초의 전형적인 자바 프로그램
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### AWT (Abstract Window Toolkit)
- 최초의 GUI 라이브러리. 버튼, 텍스트필드, 레이아웃 등 기본 컴포넌트를 제공했다.
- "피어(peer)" 방식으로 각 OS의 네이티브 위젯을 그대로 사용했기 때문에, 플랫폼마다 모양과 동작이 조금씩 달라지는 한계가 있었다(이른바 "최소 공통 분모" 문제).

```java
import java.awt.*;

public class MyFrame extends Frame {
    public MyFrame() {
        setTitle("AWT 예제");
        add(new Button("클릭"));
        setSize(200, 100);
        setVisible(true);
    }
    public static void main(String[] args) {
        new MyFrame();
    }
}
```

### 애플릿(Applet)
- 웹 브라우저 안에서 실행되는 자바 프로그램. JDK 1.0을 대중에게 각인시킨 대표 기능이다.
- `java.applet.Applet`을 상속하고 `paint()` 등을 구현하여, HTML의 `<applet>` 태그로 페이지에 삽입했다.

```java
import java.applet.Applet;
import java.awt.Graphics;

public class HelloApplet extends Applet {
    public void paint(Graphics g) {
        g.drawString("Hello from Applet!", 20, 20);
    }
}
```

## 그 외 변경 / API 추가
- 초기 릴리스는 8개 패키지로 구성되었다: `java.applet`, `java.awt`, `java.awt.image`, `java.awt.peer`, `java.io`, `java.lang`, `java.net`, `java.util`.
- `java.io`: 스트림 기반 입출력.
- `java.net`: 소켓, URL 등 네트워킹.
- `java.util`: `Vector`, `Hashtable`, `Enumeration` 등 초기 자료구조(아직 Collections Framework 이전).

## 영향과 의의
JDK 1.0은 "플랫폼 독립적 실행"이라는 개념을 산업 표준급으로 끌어올렸다. 비록 AWT는 무겁고 일관성이 부족했으며 애플릿은 후일 보안과 성능 문제로 쇠퇴했지만, 바이트코드 + JVM 모델은 이후 30년 가까이 이어지는 자바 생태계의 토대가 되었다. 이 버전은 기능보다도 "방향성"을 제시했다는 점에서 역사적 의의가 크다.

## 참고 출처
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [Java 1.0 - javaalmanac.io](https://javaalmanac.io/jdk/1.0/)
- [JDK release dates - Java Glossary (mindprod)](https://www.mindprod.com/jgloss/jdkreleasedates.html)
