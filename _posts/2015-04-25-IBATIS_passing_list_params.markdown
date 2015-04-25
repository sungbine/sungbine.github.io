---
layout: post
title:  "IBATIS에 LIST 파라미터 넘기고 구현하기"
date:   2015-04-25 17:53:00
categories: tech post
---

ibatis SQL mapper를 작성하는 중 LIST를 파라미터로 넘겨야하는 상황이되었다. 

조사한 내용을 기족해 본다.


{% highlight sql %} 
SELECT target_col
  FROM table_name
 WHERE column1 IN ( 'a','b','c')
{% endhighlight%}

위 같은 쿼리를 mapper를 이용해서 작성하려고 할때, 조건문에 있는 "IN ( 'a','b','c')" 조건을 dynamic 하게 생성하려고한다. parameter에 a,b,c가 리스트로 들어있는 경우 구현방법을 알아보자

{% highlight sql %} 
SELECT target_col
  FROM table_name
 WHERE column1 
<iterate property="column1" open="(" close=")" conjunction=",">
  #column1[]#
</iterate>
{% endhighlight%}

위 같은방법으로 <iterate> statement를 이용할 수 있다.

iterate와 함께 이용가능한 속성들을 다음과 같다.
prepend - 표현식앞에 붙는 SQL (optional)
property - 반복되기 위한 LIST타입의 변수(required)
open - iteration 앞에 붙을 문자열(optional)
close - iteration 뒤에 붙을 문자열(optional)
conjuction - 각 iteration 중간에 붙는 문자열, AND, OR와 함께쓰면 유용함(optional)


# One more thing!
controller에서 넘어오는 parameter가 list하나가 아닌 map객체에 담겨서 여러 객체가 오더라도 위와같이 간단히 사용이 가능하다.
괜히 parameterMap 속성을 이용하지 않아도 사용이 가능하다. 괜한 삽질로 parameterMap을 만들지 않고 parameterClass ="hashmap" 을 사용하면 간단하다!

끝!
