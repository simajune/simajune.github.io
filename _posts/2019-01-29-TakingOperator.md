---
layout: post
title: "Taking Operator"
author: "Tejay"
tags: "iOS"
---

# Taking Operator

### Taking Operator

* Taking Operator는 이전에 배운 skip operator와 반대라고 보면 된다.

#### 1. Take

* 아래의 표를 한번 봐보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-3/1.png?raw=true" width="800px"/>

* 표를 보면 take에 써진 매개변수까지 값을 받는다는 뜻인거 같다.
* 코드를 통해 알아보자

```swift
exampleOf(description: "Take") {
  let disposeBag = DisposeBag()
  // 1
  Observable.of(1, 2, 3, 4, 5, 6)
    // 2
    .take(3)
    .subscribe(onNext: {
      print($0)
    })
    .addDisposableTo(disposeBag)
}
-----------------------------------------
--- Example of: Take ---
1
2
3
```

* 위의 코드를 보면 Observable은 총 6개의 이벤트를 가지고 있다.하지만 take라는 operator를 이용한 것이다. take에 사용된 매개변수를 통해 처음의 이벤트에서 매개변수에 적힌 수만큼 이벤트를 받는다.

#### 2. takeWhilWithIndex

* takeWhileWithindex는 방출할 때 이벤트의 요소뿐만 아니라 요소의 index값을 참조하고 싶을 때 사용하는 것이다.
* 아래 마블 다이어그램을 보자.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-3/2.png?raw=true" width="800px"/>

* 위의 그림을 보면 클로저를 통해 두가지를 받을 수 있다. 하나는 이벤트이고 나머지는 이 이벤트에 대한 index 값을 받는 것이다. 이것을 이용해 조건을 만들고 그 조건에 들어야만 이벤트를 받을 수 있다.
* 코드를 통해 알아보자.

```swift
exampleOf(description: "TakeWhilWithIndex") {
  let disposeBag = DisposeBag()

  Observable.of(2, 2, 4, 4, 6, 6)
    .takeWhileWithIndex { int, index in
      int % 2 == 0 && index < 3
    }.subscribe(onNext: {
        print($0)
  }).addDisposableTo(disposeBag)
}
-----------------------------------------
--- Example of: TakeWhilWithIndex ---
2
2
4
```

* 위에 코드를 보면 처음 이벤트부터 위에 요소의 조건과 인덱스의 조건을 만족할때까지 이벤트를 받는다. 하지만 무조건 첫번째 조건부터 성립이 되야한다. 첫번째부터 조건이 안 맞으면 다음 이벤트는 skip된다.

### 3. takeUntil

* 이벤에 배울 takeUntil은 skipWhile하고 반대되는 것이다. 특정 구독자가 구독할 하게 되면 이벤트는 처음부터는 잘 받고 있었지만 그 구독으로 인하여 Observable이 더이상 이벤트를 받지 않는다.
* 그림과 코드를 살펴 보자

​<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-3/3.png?raw=true" width="800px"/>

```swift
exampleOf(description: "TakeUntil") {
  let disposeBag = DisposeBag()
  // 1
  let subject = PublishSubject<String>()
  let trigger = PublishSubject<String>()
  // 2
  subject
    .takeUntil(trigger)
    .subscribe(onNext: {
      print($0)
    })
    .addDisposableTo(disposeBag)
  // 3
  subject.onNext("1")
  subject.onNext("2")
}
-----------------------------------------
--- Example of: TakeUntil ---
1
2
```

* skipUntil과 마찬가지로 trigger역할을 하는 Subject가 존재한다. 그래서 trigger에 따라 이벤트를 언제 받을지 알 수 있다.
