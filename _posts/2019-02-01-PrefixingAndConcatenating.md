---
layout: post
title: "Prefixing and concatenating"
author: "Tejay"
tags: "iOS"
---

# Prefixing and concatenating

### Combining operator

* combining operator를 통해 시퀸스를 조립하고 각각의 시퀸스의 데이터를 결합하는 것을 배울 것이다.

#### 1. startWith

* observable을 통해 작업할 때 가장 필요한 것은 observable이 초기값을 받는 것이 보장되는 것이다.
* "현재 상태"가 먼저 필요한 상황이 있는데 이것의 가장 좋은 예는 "현재 위치"와 "네트워크 연결 상태"이다.
* 아래 그림을 보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch9-1/1.png?raw=true" width="800px"/>

```swift
exampleOf(description: "startWith") {
  let numbers = Observable.of(2, 3, 4)

  let observable = numbers.startWith(1)

  observable.subscribe(onNext: {
    print($0)
  })
}
-----------------------------------------
--- Example of: startWith ---
1
2
3
4
```

* startWith 연산자는 observable 시퀀스에 초기값을 주는 prefix이다.
* 다음으로 빨리 넘어가자. startWith 연산자는 하나의 요소만을 만을 초기값으로 보장하는 것이다.
* 이제부터 concat에 대해 알아보자 startWith연산자도 concat 연산자에 속한다.

#### 2. concat

* 우선 아래 그림과 코드를 보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch9-1/2.png?raw=true" width="800px"/>

```swift
exampleOf(description: "observable.concat") {
  let first = Observable.of(1, 2, 3)
  let second = Observable.of(4, 5, 6)

  let observable = Observable.concat([first, second])

  observable.subscribe(onNext: {
    print($0)
  })  
}
-----------------------------------------
--- Example of: observable.concat ---
1
2
3
4
5
6
```

* concat 연산자를 보면 구독자끼리 연결을 지을 때 좋은 것 같다. startWith는 하나의 요소밖에 초기값을 설정을 못하기 때문에!
* concat을 사용하니 첫번째 시퀀스 1, 2, 3뒤에 두번째 시퀀스 4, 5, 6이 따라왔다.
* observable.concat은 정적 함수이고 그 안에는 배열로 정렬된다. 순차적으로 다음 다음 요소로 이동하고 이동하는 중간에 오류가 발생하면 오류를 내고 종료한다.

```swift
exampleOf(description: "concat") {
  let koreans = Observable.of("태준", "지오", "로이")
  let american = Observable.of("Mike", "Ryan", "Charlotte")

  let observable = koreans.concat(american)

  observable.subscribe(onNext:{
    print($0)
  })
}
-----------------------------------------
--- Example of: concat ---
태준
지오
로이
Mike
Ryan
Charlotte
```

* 위에 코드를 보면 따로 설정해주어도 되지만 첫번째 시퀀스에 concat 연산자를 사용하여 연결 시킬 수 있다.

```swift
exampleOf(description: "concat one element") {
  let numbers = Observable.of(4, 5, 6)

  Observable.just(1).concat(numbers)
    .subscribe(onNext:{
      print($0)
    })
}
-----------------------------------------
--- Example of: concat one element ---
1
4
5
6

```

* just를 사용하여 앞의 초기값을 정하고 그 뒤에 concat을 통해 연결 시킬 수도 있다.
