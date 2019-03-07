---
layout: post
title: "Observable 구독"
author: "Tejay"
tags: "iOS"
---

# Observable 구독

- 우선 iOS 개발자라면 NotificationCenter를 많이 쓰는데 이건 너무 복잡하다. 이걸 사용한 예제이다.

```swift
let observer = NotificationCenter.default.addObserver(
    forName: .UIKeyboardDidChangeFrame,
    object: nil,
    queue: nil) { notification in
// Handle receiving notification
}
```

### 1. Subscribe

- 이건 RxSwift Observable보다 복잡하다.
- Subscribe는 NotificationCenter에서 addObserver()와 같다.
- Observable을 만들어도 구독을 하지 않으면 전혀 사용되지 않는다.

```swift
example(of: "subscribe") {
	let one = 1
	let two = 2
	let three = 3
	let observable = Observable.of(one, two, three)

    observable.subscribe { event in
	print(event)
	}
}
```

- 위의 코드는 Observable을 만들고 구독하는 과정이다.
- 위에 코드를 구독을 하게 되면 결과물은 다음과 같이 발생한다.

```
--- Example of: subscribe ---
next(1)
next(2)
next(3)
completed
```

- observable은 세개의 변수 one, two, three로 이루어져있고, 그것을 구독하여 그 안의 각각의 변수를 next라는 메소드 이벤트를 통해 출력되고 다 끝나고 나면 completed를 방출하여 이 구독은 완전히 끝나게 된다.
- 만약 그 변수의 요소만 보기 원한다면 event.element를 통해 이벤트가 발생할 때의 요소를 방출하게도 할 수 있다.

```swift
observable.subscribe { event in
	print(event.element)
}
-------------------------------
--- Example of: subscribe ---
Optional(1)
Optional(2)
Optional(3)
nil
```

- 하지만 element는 옵셔널로 방출을 하게 되고 이것을 강제로 바인딩 해주니 completed될때 nil로 인해 크래쉬가 발생한다.
- completed 이벤트에서 nil이 발생한다는 것은 completed는 요소를 가지고 있지 않다는 거다.
- 따라서 element만 빼올 때는 guard let이나 if let을 사용하여야 할 것이다.

```swift
observable.subscribe { event in
    if let element = event.element {
      print(element)
    }else {
      print("asdasd")
    }
  }
}
-----------------------------------
--- Example of: subscribe ---
1
2
3
asdasd
```

- 구독할 때는 전에 공부했듯이 3가지의 이벤트가 발생한다. .next, .error, .completed 하지만 구독을 한다고 무조건 이 세가지의 이벤트를 방출되는 것은 아니다.

```swift
observable.subscribe( onNext: { event in
    print(event)
})
```

- 위 코드를 보면 onNext를 추가하므로 인해 error나 completed 이벤트가 발생하더라도 이벤트를 발생시키지 않는다.



### 2. Empty

- 다음으로 볼 것은 **empty** 연산자이다. 이 연산자는 요소가 0일 때를 관찰하는 시퀀스를 만든다. 그리고 이것은 completed 이벤트만 방출한다.

```swift
example(of: "empty") {
  let observable = Observable<Void>.empty()
}
```

- 시퀀스를 만드는 방법은 다음과 같다. **empty** 연산자가 붙는다.
- **empty**는 타입 추론이 안되므로 무조건 타입을 적어줘야 한다.

```swift
observable
    .subscribe(
      onNext: { event in
      print(event)
    },
      onCompleted: {
      print("completed")
    }
)
---------------------------
--- Example of: empty ---
completed
```



### 3. Never

- 위에 **empty** 연산자와 반대되는 것이 있는데 그게 바로 **never** 연산자이다.
- 아무것도 방출하지 않고 끝나지 않습니다.

```swift
example(of: "never") {
  let observable = Observable<Any>.never()
  observable
    .subscribe(
      onNext: { element in
        print(element)
    },
      onCompleted: {
        print("Completed")
    }
  )
}
--- Example of: never ---
```

- 위에 observable은 지금 봐선 ''뭐하러 써?'' 라고 생각이 든다. 하지만 책은 나중에 알려준다니 참고 기다린다.


### 4. Range

```swift
example(of: "range") {
  // 1
  let observable = Observable<Int>.range(start: 1, count: 10)
  observable
    .subscribe(onNext: { i in
      // 2
      print(i)
      let n = Double(i)
      let fibonacci = Int(((pow(1.61803, n) - pow(0.61803, n)) /
        2.23606).rounded())
      print(fibonacci)
    }
  )
}
---------------------------------------------
--- Example of: range ---
0
1
2
3
5
8
13
21
34
55
```

- **range** 연산자는 처음과 끝이 있는 정수 범위를 설정하게 된다. 그리고 이 정수가 올라가면서 이벤트가 발생한다. 위에 코드는 피보나치수열을 1부터 10번째까지 방출하는 것이다.
- 지금까지 배운 연산자 중에서는 never를 제외한 모든 연산자는 정상적으로 completed 이벤트가 발생했다.
