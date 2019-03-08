---
layout: post
title: "Disposing과 Terminating"
author: "Tejay"
tags: "iOS"
---

# Disposing과 Terminating

- observable 시퀀스는 구독을 하지 않으면 절대 사용되지 않는다.

- 보통의 일반적인 구독은 .next, .error, .completed 이벤트가 발생하고 .error나 .completed 이벤트가 발생해야 구독이 종료된다. 하지만 이번에 배울 것은 수동으로 구독을 끝낼 수 있다.


### 1. Dispose

```swift
example(of: "dispose") {
	// 1
	let observable = Observable.of("A", "B", "C")
	// 2
	let subscription = observable.subscribe { event in
	// 3
	print(event)
  	}
}
-----------------------------------------------
--- Example of: dispose ---
next(A)
next(B)
next(C)
completed
```

- 위의 코드는 순서는 다음과 같다.
  - 변수 "A", "B", "C"를 요소로 가지는 observable 시퀀스를 만든다.
  - 이 시퀀스를 구독하여 이벤트가 방출될 때마다 그 이벤트를 출력한다.
- 위에 코드를 구독을 취소하기 위해서는 명시적으로 dispose()를 호출하면 된다.

```swift
subscription.dispose()
```



### 2. DisposeBag

- Dispose는 개별적으로 구독을 관리하는 것이므로 코드가 복잡하거나 많아지면 지루하다.
- RxSwift에는 DisposeBag유형이 있다. 이것은 일반적으로 .addDisposableTo () 메소드를 사용하여 객체에 dipose를 사용할지에 대해 추가하고 구독을 취소할 때 DisposeBag을 이용하여 dispose() 메소드를 부른다.

```swift
example(of: "DisposeBag") {
  // 1
  let disposeBag = DisposeBag()
  // 2
  Observable.of("A", "B", "C")
    .subscribe { // 3
      print($0)
    }
    .addDisposableTo(disposeBag) // 4
}
```

- 가장 많이 사용하는 패턴이다. 이것을 사용하는 것은 메모리 누수에 관하여다. 이것을 사용하지 않는다면 어디선가 개발자도 모르게  계속적으로 종료하지 않는 시퀀스가 있을 수도 있기 때문이다.



### 3. Create

- 이전에는 .next 이벤트 요소가 있는 observable을 만들었는데 이번에는 구독에게 내뿜는 모든 이벤트를 지정하는 다른 방법인 create연산자에 대해 알아볼 것이다.
- create 연산자는 구독자에게 방출할 모든 이벤트를 정의할 수 있다.

```swift
example(of: "create") {
  let disposeBag = DisposeBag()

  Observable<String>.create { observer in
    // 1
    observer.onNext("1")
    // 2
    observer.onCompleted()
    // 3
    observer.onNext("?")
    // 4
    return Disposables.create()
  }
}
----------------------------------
```

- 중요한 것은 마지막에 Disposable을 반환한다는 것이다. 여기에서는 Disposable.create()로 비어 있는 disposable이지만 몇몇 disposable은 부작용도 있다.
- 위에 코드를 보면 두번째 onNext의 요소인 **"?"**가 구독자에게 방출될거 같지만 이것이 발생하기 전에 onCompleted()를 통해 구독이 끝나므로 두번째 onNext 는 구독자에 방출되지 않는다.

```swift
.subscribe(
      onNext: { print($0) },
      onError: { print($0) },
      onCompleted: { print("Completed") },
      onDisposed: { print("Disposed") }
    )
    .addDisposableTo(disposeBag)
-------------------------------------------
--- Example of: create ---
1
Completed
Disposed
```

- 위에서 만든 observable을 구독을 하게 되면 첫번째 요소인 "1"이 출력되고 정상적으로 completed되고 disposed되었기 때문에 나머지 부분도 출력이 된다.

```swift
observable.subscribe( onNext: { event in
    print(event)
})
```

- 위 코드를 보면 onNext를 추가하므로 인해 error나 completed 이벤트가 발생하더라도 이벤트를 발생시키지 않는다.
- 여기에 Error가 추가해보자

```swift
example(of: "create") {
  let disposeBag = DisposeBag()

  enum myError: Error {
    case anError
  }

  Observable<String>.create { observer in
    // 1
    observer.onNext("1")
    // 2
    observer.onError(myError.anError)
    observer.onCompleted()
    // 3
    observer.onNext("?")
    // 4
    return Disposables.create()
  }
    .subscribe(
      onNext: { print($0) },
      onError: { print($0) },
      onCompleted: { print("Completed") },
      onDisposed: { print("Disposed") }
    )
    .addDisposableTo(disposeBag)
}
---------------------------------------------
--- Example of: create ---
1
anError
Disposed
```

- Completed되기 전에 에러를 발생시켰다. 이 경우엔 에러가 발생하여 구독이 종료가 되므로 onCompleted()는 방출되지 않는다.
- 그리고 구독 후에 만약 에러와 completed가 없다면 .addDisposableTo(disposeBag)를 꼭 해주어야 하며 이걸 하지 않을 경우 메모리 누수가 생기고 이 구독은 영원히 끝나지 않게된다.
