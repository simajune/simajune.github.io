---
layout: post
title: "Observable이란"
author: "Tejay"
tags: "iOS"
---

# Observable이란

### 1. Observable

- 이제 본격적으로 RxSwift의 용어에 대해 알아보자
- Observable은 과연 무엇이냐? 바로 Rx의 심장이라 할 수 있다. 이번엔 이 Observable을 만들고 어떻게 거것을 사용하는지 알아볼 것이다.
- 일단 여기서 얘기하는 Observable은 Oservable Squence, Sequence라고 할 것이다. 다 동일하다.
- Observable은 이벤트를 생성하는데 이 이벤트는 일정 시간동안 방출하는 것이다. 간단하게 밑에 그림을 통해 알아보자.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch2-1/1.png?raw=true" width="800px"/>

* 위에 그림처럼 화살표는 시간이고 숫자는 시퀀스 요소이다. 이 그림을 볼 때 1, 2, 3 순서대로 방출하게 된다.
