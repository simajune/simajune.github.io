---
layout: post
title: "Ignoring Operator"
author: "Tejay"
tags: "iOS"
---

# Ignoring Operator

### Filtering Operator란

- 이번 장에서는 .next 이벤트에 조건부를 걸어 사용할 수 있는 필터링 연산자에 대해 알아볼 것이다.
- 필터링 연산자는 가입자가 처리하고자하는 요소만을 수신하도록 하는 것이다.



### ignoring Operator

#### 1. IgnoreElements

* 아래의 마블 다이어그램을 보면 IgnoreElement에 대해 잘 보여준다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-1/1.png?raw=true" width="800px"/>

* 간단히 말로 표현하면 .next 이벤트 요소를 무시하는 것이다. 그러나 Error나 Complete와 같은 정지 이벤트를 허용한다.
* 코드를 통해 알아보자

```swift
exampleOf(description: "IgnoreElements") {
  // 1
  let strikes = PublishSubject<String>()
  let disposeBag = DisposeBag()
  // 2
  strikes
    .ignoreElements()
    .subscribe { _ in
      print("You're out!")
    }
    .addDisposableTo(disposeBag)
}
-----------------------------------------
--- Example of: IgnoreElements ---
```

* strike Subject를 만들고
* 모든 strike의 이벤트를 구독하지만 .ignoreElement를 사용하기 때문에 .next이벤트가 전부다 무시된다.
* ignoreElement는 Observable이 complete나 error이벤트를 통해 종료될 때 단지 알리는데 유용하다.

```swift
strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")
strikes.onCompleted()
-----------------------------------------
--- Example of: IgnoreElements ---
You're out!
```

* 위에 코드를 보면 .next 이벤트에 대해서는 모두 다 무시 되다가 Complete이벤트에 대해서는 반응하여 "You're out!"이란 문자열을 출력한다.

#### 2. ElementAt

* 이번에 배울 것은 ElementAt이란 연산자인데 이것은 다 무시하지만 ElementAt의 매개변수인 index 값에 해당하는 것 만은 무시하지 않는다.
* 아래 마블 다이어그램을 보자.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-1/2.png?raw=true" width="800px"/>

* 위의 그림을 보면 elementAt(1)을 사용한다. 이것을 사용함으로 index가 1인 것만을 받고 다른 것은 모두 다 무시한다.
* 코드를 통해 알아보자.

```swift
exampleOf(description: "elementAt") {
  // 1
  let strikes = PublishSubject<String>()
  let disposeBag = DisposeBag()
  //  2
  strikes
    .elementAt(2)
    .subscribe(onNext: { element in
      print(element)
      print("You're out!")
    })
    .addDisposableTo(disposeBag)

  strikes.onNext("1")
  strikes.onNext("2")
  strikes.onNext("3")
  strikes.onNext("4")
  strikes.onNext("5")
}
-----------------------------------------
--- Example of: elementAt ---
3
You're out!
```

* 위의 코드를 보면 연산자 ElementAt(2)을 사용하여 세번째의 이벤트만 받겠다고 정의를 한 후 그 이벤트가 발생했을 시에는 해당 이벤트의 요소값과 문자열 "You're out!"을 출력하게 했다.
* 이것에 대한 결과는 총 5번의 next이벤트가 발생했지만 3번째 이벤트인 "3"을 요소로 가지는 이벤트만 받고 나머지 이벤트는 모두 무시했다.

#### 3. Filter

* 앞에서 배운 ignoreElement와 ElementAt은 Observable의 요소를 필터링하여 방출한다.
* 그 조건이 한가지 일리는 없다. 한가지 이상인 경우 사용하는 것이 filter이다.
* 각 요소에 적용되는 조건부 클로저를 통해 참인 경우만 방출한다.
* 아래 그림을 보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-1/3.png?raw=true" width="800px"/>

* 그림에서 보면 filter의 조건을 3보다 작은 것만 통과 시키게 했다. 따라서 1,2,3 중 1,2만 통과되고 3은 조건에 만족하지 않기 때문에 통과 되지 않는다.
* 코드를 통해 알아보자.

```swift
exampleOf(description: "filter") {
  let disposeBag = DisposeBag()
  // 1
  Observable.of(1, 2, 3, 4, 5, 6)
    // 2
    .filter { int in
      int % 2 == 0 && int % 3 == 0
    }
    // 3
    .subscribe(onNext: {
      print($0)
    })
    .addDisposableTo(disposeBag)
}
-----------------------------------------
--- Example of: filter ---
6
```

* 위에 코드를 작성해 보았다.

* 원래는 하나의 조건만 했다. 해당 요소가 짝수(2의 배수)인것만 통과시키게했다. 그것을 통해 결과는 2,4,6이란 요소가 통과되어 방출 됐는데 하나의 조건 3의 배수인것을 추가 시켜 6의 배수만 통과 시키게 하여 6이란 요소를 얻게 되었다.
