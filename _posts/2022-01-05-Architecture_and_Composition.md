---
layout: post
title: "아키텍처와 Composition"
author: "Tejay"
tags: "iOS"
---

## 아키텍처와 Composition

- 객체를 조립해서 쓰는 방식을 **Composition**이라고 합니다. 로직을 여러 곳으로 분산시킨 후 재조립해 쓰면 많은 장점을 얻을 수 있습니다. iOS와 Swift, 그리고 잘 알려진 여러 아키텍처에서는 어떻게 Composition을 활용하고 있는자  아봅니다.

### Composition (합성, 조합)
- A way to combine objects and data types into more complex ones
- 결국 어떤 아키텍처가 중요한 것이 아니라 객체를 어떻게 쪼개서 사용하느냐가 중요한 것이다. (Ex.Massive ViewController, Massive RIBs...)
- 로직을 분산 시킨 다음 합쳐서 원하는 것을 구현
- 유지보수에 용이
- 테스트에도 용이(public API, 파라미터 적음)

### 상속이 항상 좋은 것은 아니다?!
- 유연성이 떨어짐
  * 코드의 가장 강한 결합 형태이기 때문에 원치 않는 기능들도 들어올 수 있음.
  * 예) UIView의 Frame 프로퍼티 (UISwitch는 예외)

### 객체 Composition
<img src="https://simajune.github.io/img/posting/Architecture_and_Composition01.png" width="300px" height="450px"/>

```swift
class A: UIViewController {
  func setupViews() {
    // add subviews
  }
}

class B: A {
  override func setupViews() {
    super.setupViews()
    // add B View
  }
}
```
- 화면 구현을 하다보면 위와 같은 경우의 화면이 대부분일 것이다. 이러한 화면을 구현할 때 대부분 위에 코드처럼 사용하게 될 것이다. 하지만 이것은 중장기적으로 보았을 때는 적절하지 못하다. 앞으로 추가될 때마다 A와 B와의 관계가 끈끈하기 때문에 추가되는 작업을 할때 고려해야 할 것들이 많아진다. 이것을 Composition으로 바꿔보면 아래와 같다.

```swift
class A: UIViewController {

}

class B: UIViewController {

}

class C: UIViewController {
  private let a: UIViewController
  private let b: UIViewController

  // add a,b to child viewcontroller
}
```

이렇게 A와 B 화면을 각각 객체를 가지는 새로운 화면 C를 만들어 사용하게 되면 이전에 구현된 A와 B의 관계를 좀더 유연하게 활용할 수 있다.

### 함수의 Composition

##### 1. map 함수

```swift
import UIKit

[1, 2, 3].map { $0 + 1 }
-> [2, 3, 4]

[1, 2, 3].map { $0 + 1 }.map { "만 \($0) 살" }
-> ["만 2 살", "만 3 살", "만 4 살"]

let num: Int? = 1
num.map { $0 + 1 }
-> Optional(2)

let num: Int? = nil
num.map { $0 + 1 }
-> nil

let myResult: Result<Int, Error> = .success(2)
myResult.map { $0 + 1 }
-> success(3)


// 1. Generic 타입
// enum Optional<Wrapped>
// associatedtype Element
// enum Result<Success, Failure> where Failure : Error
// associatedtype Output

// 2. transform 함수를 인자로 받음

// t: A -> B
// F<A> -(map)-> F<B>

let ageString: String? = "10"
let result4 = ageString.map { Int($0) }
// result4의 타입은? -> Int??
// t: A -> Option<B>
// 왜? 원래의 Optional<A> -(map)-> Optional<Option<B>>
// 하지만 Int()의 이니셜라이즈는 옵셔널이기때문에 두번의 옵셔널처리가 됨
// 이것을 사용하기 위해서는

if let x = ageString.map(Int.init), let y = x {
  // y 사용
}

if case let .some(.some(x)) = ageString.map(Int.init) {
  // x 사용
}

if case let x?? = ageString.map(Int.init) {
  // x 사용
}

// flatMap사용

let result5 = ageString.flatMap { Int.init }
// t: A -> Option<B>
// Optional<A> -(flatMap)-> Optional<B>


// UIEvent -> IndexPath -> Model -> URL -> Data -> Model -> ViewModel -> ViewModel

// 예시

struct MyModel: Decodable {
  let name: String
}

let myLabel: UILabel()

if let data = UserDefaults.standard.data(forKey: "my_data_key") {
  if let model = try? JSONDecoder().decode(MyModel.self, form: data) {
    let welcomeMessage = "Hello \(model.name)"
    myLabel.text = welcomeMessage
  }
}

UserDefaults.standard.data(forKey: "my_data_key")
  .flatMap { try? JSONDecoder.decode(MyModel.self, from: $0) }
  .map(\.name)
  .map { "Hello \($0)" }

myLabel.text = welcomeMessage
```

### 모듈 Composition

<img src="https://simajune.github.io/img/posting/Architecture_and_Composition02.png" width="600px" height="400px"/>

- 복잡한 기능이 단 하나의 모듈 안에 있다면 내부 객체들끼리 혼잡한 참조 관계에 매우 취약
- 택시 호출을 위한 모듈만 해도 유저의 위치, 결제, 쿠폰 적용, 백엔드와의 통신 등 기능이 굉장히 많음

<img src="https://simajune.github.io/img/posting/Architecture_and_Composition03.png" width="600px" height="400px"/>

- 이걸 독립적인 모듈로 나눠보면 복잡한 참고 관계를 많이 해소할 수 있음
- public으로 공개되지 않은 부분은 다른 모듈에서 아예 접근조차 어렵기 때문에 사이드 이펙트의 영향도 덜함
- public interface만 봐도 모듈 내부의 코드를 보지 않아도 무슨 역할을 하는지 쉽게 파악
- UI도 각 모듈이 자신이 맡은 역할만 바꾼다면 모듈 내에서 빠르게 변동 가능

### 마무리
- Composition은 앱 전반적인 구조를 담당하는 아키텍처 패턴에서도 핵심적인 요소임

##### Redux 기반 아키텍처의 Composition

<img src="https://simajune.github.io/img/posting/Architecture_and_Composition04.png" width="600px" height="400px"/>

- Redux 기반의 아키텍처(The Composition Architecture, ReSwift, etc)를 예를 들어보면 이 아키텍처는 Reducer, Store, State, Action, View 요소로 로직을 분산 시킴
- Reducer: 상태를 업데이트
- Action: 유저가 발생시킨 액션
- 앱이 커질수록 Reducer의 로직이 많아질 수 밖에 없음
- 하지만 subReducer를 통해 분리를 시켜서 다시 조합하면 쉽게 유지보수할 수 있음.
- Action도 마찬가지로 subState로 분리

<img src="https://simajune.github.io/img/posting/Architecture_and_Composition06.png" width="600px" height="400px"/>

##### Interactor, Router 기반 아키텍처의 Composition

<img src="https://simajune.github.io/img/posting/Architecture_and_Composition06.png" width="600px" height="400px"/>
- Interactor, Router 기반 아키텍처(RIBs, VIPER)는 Builder, Router, Interactor, Presenter, View로 로직을 분리 시킴. 이 하나의 단위를 리블이라고 함
- 리블렛은 한 화면 전체를 담당할 수도 있고 리브를 담당할 수도 있고 화면이 없을 수도 있음
- 리블렛은 여러개의 child 리블렛으로 분리가 가능

##### 아키텍처에서 Composition이 중요한 이유
- 앱이 아무리 커지고 복잡해져 결국 맨 마지막에는 이해하기 쉬운 작은 요소들로 분해가 됨
- 코드 이해가 쉬움
- 원하는 코드를 찾기 쉬움
