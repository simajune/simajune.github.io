---
layout: post
title: "Observable의 생명주기"
author: "Tejay"
tags: "iOS"
---

# Observable의 생명주기

- 이전의 다이어그램을 보면 Observable은 3개의 요소를 방출한다. Observable이 각각의 요소를 방출할 때마다 장 알려진 **next** 이벤트가 발생하게 된다.
- 아래의 다이어그램을 보자 이 Observable 끝에는 수직으로 표시해 놨다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch2-2/1.png?raw=true" width="800px"/>

- 위에 다이어그램을 보면 Observable이 3개의 요소를 방출하고 난 후 이것은 끝난다. 이 수직점을 **complete** 이벤트라고 한다. 이것은 끝나는 것의 의미를 한다.
- 그래서 Observable이 끝나면 어떠한 요소도 방출을 하지 않는다.
- 하지만 때때로 잘 못되는 경우도 있다. 바로 아래와 같은 상황이다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch2-2/2.png?raw=true" width="800px"/>

- 위의 다이어그램은 Error가 발생한 경우이다. 이것은 X로 표시되어 있다. 에러가 포함된 경우에는 Observable은 **error** 이벤트가 발생하게 된다. 이것은 **complete** 이벤트와 크게 다른 것이  없어보인다. 이것도 Observable이 끝나게 되고 더이상 Observable이 요소를 방출하지 않게 때문이다.
- 간단하게 요약하면 아래와 같다.

  - Observable이 끝나기 전까지는 요소가 포함된 **next**이벤트가 방출된다.
  - Error 발생하면 Observable은 **error**이벤트를 발생하고 종료된다.
  - 아무 이상없이 Observable이 끝나면 **complete**이벤트가 발생한다.

- RxSwift코드를 보게 되면 이 이벤트에 대해서는 열거형으로 정리되어 있다.

```swift
/// Represents a sequence event.
///
/// Sequence grammar:
/// **next\* (error | completed)**
public enum Event<Element> {
	/// Next element is produced.
	case next(Element)
	/// Sequence terminated with an error
	case error(Swift.Error)
    /// Sequence completed successfully.
	case completed
}
```

- 위에 코드를 보면 next 이벤트는 element라는 인스턴스를 가지고 있고 error 이벤트는 Swift.Error 인스턴스를 가지고 있고 complete 이벤트는 어떠한 데이터도 가지고 있지 않는다.
