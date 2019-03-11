---
layout: post
title: "PublisherSubjects란"
author: "Tejay"
tags: "iOS"
---

# PublisherSubjects란

- PublisherSubject는 구독이 끝나거나 completed 이벤트와 error이벤트로 끝날 때까지 구독자가 구독한 시점부터 새로운 이벤트를 알리가 원할 때 사용된다.
- 아래 그래프를 보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch3-2/1.png?raw=true" width="800px"/>

* 첫번째 구독자는 1번 이벤트가 발생한 후에 구독을 시작했기 때문에 1번에 대한 이벤트를 받을 수 없고 구독 이후에 발생한 2번과 3번의 이벤트만 받을 수 있다.
* 두번째 구독자는 2번 이벤트가 발생하기 전까지 구독을 하지 않고 그 이후에 구독을 했기 때문에 3번의 이벤트만 받을 수 있다.

```swift
let subscriptionTwo = subject
  .subscribe { event in
	print("2)", event.element ?? event
)}
```

- 전에 작성했던 코드에 두번째 구독자를 추가했다. 이 구독자는 이벤트가 발생시 "2)"를 출력하고 그뒤에 이벤트의 요소 또는 요소가 없을 경우 이벤트에 대해 출력을 한다.
- 이것을 추가하고 아래의 코드를 추가해보자.

```swift
subject.onNext("3")
-----------------------------------------------
3
2) 3
```

- 위에 코드를 작성한 뒤 출력되는 값을 확인해보면 두 줄로 출력이 된다. 원인은 구독자가 두명이기 때문이다. 첫번째 줄은 첫번째 구독자를 통해 출력된 값이고 두번째 줄은 두번째 구독자에 대한 출력값이다.
- 그리고 아래의 코드를 추가해보자.

```swift
subscriptionOne.dispose()
subject.onNext("4")
-----------------------------------------------
2) 4
```

- 첫번째 줄은 첫번째 구독자가 dispose하고 그 다음 "4"의 요소를 가지는 이벤트가 발생했다.
- 그런 후 출력된 값을 보면 한줄이 출력된 것을 볼 수 있고 이것은 두번째 구독자에 대한 출력된 값이다.
- 첫번째 구독자는 더이상 이벤트를 받지 못하게 된다.

```swift
// 1
subject.onCompleted()
// 2
subject.onNext("5")
// 3
subscriptionTwo.dispose()
let disposeBag = DisposeBag()
// 4
subject
  .subscribe {
	print("3)", $0.element ?? $0)
  }
  .disposed(by: disposeBag)
subject.onNext("?")
-----------------------------------------------
2) completed
3) completed
```

- 위에 코드를 추가했을때 이 이벤트가 발생했을 때 출력하는 값을 보게되면 일단 1번에 subject가 onCompleted()가 되면서 이벤트가 발생하고 두번째 구독자는 값을 출력하는데 onCompleted()는 요소가 없기 때문에 이벤트를 출력한다. 따라서 "2) onCompleted"라는 값을 출력한다.
- .complete 이벤트가 발생했기 때문에 두번째 구독자는 더이상 이벤트를 받지 못한다. 그래서 다음에 "5"라는 요소를 가진 이벤트가 발생했지만 값을 출력하지는 않는다.
- 그 이후에 두번때 구독을 완전히 처분하고 subject에 대해 새로운 구독을 하고 이것은 이벤트 발생시 "3)"과 이벤트 요소 또는 요소가 없는 경우 이벤트를 출력하고 바로 처분되는 코드를 추가했다. 하지만 이 구독은 다음에 발생하는 이벤트에 대해 반응을 하지 않고 생성 되자마자 이벤트가 발생한다. 왜냐하면 이번에 subject가 complete 이벤트로 인해 완료가 되었기 때문이다. 더 이상 이 subject는 추가적인 이벤트를 받지 못한다.
