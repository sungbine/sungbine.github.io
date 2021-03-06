---
layout: post
title:  "HTML INPUT 태그의 콤마가 포함된 STRING의 SPRING에서의 파라미터 바인딩"
date:   2015-03-07 15:00:00
categories: tech post
---



프로그래밍 중 이상한 현상을 경험했다. Spring 프레임웍으로 서비스중인 웹어플리케이션의  input 태그의 value에 "123,444,555" 처럼 숫자와 컴마로 구성된 값을 컨트롤러로 전송했다. 이 값을 바인딩 하는 VO의 변수 타입이 String[] foo이었고, 이때 내가 의도한 것은 foo[0]="123,444,555" 가 assign되길 기대했다. 그런데 바인딩 결과는 foo[0] = "123", foo[1] = "444", foo[2] = "555" 로 하나의 배열로 변수에 할당되었다. 그 이유가 궁금해 조사해 보았다.

이유는 스프링에서 기본으로 제공하는 바인더가 이런 기능을 포함하고 있기 때문이다. [참고](http://springsource.tistory.com/15)

위의 링크에 따르면 Spring의 기본 property editor가 ','를 포함한 스트링에 대해 배열로 처리하게 되어있다. 

리퀘스트가 들어오면 3가지 과정으로 동작한다.<br/>
1. 파라미터 타입의 오브젝트 생성<br/>
2. 웹파라미터 바인딩<br/>
3. validation<br/>

세개의 과정을 거치는중 binding하는 과정이 있다. 이 때 프로젝에서 별도의 binder를 지정해주지않았기 때문에 Spring의 기본 property eiditor를 사용했고, 원하지 않은 방법으로 바인딩이 이루어졌던 것이다.
이를 해결하기 위해 별도의 binder를 지정해 주므로서 해결이 가능하다. 
바인딩하는 방법에는 두가지가 있다.  xml을 통해 resolver를 지정해 주거나,  initBinder 함수와 annotation 을 사용하는 방법이다.
내가 처리했던 방법은 컨트롤러를 수정하기 보다 front에서 넘어 오는값에서 ','를 제거하는 방법을 택했다(이유는 db에 저장될때 다시 ,가 제거되어야 하기때문에 두번의 과정을 거치는 것보다 효율적이라 생각했기 때문이다)

이 관련된 것을 조사하는중 url파라미터와 modelAttriubte 가 처리되는 handler의 차이가 있다는 것을 알게 되었고 이다음에 이부분에 집중 분석해 봐야할 필요성을 느꼈다.


[참고-Rednics Blog(http://springsource.tistory.com/15)](http://springsource.tistory.com/15)
