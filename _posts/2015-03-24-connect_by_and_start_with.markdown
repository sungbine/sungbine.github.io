---
layout: post
title:  "SQL에서 CONNECT BY 와 START WITH의 활용"
date:   2015-03-14 21:00:00
categories: tech post
---

SQL문중 특이한 SQL 문이 존재한다.

CONNECT BY와 SATRT WITH 문이 그것이다. 이 구문은 ORACLE DB에 존재하던 구문으로 CUBRID에서도 지원하고 있다.

이 구문을 사용하는 때는 하나의 디비에 계층형으로 이루어진 DB를 다루는데 유용한다.

예를 들어 

|| id || name || upper_id ||
| 1  | 자동차 | 0|
| 2  | 바퀴 | 1|
| 3  | 고무 | 2|

와 같은 형태의 DB 가 존재한다고 할 때, 고무의 상위 항목은 바퀴이고 바퀴의 방위 카테고리는 자동차이다

자동차가 바퀴를 포함하고 바퀴는 고무를 포함하는 관계를 하나의 DB로 표현하면 이러한 식으로 표현이 가능하다. (이러한 구조를 순환관계 테이블이라고도 부른다.)

위 순환관계 테이블에서고무를 조회할때 고무가 포함되는 상위 카테고리를 알고 싶을때, join구문과 CONNECT BY와 START WITH를 사용하면 효과적으로 조회할 수있다.

SELECT name 
  FROM mydb db1
     , mydb db2
CONNECT BY db1.id = db2.upper_id
START WITH upperId = 0
라는 구문을 사용하면
