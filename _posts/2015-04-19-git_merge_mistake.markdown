---
layout: post
title:  "GIT 잘못된 머지방법"
date:   2015-04-19 23:14:00
categories: tech post
---

GIT을 이용한 개발 중 Merge의 실수로 큰 어려움을 겪었다.

이 글을 보는 개발자라면 이러한 머지 방법은 지양하는 것이 좋을것이다.

사건은이러하다.

develope의 branch에 A 개발자는 자신의 개발중인 코드를 merge했다.

B는 develope에서 브랜치를 새로 만들고, 자신의 코드를 개발하여 커밋하였다.

그리고 이 커밋을 master에 merge하게된다. 그런데 이때 B는 A의 코드가 master에 머지되는 것을원치 않아 의도적으로 A의 코드를 삭제하고 merge를 했다. (이후 A가 코드를 merge하면 아무 문제가 없을것이라 착각하고..)

하지만 A가 자신의 코드를 master에 merge하려고 할때, merge할 커밋에 A의 코드가 정상적으로 나오지않는다. 그 이유는 이미 B의 이전 merge시점에서 A의 코드를 지워버렸기 때문에이다. A가 merge하려는 코드가 B가 merge한 시점의 이전의 버전이라면 A가 지워버린 것이 그대로 반영된다.
결과적으로 A의 코드가 정상적으로 master에 merge할 수 없게 된다. 

이것을 해결하는 방법은 여러가지가 있을 수 있다.
