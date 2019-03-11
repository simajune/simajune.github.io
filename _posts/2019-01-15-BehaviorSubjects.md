---
layout: post
title: "BehaviorSubjects란"
author: "Tejay"
tags: "iOS"
---

# BehaviorSubjects란

- BehaviorSubject는 PublisherSubject와 마찬가지 이지만 다른 점은 구독했을 때 최신의 next이벤트를 구독자에게 재생시킨다.
- 아래 그래프를 보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch3-3/1.png?raw=true" width="800px"/>

* 첫번째 구독자는 1번 이벤트가 발생한 후에 구독을 시작했다 하지만 구독한 시점에 최근에 발생한 next의 이벤트 1번이 방출된다.
* 두번째 구독자는 2번 이벤트가 발생하기 전까지 구독을 하지 않고 그 이후에 구독을 했다. 그래서 그독 시점에 가장 최근에 발생한 2번 이벤트가 발생하고 그 다음 발생한 3번의 이벤트를 받게 된다.

```swift
example(of: "BehaviorSubject") {
  //1
  let subject = BehaviorSubject(value: "initial Event")
  let disposeBag = DisposeBag()
  //2
  let initialSubscription = subject.subscribe(onNext: {
    print("Line number is \(#line) and value is", $0)
  })
  //3
  subject.onNext("second Event")
  //4
  let secondSubscription  = subject.subscribe(onNext: {
    print("Line number is \(#line) and value is", $0)
  })
}
-----------------------------------------------
--- Example of: BehaviorSubject ---
Line number is 15 and value is initial Event
Line number is 15 and value is second Event
Line number is 20 and value is second Event
```

- 위에 코드를 보면 출력을 통해 어떤 구독자에서 이벤트가 발생하는 볼 수 있다.
- 1번을 통해 BehaviorSubject를 String 타입으로 정의 했고 초기 값으로 "initial Event"를 넣어주었다.
- 2번에 subject에 대해 첫번째 구독자가 구독을 시작했고 구독을 시작한 시점에서 가장 최근 발생한 이벤트는 초기값을 넣어준 시점의 이벤트이므로 이것에 대한 이벤트가 발생한다.
- 3번은 subject에 next이벤트가 발생했고 구독자는 이 이벤트를 감지해 해당 이벤트에 대한 출력을 한다.
- 4번은 두번째 구독자가 구독을 시작했고 이 구독을 한 시점에서 가장 최근의 next 이벤트는 "second Event"를 요소로 가지는 이벤트이므로 이것에 대한 출력을 하게 된다.
-
- 이것을 추가하고 아래의 코드를 추가해보자.

```swift
subject.onCompleted()

subject.subscribe({ event in
    print("Line number is \(#line) and value is", event)
})
-----------------------------------------------
--- Example of: BehaviorSubject ---
Line number is 15 and value is initial Event
Line number is 15 and value is second Event
Line number is 23 and value is second Event
Line number is 29 and value is completed
```

- 위에 코드를 작성하고 하나의 구독자를 추가해주었는데 이 구독자는 이벤트를 출력한다. 이전에 PublisherSubject는 complete이벤트가 발생하면 다음에 구독이 일어나면 complete 이벤트가 재생하는데 BehaviorSubject도 마찬가지로 complete이벤트가 발생하면 재생된다.
- BehaviorSubject는 가장 최근에 어떤 데이터가 발생했는지에 대해 알아보는데 유용하다. 만약 어떠한 값을 최신화 시켜줄 때 그 동안 이전의 데이터를 채워 보여줄 수 있다.
