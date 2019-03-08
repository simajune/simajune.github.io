---
layout: post
title: "Observable Factory 만들기"
author: "Tejay"
tags: "iOS"
---

# Observable Factory 만들기

- observable 시퀀스를 만들고 나서 보통에 경우에는 구독자가 구독을 하기를기다리는데 observable factory는 각각의 구독자에게 새로운 observable 시퀀스를 제공할 수 있다.

```swift
example(of: "deferred") {
	let disposeBag = DisposeBag()
	// 1
	var flip = false
	// 2
	let factory: Observable<Int> = Observable.deferred {
	// 3
    	flip = !flip
		// 4
		if flip {
			return Observable.of(1, 2, 3)
		} else {
 	       return Observable.of(4, 5, 6)
        }
 	}
}
-------------------------------------------------
--- Example of: deferred ---
```

- 위의 코드는 순서는 다음과 같다.
  - flip의 Bool값으로 어떠한 observable 시퀀스를 반환할지 만든다.
  - deferred 연산자를 사용하여 Int를 요소로 갖는 observable factory를 만든다
  - flip을 바꿀 때마다 사용된다.
  - flip이 true, false인지에 따라 반환하는 결과가 다르다.
- 아래 코드를 추가해보자

```swift
for _ in 0...3 {
    factory.subscribe(onNext: {
      print($0, terminator: "")
    })
      .addDisposableTo(disposeBag)
    print()
}
-------------------------------
--- Example of: deferred ---
123
456
123
456
```

- 위에 코드는 총 4번에 걸펴 factory를 구독하는 것이다. 이 구독을 통해 할때마다 반환되는 것이 다르다.
