---
layout: post
title:  "tomcat6 에서 tomcat7 migration 하기"
date:   2015-02-28 14:00:00
categories: tech post
---


현재 사용하고 잇는 tomcat 6.0.x 버전을 더 높은 버전의 tomcat으로 migration 하기 위해 필요한 내용이다.
이 글은 http://tomcat.apache.org/migration-7.html 사이트의 내용을 정리한 것임을 밝힌다.

http://tomcat.apache.org/migration.html 페이지에 내용을 통해 다른 버전의 migration을 확인할 수 있다.

# 6.0.x에서 7.0.x 로 migration하기 #
### Java 6
 * Apache Tomcat 7.0.x는 Java 6이상의 버전이 필요 하다. (Apache Tomcat 6.0.x는 Java5를 필요로한다)

### Servlet 3.0 API 
* Apache Tomcat7은 Java Servlet 3.0, JavaServer Pages(JSP) 2.2, Expression Langues(EL) 2.2을 지원한다([spec](http://wiki.apache.org/tomcat/Specifications)).
* JSP페이지에 아래와 같은 Wildcard import 문법을 사용하는 경우 Sevlet API에서 새롭게 추가된 클래스와 충돌이 발생할 수있다. 그러면 Tomcat 7은 컴파일을 정지한다. 아래 코드는 "a" 패키지에 Part 클래스가 있는 경우이다.
  <br /> <br /> 
      ```
      <%@page import="a.*"%>
      <% Part page = new Part(); %>
      ```
      <br /> <br /> 
이 현상이 발생하는 이유는 javax.servlet.http.\* 의 implicit import와 a.\*이 동일한 Part 정의를 제공하기 때문이다. (Servlet 3.0에 Part 클래스가 추가되었다.) 해결책은 import="a.Part" 와같은  explicit import 방법을 사용하는 것이다.

### 정규식(Regular expressions)
* 정규식을 활용한 모든 설정 옵션값은 콤마나 세미콜론으로 구분된 리스트형태의 정규식보다 하나의 정규식을 요구한다(java.util.regex를 사용). 
* 이것은 아래와 관련된다
  * RemoteAddrFilter, RemoteHostFilter, RemoteAddrValve, RemoteHostValve의  allow와 deny 어트리뷰트들
  * RemoteIpFilter, RemoteIpValve 의 internalProxies와 trustedProxies 어트리뷰트들
  * ReplicationValve의 filter 어트리뷰트
  * HTTP connectos의 restrictedUserAgents, noCompressionUserAgents 어트리뷰트들
* 분리된 여러 정규식을 하나로 합치는 방법은 "\|" 연산자(or)를 이용하는 것이다. "\|"연산자는 Tomcat이전버전과도 호환한다.

### 배포(Deployemnt)
* XML 컨텍스트 기술파일(META-INF/context.xml)은 더이상 배포된 WAR와 디렉토리에서 호스트의 xmlBase로 복사되지 않는다. Tomcat6과 같이 복사하기 위해서는 Host 요소의 copyXML 속성을 true로 셋팅이 필요하다.
* 7.0.12부터 7.0.47까지 버전에서는 호스트의 appBase밖의 WAR는 unpack 되지 않는다(Host의 unpackWARs 셋을 하더라도). 이 unpacking은 7.0.48 이후로 지원한다.

### 관리 어플리케이션(Manager Application)
* 관리 어플리케이션은 tomcat7이후로 재구성되었고 어떤 URL은 바뀌었다. 관리 어플리케이션에 접근하기 위해 사용되는 모든 URL들은 아래의 옵션중 하나로 시작해야한다.
  * <ContextPath>/html - HTML GUI
  * <ContextPath>/text - text interface
  * <ContextPath>/jmxproxy - JMX proxy
  * <ContextPath>/status - status pages
* HTML 인터페이스는 CSRF(Cross Site Request Forgery)에 대비해 보안이 되어있다. 그러나 text, JMX인터페이스는 그렇지 않다. CSRF 보안을 위해 아래의 사항을 따라야한다.
  * manager-gui 권한과 manager-script 또는 manager-jmx권한들을 분리되어야 한다.
  * manager-script나 manager-jmx 권한을 이용해 브라우저로 관리 어플리케이션에 접근하는 경우에 세션이 종료되고난후 반드시 브라우저를 종료해야한다.
기본 설정과 대부분의 realm들이 이것을 지원하지 않기 때문에 역할 명령어(role command)는 제거되었다.

### 호스트 관리 어플리케이션(Host Manager Application)
* 홋트 관리 어플리케이션은 tomcat7이후로 재구성되었고 어떤 URL은 바뀌었다. 관리 어플리케이션에 접근하기 위해 사용되는 모든 URL들은 아래의 옵션중 하나로 시작해야한다.
  * <ContextPath>/html - HTML GUI 페이지
  * <ContextPath>/text - text 인터페이스
주목할점은 text interface URL이 기존 "<ContextPath>" 에서  "<ContextPath>/text"로 바뀐것이다.

  권한(role)들은 단일 admin권한에서 아래처럼 두개의 권한으로 분리된 호스트 관리 어플리케이션을 사용해야햔다. 사용자는  기능적으로 필요한 역할을 할당해야한다.
* amdin-gui - HTML GUI와 상태페이지 접근을 허용한다.
* admin-script - text 인터페이스와 상태페이지 접근은 허용한다.
HTML 인터페이스는 CSRF에 대해 보안이 되어있지만 text 인터페이스는 그렇지 않다. CSRF 보안을 위해 아래를 따라라.
* admin-gui 권한을 가진 사용자는 admin-script 권한을 가져서는 안된다.
* admin-script권한을 가진 사용자가 브라우저를 통해 접근하는 경우세션이 종료되고난후 반드시 브라우저를 종료해야한다.

### 세션 관리 설정
세션관리에 많은 변화가 있었다. 이것은 세션을 생성되고 제거할때 또는 세션의 아이디생성하는 성능을 향상하기 위해 이루어졌다. 세션 아이디 생성의 변화는 java.secure.ScureRandom을 사용하면서 변화했다. 설정의 변화는 이렇다.
* Manager의 randomClass 속성은 secureRandomClass로 바뀌었고, 제공되는 클래스는 반드시 java.secure.SecureRandom을 확장해야한다.
* secureRandomAlgorithm과 secureRandomProvider의 두개의 새로운 property들은 ScureRandom의 구현선택을 활성화하기 위해 추가되었다.
* algorithm 속성은 제거되었다.
* entropy 속성은 제거되었다.

java.secure.SecureRandom과 관련된 이슈중 하나는 이것의 초기화시 entropy 소스에 있는 임의(random)의 데이터를 요구하는 것이다. 어떤 entorpy 소스 구현에서는 임이의 데이타를 모으는데 어느정도 시간이 걸릴 수있다. 만약 세션아이디 생성의 초기화가 100ms 이상의 시간을 사용한다면 아래의 진단 메시지가 로그에 나타날 것이다.
ex:
DATE org.apache.catalina.util.SessionIdGenerator createSecureRandom 
INFO: Creation of SecureRandom instance for session ID generation using [SHA1PRNG] took [406] milliseconds.

JRE를 통해 entropy 소스를 바꾸는 것이 가능하다. (ex : -Djava.security.egd=file:/dev/./urandom)

### 세션쿠기 설정(Session cokie configuration)
Servlet 3.0 스펙에서 SessionCookieConfig가 추가되면서 코드의 복잡도를 낮추기위해 많은 수의 쿠키설정 옵션이 제거되었다. 
* Connector.emptySessionPath: 이 옵션은 제거되었다. 글로벌 context.xml(CATALINA_BASE/conf/context.xml)의 sessionCokiePath="/"설정을 동일한 기능이 가능하다.
* org.apache.catalina.SESSION\_COOKIE\_NAME 시스템프로퍼티: 이 옵션도 제거되었다. 글로벌 contex.xml 파일의 sessionCookieName 설정을 통해 동일한 기능이 가능하다.
* org.apache.catalina.SESSION\_PARAMETER\_NAME 시스템프로퍼티: 이 옵션도 제거되었다. 글로벌 contex.xml 파일의 sessionCookieName 설정을 통해 동일한 기능이 가능하다.
* Context.disableURLRewriting: 이 옵션도 제거되었다. 웹어플리케이션이나 CATALINA\_BASE/conf/web.xml 파일의 session-config/tracking-mode 요소를 설정함으로써 동일한 기능이 가능한다.

Tomcat7에서 세션과 SSO 쿠키는 기본값으로 HttpOnly플래그가 셋팅되어 보내진다. 이것은 자바스크립트로가 접근하는것을 막기위해 브라우저에게 지시하는 것이다. 이는 더 안전한 방법이지만, 자바스크립트가 쿠키의 값에 접근하는 것을 방지한다. 이 기능은 Context 요소의 useHttpOnly 속성을 통해 조절된다.(이미 Tomcat6.0의 마지막 버전에서도 구현이 되어있지만 기본적으로 false로 설정되어있다. 이를 사용하기위해서는 CATALINA\_BASE/conf/context.xml 파일의 useHttpOnly="true"를 설정함으로써 사용가능하다)

### 쿠기(Cookies)
톰캣은 기본적으로 스펙을 준수하지않은 이름만 쿠키인 것들은 더이상 수용하지 않다. 그러나 새로운 시스템 프로퍼티가 추가되었다. org.apache.tomcat.util.http.ServerCooki.ALLOW\_NAME\_ONLY 는 이러한 쿠키를 수용할 수있게 한다.

### 요청 속성(Request attributes)
커스텀 request속성 javax.servlet.request.ssl\_session(새로운 표준 request 속성을 위해 제거되었다. servlet 스펙문서의 javax.servlet.request.ssl\_session\_id에서 확인가능하다) 은 SSL 세션아이디에 접근하기 위해 제공된다. 커스텀 request속성은 tomcat8에서 제거될 것이다.

### Comet
코맷 클래스가 org.apache.catalina package 에서 the org.apache.catalina.comet package 으로 옮겨졌다. 코맷을 사용하는 코드는 새로운 패키지 이름을 반영하여 재 컴파일 해야한다.

### XML 검증(validation)
XML 검증 설정이 간편해 졌다. xmlValidation과 xmlNamespaceAware 속성이 Host요소에서 제거됐다. 이 속성들은 tldValidation과 tldNamespaceAware와 같이 Context 요소마다 설정된다. 기본설정(fales) 은 바뀌지 않았다. 그러나 Servlet의 스펙에 의하면, 만약 org.apache.catalina.STRICT\_SERVLET\_COMPLIANCE 스스템 프로퍼티가 true로 설정 되어있다면 XML 검증과 namespace 인지가 초기값으로 활성화될 것이다.

### 시스템 프로퍼티
org.apache.catalina.STRICT\_SERVLET\_COMPLIANCE 시스템 프로퍼티가 수정되었다. 각각의 변화는 해당하는 시스템 프로퍼티에 의해 조절된다. 기본적인 행위는 변화지 않았다. 이제 org.apache.catalina.STRICT\_SERVLET\_COMPLIANCE 시스템프로퍼티는 spec 준수 기본값이 다른 시스템 프로퍼티에서 사용되는지 아닌지를 주절한다. 만약 org.apache.catalina.STRICT_\SERVLET\_COMPLIANCE이 true라면 개별적인 시스템 프로퍼티보다 언제나 우선시 된다.
org.apache.coyote.MAX\_TRAILER\_SIZE는 제거됐다. 그리고 Connector의 maxTrailerSize가 이를 대체한다.

### conf/web.xml 파일의 처리
Servlet 3.0 spec은 어플리케이션의 web.xml 파일이 어떻게 web fragments와 annotations를 조합하는지 정의한다. 서버전반의 기본값을 정의 하는 conf/web.xml파일의 처리는 이런 rule 구현의 결과로 변했다.
주목할만한 변화는 글로벌 conf/web.xml에 정의된 필터가 이제는 web application의 web.xml을 따른다는 점이다.

### Welcom 파일 처리
welcome파일의 처리는 Servlet 3.0 spec의 설명을 따르도록 변했다. 만약 welcome 파일들의 리스트에 servlet에 의해 처리되는 파일(예를 들어 *.jsp)이 추가되어있을 경우 변화된 행위를 관측할 수 있다. Context 의 resourceOnlyServlets 을 보길바란다.

### Annotation 스캐닝
Annotation 스캐닝은 웹어플리케이션의 startup 시간에 영향을 미친다. 또한 클래스의 스캔에 필요한 메모리 양도 늘어난다. Servlet 2.4와 그 이전버전을 스캔을 사용하는 어플리케이션은 이점을 유의해야한다.
이 이슈를 다루는 여러방법이 있다. 추천하는 방법은 이러한 어플리케이션들은 annotation스캐닝을 요구하지않는다고 마크하는 것이다. WEB-INF/web.xml 파일에서 아래의 절차로 가능하다.
* web-app 요소에 웹어플리케이션이 사용하는 spec의 버전이 3.0이 되도록 바꿔라. 이것은 기본 conf/web.xml파일에서 version. xsi:schemaLocation, xmlns, xmlns:xsi 속성을 복사할 수 있다.
* metadata-compete="true" 속성을 web-app 요소에 추가하라.
* <absolute-ordering/> 속성을 추가하라.
metadata-complete 속성은 Servlet 2.5 스펙부터 지원하기 시작했다. absolute-ordering은 Servlet 3.0을 요구한다.

두 번째 방법은 JarScanner 컴포넌트가 특정 JAR파일(이름에따라)을 무시하도록 설정하는것이다. 이것은 보통 conf/catalina.properties파일에서 설정한다. [System properties](http://tomcat.apache.org/tomcat-7.0-doc/config/systemprops.html)챕터의 jarsToSkip을 확인하라. Tomcat 7.0.30을 시작으로 Servlet 3.0에서 JARs 스캐닝(annotation과 웹어플리케이션 fragment)과 TLD 스캐닝(tag 라이브러리들)에서 분리시켜서 스킵하는 설정하는것이 가능하다. 톰캣의 이후 버전은 더욱 나은 방법을 제공할 것이다.

### TLD 처리
TLD처리에서 많은 향상이 있었다. 게다가 일관성을 향상하고 중복을 제거하기 위해 내부적인 리펙토링을 했고, 많은 기능적인 향상이 있었다.
* 이제는 태그 파일의 EL 처리가 태그파일에 선언된 JSP 버전과 일괄성이 있게 되었다.
* JSP 스펙의 JSP 7.3.1의 요구사항이 이제는 강요된다. 그리고 TLD파일들은 WEB-INF/lib 나 WEB-INF/classes 위치에 존재하는것이 허용되지 않는다.

### 내부 API들(Internal APIs)
Tomcat 7 내부 API들이 대부분이 Tomcat 6과 호환하지만 세부적인 수준에서 많은 변화가 있었다. 때문에 바이너리(binary)호환은 하지 않느다. Tomcat의 내부 API를 사용하는 커스텀 컴포넌트의 개발자는 관련된 API의 JavaDoc을 확인하길 바란다.

특히 주의할 점은
* 모든 컴포넌트들이 확장하고 있는 Lifecycle interface의 표준구현
* 제네릭의 사용
* Host conntext의 유일한 식별자로서의 Context path를 사용하기보다 Context의 이름의 직접 사용의 경우

### JSP 컴파일러
성능 최적화의 하나를 컨트롤하는 JspServlet의 초기화 파라미터의 이름이 genStrAsCharArray에서 genStringAsCharArray로 바뀌었고 Apache Ant의 Jasper Task의 관련된 속성의 이름과 일관된다.

# 7.0.x 로 업그레이드 하기(Upgrading 7.0.x)
### Tomcat 7.0.x의 주목할만한 변화
Tomcat개발자는 완벽하게 이전 버전화 호환하게 패치를 릴리즈 하려고 한다. 때때로, 버그를 고치기위해 이전 버전과의 호환성을 지키지 못하는 경구가 있다. 대부분의 경우 이런 변화는 간과된다. 이 섹션은 이전버전과 완벽히 호환되지 않고 업그레이드 할때 충돌이 있을수 있는 변화를 나열한다.
* 7.0.51 이후에는 웹어플리케이션 클래스 로더가 시스템 클래스 로더보다 클래스를 로딩하는데 높은 우선권을 가진다.

### Tomcat 7.0.x 설정 파일의 차이점들
Apache Tomcat 의 인스턴스를 7의 한버전에서 7의 다른 버전으로 업그레이드할때(특별히 분리된 $CATALINA\_HOME 과 $CATALINA\_BASE를 를 사용하는 경우), 새로운 설정들이나 적용되는 기본값이 달라지는 설정파일을 확인해야한다. 이것을 보조하기 [여기](http://tomcat.apache.org/migration-7.html)의 맨 아래 폼을 참고하길 바란다.



# 기타
원본 - [아파치 톰캣](http://tomcat.apache.org/migration-7.html)
