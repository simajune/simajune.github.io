---
layout: post
title: "ReplaySubjects란"
author: "Tejay"
tags: "iOS"
---

# ReplaySubjects란

- ReplaySubject는 구독 할 때 버퍼의 사이즈를 통해 이전의 데이터를  받는다.
- 아래 그래프를 보자

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch3-4/1.png?raw=true" width="800px"/>

* 이 그래프는 버퍼 사이즈가 2인 ReplaySubject이다.
* 첫번째 구독자는 처음부터 구독을 시작했기 때문에 이 구독자는 기존의 값과 동일하게 움직인다.
* 두번째 구독자는 2번 이후에 구독을 했지만 버퍼 사이즈가 2이므로 구독 시점에서 1과 2가 리플레이 된다.
* 아래 코드를 보자

```swift
example(of: "ReplaySubject") {
  // 1
  let subject = ReplaySubject<String>.create(bufferSize: 1)
  let disposeBag = DisposeBag()
  // 2
  subject.onNext("1")
  subject.onNext("2")

  subject
    .subscribe {
      print((#line), $0)
    }
    .addDisposableTo(disposeBag)

  subject.onNext("3")
  // 3

  subject
    .subscribe {
      print((#line), $0)
    }
    .addDisposableTo(disposeBag)
}
---------------------------------------------
--- Example of: ReplaySubject ---
43 next(2)
43 next(3)
52 next(3)
```

- 버퍼 사이즈가 1인 데이타 타입이 String인 ReplaySubject를 생성한 후 "1" 이벤트와 "2"이벤트가 밸생 후에 첫번째 구독자가 구독을 한다. 이때 버퍼사이즈가 1이기때문에 이전에 발생한 이벤트 "2"를 리플레이한다. 그리고 "3"이벤트가 발생하여 아래 출력에서 이 구독자에 대한 라인과 어떤 이벤트가 발생했는지 보여준다.
- 그 다음 "3"이벤트가 발생 후 두번째 구독자가 구독을 하기 시작하면 가장 최근에 발생한 "3"이벤트를 리플레이하게 된다.
- 그 다음 아래의 코드를 추가해 보자.

```swift
subject.onError(MyError.anError)
---------------------------------------------
--- Example of: ReplaySubject ---
43 next(2)
43 next(3)
52 next(3)
43 error(anError)
52 error(anError)
```

- 위에 결과가 나온 것은 뭐 당연히 에러가 발생하면 이것을 구독자는 감지하기 때문에 에러에 대한 것을 출력한다.
- 만약 중간에 subject가 disposed되면 어떻게 될까?
- 아래의 코드를 두번째 구독자 전에 추가해봤다.

```swift
subject.dispose()
---------------------------------------------
--- Example of: ReplaySubject ---
43 next(2)
43 next(3)
53 error(Object `RxSwift.(unknown context at 0x11fd20dd0).ReplayOne<Swift.String>` was already disposed.)
```

- 위와 같이 출력이 된다. 이미 subject는 끝났기 때문에 구독을 할수 없다는 뜻이다.
