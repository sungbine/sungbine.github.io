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

| id | name | upper_id |
| 1  | 자동차 | 0|
| 2  | 바퀴 | 1|
| 3  | 고무 | 2|

와 같은 형태의 DB 가 존재한다고 할 때, 고무의 상위 항목은 바퀴이고 바퀴의 방위 카테고리는 자동차이다

자동차가 바퀴를 포함하고 바퀴는 고무를 포함하는 관계를 하나의 DB로 표현하면 이러한 식으로 표현이 가능하다. (이러한 구조를 순환관계 테이블이라고도 부른다.)

위 순환관계 테이블에서고무를 조회할때 고무가 포함되는 상위 카테고리를 알고 싶을때, CONNECT BY와 START WITH를 사용하면 효과적으로 조회할 수있다.
```sql
SELECT name 
  FROM mydb
START WITH upperId = 0
CONNECT BY PRIOR id = upper_id
```
라는 구문을 사용하면 계층화된 db를 접근이 가능하다.

START WITH 구문은 계층화된 구조의 루트노드를 지정하기 위해 사용된다.
CONNECT BY PRIOR절은 부모와 자식노드간의 관계를 설정하기 위해 사용되었다. 특히 PRIOR 은 upper\_id가 아닌 id앞에 붙어야한다.  부모노드의 id와 현재 노드의 upper\_id간의 관계설정을 위해 부모 노드의 id에 PRIOR 을 지정해야한다.

또한 특정 조건의 열에대해서 필터링을 위해서는 FROM과 START WITH사이에 WHERE 절을 삽입 할 수있다.

```sql
SELECT name 
  FROM mydb
 WHERE id = '1'
START WITH upperId = '0'
CONNECT BY PRIOR id = upper_id
```

위와 같은 절이 가능하다. 절의 실행 순서는 START WITH -> CONNECT BY PRIOR -> WHERE 임을 유의해야한다.

또한 connect by는 잘못된 구조의 DB에서 잘못사용하는 경우 무한루프가 발생할 수있다. 이때 connect by prior은 에러를 발생시키지만  connect by nocycle prior 을 사용하면 에러가 나지않고 처리할 수있다.


그리고 한가지 유용한 가상의 컬럼이 존재한다. 
LEVEL이라는 컬럼으로 실제 TREE 구조에서의 level을 의미하며 깊이를 알 수있어 유용하게 활용 할 수있다.
