# JDK 1.1 (1997년 2월)

> 자바를 "장난감 언어"에서 본격적인 엔터프라이즈 개발 플랫폼으로 끌어올린 버전. 내부 클래스, JDBC, RMI, 리플렉션, JavaBeans 등 오늘날까지 쓰이는 핵심 인프라가 대거 등장했다.

## 릴리스 정보
- 정식 출시일: 1997년 2월 19일
- 개발 주체: Sun Microsystems
- 공식 명칭: JDK 1.1
- 코드네임: 없음

## 시대적 배경
JDK 1.0이 "자바가 가능하다"를 증명했다면, JDK 1.1은 "자바로 실제 시스템을 만들 수 있다"를 목표로 했다. 1.0의 AWT 이벤트 모델은 비효율적이었고(이벤트를 컴포넌트 계층 위로 전파시키는 방식), 데이터베이스 연동·분산 처리·재사용 가능한 컴포넌트 모델 같은 기업용 기능이 부족했다. JDK 1.1은 이 공백들을 한꺼번에 메웠다.

## 주요 추가 기능

### 내부 클래스(Inner Classes)
- 클래스 안에 클래스를 정의할 수 있게 되었다. 익명 클래스(anonymous class)도 이때 도입되어, 새 AWT 이벤트 모델의 리스너 구현을 간결하게 해주었다.

```java
button.addActionListener(new ActionListener() {   // 익명 내부 클래스
    public void actionPerformed(ActionEvent e) {
        System.out.println("버튼 클릭됨");
    }
});
```

### 새로운 AWT 이벤트 모델 (위임 이벤트 모델)
- 1.0의 상속 기반 이벤트 처리를 버리고, 리스너(listener)를 컴포넌트에 "등록"하는 위임(delegation) 모델로 전면 개편했다.
- 이 모델은 이후 Swing을 포함한 모든 자바 GUI 이벤트 처리의 표준이 되었다.

```java
// 1.0: public boolean action(Event e, Object arg) { ... }  (오버라이드 방식)
// 1.1: 리스너 등록 방식
button.addActionListener(e -> { /* ... */ }); // (람다는 후일이지만 등록 모델 자체는 1.1)
```

### JavaBeans
- 재사용 가능한 소프트웨어 컴포넌트 모델. getter/setter 명명 규약, 이벤트, 인트로스펙션(introspection)을 기반으로 한다.
- 비주얼 개발 도구에서 컴포넌트를 드래그&드롭으로 조립하는 시대를 열었고, 이후 자바 생태계 전반의 "POJO + 프로퍼티" 관례의 뿌리가 되었다.

```java
public class Person {
    private String name;
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

### JDBC (Java Database Connectivity)
- 데이터베이스 접속을 위한 표준 API. 드라이버를 통해 어떤 DB든 동일한 인터페이스로 다룰 수 있게 했다.

```java
Connection con = DriverManager.getConnection(url, user, pwd);
Statement st = con.createStatement();
ResultSet rs = st.executeQuery("SELECT name FROM users");
while (rs.next()) {
    System.out.println(rs.getString("name"));
}
```

### RMI (Remote Method Invocation)와 객체 직렬화
- 원격 JVM에 있는 객체의 메서드를 마치 로컬 객체처럼 호출하는 분산 컴퓨팅 기능.
- 이를 뒷받침하기 위해 객체 직렬화(`Serializable`)가 함께 도입되었다.

```java
public interface Hello extends java.rmi.Remote {
    String sayHello() throws java.rmi.RemoteException;
}
```

### 리플렉션(Reflection)
- 실행 중에 클래스의 필드·메서드·생성자 정보를 조회하는 기능(1.1에서는 주로 인트로스펙션 위주). JavaBeans, 직렬화, 후일의 프레임워크(Spring 등)의 근간이 되었다.

```java
Class<?> c = obj.getClass();
for (java.lang.reflect.Method m : c.getMethods()) {
    System.out.println(m.getName());
}
```

### 국제화(i18n)와 유니코드
- `Locale`, `ResourceBundle`, `java.text` 패키지 도입으로 다국어 애플리케이션 작성이 가능해졌다. 유니코드 지원이 강화되었다.

## 그 외 변경 / API 추가
- JAR(Java ARchive) 파일 포맷 도입 — 여러 클래스와 리소스를 하나로 묶어 배포.
- Windows 환경에서의 JIT(Just-In-Time) 컴파일 지원으로 실행 성능 향상.
- `java.math`(BigInteger, BigDecimal) 추가.

## 영향과 의의
JDK 1.1은 자바의 "엔터프라이즈 시대"를 본격적으로 연 버전이다. JDBC·RMI·JavaBeans·리플렉션은 이후 J2EE와 수많은 프레임워크의 기반이 되었고, 위임 이벤트 모델은 자바 GUI의 표준으로 굳어졌다. 이 시점부터 자바는 단순 웹 애플릿 언어를 넘어 서버·기업용 시스템 언어로 자리잡기 시작했다.

## 참고 출처
- [Java version history - Wikipedia](https://en.wikipedia.org/wiki/Java_version_history)
- [Java 1.1 - javaalmanac.io](https://javaalmanac.io/jdk/1.1/)
- [JDK release dates - Java Glossary (mindprod)](https://www.mindprod.com/jgloss/jdkreleasedates.html)
