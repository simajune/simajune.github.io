---
layout: post
title: "Distinct Operator"
author: "Tejay"
tags: "iOS"
---

# Distinct Operator

### Distinct Operator

* Distinct operator는 연속으로 중복된 요소를 받지 않는 연산자이다.

#### 1. distinctUntilChanged

* 아래의 표를 한번 봐보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-4/1.png?raw=true" width="800px"/>

* 표를 보면 연속으로 중복된 요소가 들어오면 그 요소는 넘어가고 다른 요소가 들어와야 해당 요소가 방출된다.
* 코드를 통해 알아보자

```swift
exampleOf(description: "distinctUntilChanged") {
  let disposeBag = DisposeBag()
  // 1
  Observable.of("A", "A", "B", "B", "A")
    // 2
    .distinctUntilChanged()
    .subscribe(onNext: {
      print($0)
    })
    .addDisposableTo(disposeBag)
}
-----------------------------------------
--- Example of: distinctUntilChanged ---
A
B
A
```

* 위의 코드를 보면 Observable은 String을 데이터 타입으로 가진다. distinctUntilChanged 연산자를 사용하면 연속된 요소는 무시한다. 따라서 위의 요소에서 출력되는 것은 "A", "B", "A"이다.

#### 2. distinctUntilChanged

* distinctUntilChanged는 클로저 형태로도 쓰인다. 두개의 요소를 받고 그 요소 2개를 이용하여 연산자를 만들 수 있다.
* 아래 마블 다이어그램을 보자.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch5-4/2.png?raw=true" width="800px"/>

* 위의 그림을 보면 클로저를 통해 요소 2개를 이용해  값을 비교하여 첫번째 요소와 두번째 요소가 같으면 첫번째 요소를 방출하는 연산자이다.
* 코드를 통해 알아보자.

```swift
exampleOf(description: "distinctUntilChanged") {
  let disposeBag = DisposeBag()
  // 1
  let formatter = NumberFormatter()
  formatter.numberStyle = .spellOut
  // 2
  Observable<NSNumber>.of(10, 110, 20, 200, 210, 310)
    // 3
    .distinctUntilChanged { a, b in
      // 4
      guard let aWords = formatter.string(from: a)?.components(separatedBy: " "),
        let bWords = formatter.string(from: b)?.components(separatedBy: " ")
          else {
          return false
      }
      var containsMatch = false
      // 5
      for aWord in aWords {
        for bWord in bWords {
          if aWord == bWord {
            containsMatch = true
            break
          }
        }
      }
      return containsMatch
    }
    // 4
    .subscribe(onNext: {
      print($0)
    })
    .addDisposableTo(disposeBag)
}
-----------------------------------------
--- Example of: distinctUntilChanged ---
10
20
200
```

* 위에 코드를 보면 숫자를 스펠링으로 바꾸는 formatter를 사용하여 첫번째 요소와 두번째 요소를 비교해서 같은 문자가 있을 경우 첫번쨰 요소를 방출하게 했다. 따라서 아래와 같은 출력의 결과가 나왔다.
