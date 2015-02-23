---
layout: post
title:  "tomcat과 apache 연동하면서 겪은 애로 사항"
date:   2015-02-23 20:00:00
categories: tech post
---

Tomcat과 Apache 웹서버를 직접 설치하면서 겪은 어려움과 해결법을 나열해 보겠다.
Linux CentoOS 6.6, jdk 1.8, Tomcat 8.0, Apache HTTP 웹서버 2.4.9 버전을 이용했다.

1. jdk 버전 바꾸기
  /etc/alternatives라는 명령어를 이용해 여러버전의 JDK가 설치되어있는 환경의 Default JDK 변경할 수 있다.
  처음 할당받은 서버에 이미 JDK 1.6이 설치가 되어있는 상황이었는데, JDK 1.8을 사용해야하는 상황이었기 때문에 어떻게 변경할 수있을지 고민이 많았다. alternatives라는 명령을 이용해 해결할 수 있었다.
  - alternatives --install /usr/bin/java java /user/wh/jdk1.6.0_35/bin/java 100 ## JDK를 변경가능한 리스트에 추가한다.
  - alternatives --config java ## 이 옵션을 이용하면 멋진UI(?)를 통해 변경이 가능하다.
  이뿐만 아니라, 명령어를 직접 입력하는 방법이 있는것같으나(사실 이렇게 해결했는데 어떻게 했는지 검색해도 도통 찾을수가 없다.) 이방법이 훨씬 편리하고 간단하다.

2. apache build 시 에러1
  apache 빌드시 
    ./configure
    --prefix=/usr/local/apache2 
    --enable-all         
    --enable-so
    --with-included-apr
    --with-mpm=prefork
이런 명령으로 configure을 하게 된다. 이때 이전 단계에서 apr과 apr-util의 위치를 잘못 지정해 주는 바람에 빌드가 되지 않아 많은 삽질을 햇다.

3. apache build 시 에러2
  빌드를 준비하는 과정에서  yum install make gcc gcc-c++ autoconf automake libtool pkgconfig findutils 이런 명령어를 인테넷에서 그래도 썼는데 저중 어디선가 제대로 설치가 되지 않아서 무엇인가 찾을 수 없다는 메세지를 보았다. 하나하나 꼼꼼히 다시 설치해서 해결했다. 그리고 jk_mod를 컴파일하는 과정에서 apxs의 경로를 설정해주는데 이것은 /usr/local/bin 폴더에 pcre라는 파일이 존재하느데, 이곳으로 경로를 지정해줘야 성공적으로 빌드가 된다.

4. tomcat과 apache의 권한 문제...
  이것은 아주 특수한 케이스일 수 있다(우리 사내에서만 해당) su(root를 위임한)계정과 그렇지 않은 계정을 이용해 apache와 tomcat을 설치하게 되는데.. 두개를 모두 su계정으로 설치했더니 tomcat이 동작하지않는 문제가 생겼다. 이것은 어떤 이유인지 정확히 모르겠으나(알아보겟다!!!) tomcat을 su계정이 아닌 계정으로 설치했더니 해결되었다.

5. connector설정 문제..
  tomcat과 apache를 연동하기 위해 connector를 설치했는데 apache의 http.conf파일로 jk_mod 무듈을 로드하고 virtual host를 지어해야하는 설정이 필요하다. 아직 http.conf파일의 구조와 기능(특히 virtualhost)에 대해 지식이 부족하여 많은 공부가 필요하다. 일단은 인터넷에 있는 가이드를 참고하여 진행했다.
