---
layout: post
title: "Skip Operator"
author: "Tejay"
tags: "iOS"
---

# Skip Operator

### Skip Operator

* 특정 요소의 수가 필요 없을 때가 있다. 그럴 때 사용하는 것이 바로 Skip Operator이다.

#### 1. Skip

* 아래의 마블 다이어그램을 보면 skip에 대해 잘 보여준다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-2/1.png?raw=true" width="800px"/>

* 이것은 해당 매개변수의 수만큼 이벤트를 무시하는 것이다. 그림으로 보면 매개변수가 2이기 때문에 첫번째 두개의 요소만을 무시하고 3번째 요소인 3부터 받게 된다.
* 코드를 통해 알아보자

```swift
exampleOf(description: "skip") {
  let disposeBag = DisposeBag()
  // 1
  Observable.of("A", "B", "C", "D", "E", "F")
    // 2
    .skip(3)
    .subscribe(onNext: {
      print($0)
    })
    .addDisposableTo(disposeBag)
}
-----------------------------------------
--- Example of: skip ---
D
E
F
```

* 위의 코드를 보면 skip(3)을 사용함으로 첫번째부터 3개의 요소를 가진 이벤트를 무시하고 4번째 이벤트부터 받는 것이다. 따라서 결과 값이 D, E, F가 출력된 것이다.

#### 2. SkipWhile

* skipWhile은 filter와 비슷하다. 조건을 만족시킬 때까지 스킵하는 것이다.
* 아래 마블 다이어그램을 보자.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-2/2.png?raw=true" width="800px"/>

* 위의 그림의 조건은 해당 요소가 2로 나누었을 때 1이 나올 때까지 모든 이벤트를 무시하고 1이  나오지 않는 이벤트부터 계속 받게 된다. 이벤트를 받게 될때 조건이 만족되는 경우도 있겠지만 그때는 이 조건은 더이상 효력이 없다. 일단 이벤트를 받기 시작하면 계속 받는다.
* 코드를 통해 알아보자.

```swift
exampleOf(description: "skipWhile") {

  let disposeBag = DisposeBag()
  // 1
  Observable.of(2, 2, 3, 4, 5, 6, 4, 34, 5, 8, 11, 17)
    // 2
    .skipWhile { integer in
      integer % 2 == 0 || integer % 3 == 0
    }
    .subscribe(onNext: {
      print($0)
    })
    .addDisposableTo(disposeBag)
}
-----------------------------------------
--- Example of: skipWhile ---
5
6
4
34
5
8
11
17
```

* 위의 연산자 skipWhile에 조건을 두가지를 걸어 주었다. 2로 나누어 떨어지거나 3으로 나누어 떨어지는 수는 무시하게 했다. 이런 조건이다보니 처음부터 2, 2, 3, 4는 모두 주건에 만족하기 때문에 skip되었고 5부터 조건이 만족하지 않기 때문에 통과되어 이벤트가 발생한다.

#### 3. SkipUntil

* 지금까지의 조건은 정적인 조건이었다. 이번에 배울 것인 동적 조건을 통해 필터링을 할 것이다.
* 이것을 하기위해선 sklipUntil을 사용하는데 이것은 다른 트리거 Observable이 방출되기 전까지 skip하는 것이다.
* 아래 그림을 보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-2/3.png?raw=true" width="800px"/>

* 위의 그림을 보면 중간의 트리거가 발생하기전까지는 skip하다가 트리거가 발생한 후에는 이벤트가 발생한다.
* 코드를 통해 알아보자.

```swift
exampleOf(description: "skipUntil") {
  let disposeBag = DisposeBag()
  // 1
  let subject = PublishSubject<String>()
  let trigger = PublishSubject<String>()
  // 2
  subject
    .skipUntil(trigger)
    .subscribe(onNext: {
      print($0)
    })
    .addDisposableTo(disposeBag)

  subject.onNext("1")
  subject.onNext("2")
  subject.onNext("3")
  subject.onNext("4")

  trigger.onNext("trigger")

  subject.onNext("5")
}
-----------------------------------------
--- Example of: skipUntil ---
5
```

* 위에 코드를 작성해 보았다.

* 두개의 subject를 만들었다. subject는 원하는 일을 하는 것 다른 subject(trigger)는 앞의 subject를 어떻게 변화시킬지에 대한 것.

* 그래서 trigger의 역할에 따라 필터의 내용이 달라진다. 즉 tigger가 어디에서 발생하는지에 따라 하는 역할이 바뀔 것이다.

* 위에 보면 trigger가 발생되기 전에는 모든 이벤트를 skip하기로 되어 있고 그래서 결과 값도 앞에서 subject에 next 이벤트가 4번이나 발생했지만 trigger가 발생하지 않아 결국 5만 출력되게 된다.
