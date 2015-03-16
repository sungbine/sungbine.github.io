---
layout: post
title:  "javascript에서 문자열 replaceAll하기"
date:   2015-03-16 09:00:00
categories: tech post
---

자바에서 문자열 함수 중 replaceAll() 함수가 존재한다. 이것은 문자열 내의 하나의 문자를 모두 찾아서 원하는 문자로 바꿔주는 기능을 한다.
자바스크립트에서도 당연히 존재할 줄 알았던 이 기능이 없다는 사실에 당황했다.

하지만 이와 동일한 기능을 수행하게 할수있다!

어떻게?


정규식과 replace의 옵션을 이용해서!

var tmpStr = 'abcdefgabaa';

라는 문자열이 존재할때, 

tmpStr.replace(/a/g, '!');

라고 쓰면 'a'가 모두 '!'로 바뀐다

원리는 간단하다.  /a/는 정규식으로 a를 찾는것이고 g는 모든 문자열에서 'a'를  다 찾아라른 옵션이 된다. 그렇게 지정하게 되면 replaceAll()과 같은 기능을 수행할 수있다. 한가지 추가로 tmpStr.replace(/a/gi, '!'); 와 같이 gi옵션을 쓰기도 하는데 i옵션은 대소문자 구분을 하지 않는다는 뜻이다.

꿀팁!
