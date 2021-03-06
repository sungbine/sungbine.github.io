---
layout: post
title:  " ibatis $preferredOrder$, #value# 의 차이 "
date:   2015-03-30 09:00:00
categories: tech post
---

ibatis로 개발을 하는중에 controller에서 넘어오는 값이 '$'로 감쌓지는 경우와 '#'로 감쌓지는 경우를 보게되어, 두개의 차이점을 찾아 보게 되었다.

### $preferredOrder$는 이곳에 들어오는 문자열 그대로 query문이 만들어진다.

예를 들어

{% highlight sql %}SELECT * FROM table1 WHERE col1 = '$whereClaus$';{% endhighlight%}

이라는 쿼리를  mapper에 작성했다고 가정하자. 그리고 whereCond라는 변수(맵객체의 키가 whereCond인 경우 포함)에 "kim" 이라는 문자열이 들어있다면, 위의 쿼리는 

{% highlight sql %}SELECT * FROM table1 WHERE col1 = 'kim' {% endhighlight %}

이라는 쿼리가 완성되어 실행될 것이다.


### #value#는 값만 넘겨준다.

{% highlight sql %}SELECT * FROM table1 WHERE col1 = #whereClaus#;{% endhighlight %}

맵퍼에 이런 쿼리가 존재한다. 그리고 whereClaus에 "kim"이라는 문자열이 들어있다면
결과적으로 실행되는 쿼리는 위의 1에서와 같은 결과를 실행하게 된다. 그렇다면 차이점은?

유심히 보면

'$whereClaus$'; 이것과 #whereClaus#; 의 차이는 따움표에 있다. $$는 따움표로 감쌓져 있는데 그 이유는 문자열 그대로를 넘기기때문에 따움표가 없으면 "kim"이라는 문자열을 쿼리는 
SELECT * FROM table1 WHERE col1 = kim 
이렇게 생성되고 데이터 타입이 맞지 않기때문에 에러를 뱉어낼 것이다.

반대로 ##로 감쌓여 있는 것은 값을 넘겨서 이것이 문자열인지 잘알고 제대로된 쿼리가 수행된다.
이 같은 차이가 발생하는 이유는 동작원리가 다르기 때문이다.

SELECT * FROM table1 WHERE col1 = '$whereClaus$'; 이 쿼리가 동작하는 순서는

1) SELECT * FROM table1 WHERE col1 = '$whereClaus$'; 에서 $whereClaus$가 문자열 "kim"으로 대체되어 등록된다.
2) 등록된 쿼리가 실행된다.

SELECT * FROM table1 WHERE col1 = #whereClaus#;가  동작하는 순서는
1) SELECT * FROM table1 WHERE col1 = #whereClaus#; 에서 SELECT * FROM table1 WHERE col1 = ?; 쿼리가 등록된다. 
2) 등록된 쿼리에 ?에  넘겨진 변수가 채워진다.
3) 쿼리가 실행된다.


두 과정의 차이가 보이는가?

'$preferredOrder$'는 쿼리가 쿼리 파서에 들어가기 전에 이미 쿼리가 만들어져서 실행되고
'#value#'는 쿼리파서에 들어간후 preparedStatement가 되고 데이터가 바인딩되어 실행되는 차이가 있는 것이다.


사용처에 따라 골라쓰면 유용하게 사용할 수 있다.

'#value#'는 조건절에 변수를 선언하고 쓸경우 유용하고, '$preferredOrder$'는 조건절이 아닌 다른 절에 직접 스트링을 집어넣을 수있다는 장점이 있다. 다이나믹하게 테이블을 변경한다던가 dml자체를 수정 할 수있도록 할 수있다는 강력한 기능이 있다.
