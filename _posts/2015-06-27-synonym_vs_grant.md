---
layout: post
title:  "Oracle Synonym과 Grant"
date:   2015-06-27 14:34:00
categories: tech post
---
#Oracle의 Grant와 Synonym에 대해 알아 보자.


"여러 DB 계정에서 같은 DB의 동일한 테이블에 접근하려면 어떻게 해야할까?"</br>
"어떤 계정은 Select권한만 있으면 좋겠고, 어떤 계정은 Intert와 Select가 있었으면 좋겠다면?"<br>
위와 같은 경우처럼 DB는 여러 계정을 만들어 상황에 맞게 권한을 설정해서 사용할 수있다. 


예를 들어 A라는 계정은 모든 DML을 사용할 수 있다고 하자. 그런데 필요에 의해 A계정의 TableA 라는 테이블 SELECT만 가능한 B 계정이 필요해 졌다. 

이럴 때 사용하는 명령어가 바로 ***<highlight start>GRANT<highlight end>*** 이다.<br>
GRANT는 계정에 권한을 부여 할 수있는데, 이를 사용할 수 있는 계정은 그 객체(예를 들어 TABLE)의 OWNER만이 부여 할 수있다.<br>

위의 예제에서는 A라는 계정으로 B 계정에 TableA에 GRANT를 설정함으로서 B가 TableA를 SELECT 하도록 할 수있다

``` 
GRANT select on A.TableA to B 
```

라고 하면 B는 

```
SELECT * FROM A.TableA
```
가 가능해 진다.

###이렇게 각 권한을 부여 할 수있도록 하는 것이 바로  GRANT 이다.

근데 불편한 점이 한가지 눈에 띈다. 보통 우리가 쿼리를 작성할때는 Table명만을 사용하여 쿼리르 작성하지 위의 경우처럼 A.TableA라고 계정명을 함께 사용하지 않는다. 아주 번거롭고 귀찮은 쿼리 작성이 될것이다. 이를 위해 사용하는 것이 바로 ***==Synonym==***이다.

Synonym은 객체의 OWNER가 아닌 권한을 부여 받은 계정에서 적용 할 수있다.*(해당 계정은 Synonym 생성 권한이 필요하다. 없다면 sys계정을 통해 부여할 수있다.)*

```
CREATE synonym TableA for A.TableA
```

라고 작성하면 B 계정에서도

``` 
SELECT * FROM TableA 
```

의 쿼리가 정상적으로 동작하게 된다.

일종의 Alias를 지정하는 방법인 것이다.

내가 헷갈렸던 부분은 Synonym에도 기본적으로 권한이 들어가지 않을까 하는 것이었다. 그러나 Synonym은 권한을 부여하지는 않는다. 이를 사용하기 위해서는 꼭 Grant로 권한이 부여된 상태에서만 가능하다는 것을 잊지 말아야 한다.


#Tip. Synonym 제어 쿼리
## synonym 확인
``` 
SELECT * FROM user_sysnonyms 
-- 현재 계정의 synonym만 확인
```
```
 SELECT * FROM all_sysnonyms  
 -- 모든 계정의 synonym을 확인
```

위의 두가지 쿼리를 통해 synonym을 확인 할 수있다.

## synonym 삭제

``` 
DROP synonym TableA 
```

라고 하면 삭제된다.

#==*참고로 MySql과 Cubrid에서는 synonym을 지원하고 있지 않다.*==

