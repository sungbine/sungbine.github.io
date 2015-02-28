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
### Java 6 필요
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
The roles command has been removed from the Manager application since it did not work with the default configuration and most Realms do not support providing a list of roles.
