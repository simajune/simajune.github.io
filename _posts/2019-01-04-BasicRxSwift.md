---
layout: post
title: "RxSwift의 기초"
author: "Tejay"
tags: "iOS"
---

# RxSwift의 기초



### 1. RxSwift

- RxSwift는 새로운 컨셉이 아니다. 상당히 오랫동안 다루어져왔지만 최근 10년에 결쳐 급격하게 만들어져왔다.
- Rx를 쓰는 이유는 이 한마디면 될거 같다. 복잡한 비동기를 다루기 위함이다.

- RxSwift의 중요한 세개는 바로 **Observables**, **Operators**, **Schedulers** 이다.



### 2. Observables

- Observable <T>  클래스는 Rx의 기초중에 기초이다. 이것은 불변의 데이터 T를 저장하여 비동기적으로 옮기는 이벤트를 하는 시퀀스이다. 간단하게 말하면 클래스이고 구독을 하게 되면 다른 클래스로 값을 옮길 수 있다.
- Observable <T>  클래스 하나 또는 많은 관찰자가 실시간으로 발생하는 이벤트와 UI의 업데이트에 반응한다. 그리고 새로운 데이터와 들어노는 데이터를 처리하거나 활용한다.
- ObservableType의 프로토콜은 매우 간단하게 3가지의 이벤트 타입만 방출할 수 있다.

  - A next event: 이 이벤트는 가장 최신(또는 다음)의 값을 던지고 관찰자는 값을 받는 역할을 한다.
  - A completed event: 이 이벤트는 이벤트 시퀀스가 성공하여 종료될 때 발생한다. 이것은 Observable이 성공하여 완료되었을 때 발생하고 그 이외에는 발생하지 않는다.
  - A error event: 이 이벤트는 에러가 발생하거나 값을 방출하지 못했을 경우 발생한다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch1-2/1.png?raw=true" width="800px"/>

- 위의 그림은 시간 순차적으로 이벤트가 발생할 때 정수값을 방출하는 Observable을 시각화한 것이다.

- Observable은 Obvervable또는 Observer의 본질에 대해 가정을 하지 않는다. 따라서 이제는 더이상 delegate나 클로저를 쓸 필요가 없다.



<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch1-2/2.png?raw=true" width="800px"/>

- 실 상황에 대한 아이디어를 얻기 위해 2가지의 다른 Observable을 살펴보자.

  **1. 유한한 Observable 시퀀스**

  - 몇몇의 Observable은 0개, 1개 또는 그 이상의 값을 방출하고 나중에 성공적으로 종료되거나 오류를 발생하여 종료가 된다.
  - iOS 앱에서 인터넷으로 파일을 다운받을 때의 코드를 생각해보자
    - 첫째로 들어오는 데이터를 다운로드를 시작하고 관찰한다.
    - 그 후에는 들어오는 파일의 부분들을 데이터의 덩어리를 반복적으로 받는다.
    - 인터넷 연결이 끊기면 다운로드는 종료되고 에러와 함꼐 타임아웃이 발생할 것이다.
    - 또는, 만약 파일의 모든 데이터가 다운완료되면, 성공과 함께 완료될 것이다.
  - 이 과정은 분명하게 형식작인 life cycle이다. 이것을 코드로 나타내면 아래와 같다

  ```swift
  API.download(file: "http://www...")
    .subscribe(onNext: { data in
      ... append data to temporary file
    },
    onError: { error in
      ... display error to user
    },
    onCompleted: {
      ... use downloaded file
    })
  ```


#### 무한한 Observable 시퀀스

- 파일을 다운로드하거나 이와 유사하게 자연스럽거나 강제적으로 종료되는 것과 달리 무한한 것과 비슷한 Obervable 시퀀스가 있다. 종종, UI 이벤트가 무한한 Observable 시퀀이다.
- 예를 들어, 앱에서 디바이스의 방향에 반응해야 할 때를 생각해보자.
  - 너는 노티피케이션 센터를 통해 **UIDeviceOrientationDidChange** 노티를 관찰하는 클래스를 추가해야 한다.
  - 그 후에 방향 변화를 다루는 콜백 메소드가 필요하다. 이것은 디바이스 현재 방향을 잡고 최신 값에 반응한다.
- 이 디바이스 방향에 대한 시퀀스는 끝이 없다. 방향을 바꾸지 않아서 이벤트가 발생하지 않을 수는 있지만 그럼에도 Oberver는 계속해서 이벤트를 기다리고 있는 중이기 때문이다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch1-2/3.png?raw=true" width="800px"/>

- 위에 내용을 RxSwift 코드로 나타내면 아래와 같다.

  ```swift
  UIDevice.rx.orientation
    .subscribe(onNext: { current in
  switch current {
  case .landscape:
          ... re-arrange UI for landscape
  case .portrait:
          ... re-arrange UI for portrait
      }
    })
  ```


### 3. Operators

- ObservableType과 Observable 클래스는 많은 복잡한 로직을 구현하는 비동기적 작업을 추상하는 많은 메소드을 가지고 있다. 그들은 매우 분리되어 있고 구성가능하기 때문에 이러한 것을 **Operators**라고 한다. 이 **Operators**는 대게 비동 입력을 가지고 있고 부작용없이 출력만 만들기 때문에 퍼즐 조각처럼 쉽고 더 큰 그림을 만들 수 있다.
- 예를 들어 수학적인 표현인 (5 + 6) * 10 - 2를 보자.
- 분명하게 결정론적이고 사칙연산이라는 미리 정의된 명령대로 입력하여 결과를 얻을 수 있다.
- 이와 유사하게 **Rx Operators**도 Obervable을 통해 입력과 출력을 최종 값이 표현될 때까지 결정적으로 처리할 수 있다. 그런 다음 최종 결과 값을 사용하여 부작용을 유발할 수 있다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch1-2/4.png?raw=true" width="800px"/>

```swift
UIDevice.rx.orientation
  .filter { value in
return value != .landscape
  }
  .map { _ in
return "Portrait is the best!"
  }
  .subscribe(onNext: { string in
    showAlert(text: string)
  })
```



- 위의 그림은 **Rx Operators**를 사용하도록 조정된 방향 변화를 관찰하는 예제이다.

- 첫째 filter는 .lanscape가 아닌 값만을 통과시킨다. 만약 디바이스의 모드가 landscape라면 이 구독은 더이상 실행될 수 없다. 왜냐하면 filter는 이 이베느를 억제할 것이기 때문이다.
- .portrait값의 경우는 map을 해 이 방향의 타입의 입력이 "Portrait is the best!"이란 String타입으로 변환되어 출력 될것이다.
- 마지막으로 next 이벤트를 구독하여 위에서 발생한 String을 가지는 Alert 메세지를 스크린에 보여줄 것이다.
- **Operators**는 항상 구성 가능하다. 즉, 항상 데이터를 입력으로 가져와서 결과로 출력을 한다. 그래서 단일연산자보다 훨씬 쉽게 그것들을 많은 방법으로 연결할 수 있다.

- 그러나 실제로는 병렬로 실행되는 코드를 작성하는 것은 다소 복잡하다. 왜냐하면 각각 병렬로 실행되는 코드가 어떤것이 먼저 끝나고 업데이트되는지 알기 어렵기 때문이다. 이걸 해결하기 위해서 대부분은 프로젝트에서 애플이 제공하는 비동기 API를 많이 사용하고 아래의 것을 사용했을 것이다.
  - NotificationCenter: 사용자의 기기가 방향을 바꾸거나 키보드와 같은 이벤트가 발생할 때마다 사용했을 것이다.
  - Delegate: 클래스와 클래스간의 언제 끝날지 모를 때 많이 사용한다.
  - Grand Central Dispatch: 직렬 대기열에서 순차적으로 실행되어야 할 때나 동시에 많은 작업이 필요할 떄 사용한다.
  - Closure: 클래스간에 전달할 수 있는 분리된 코드 조각을 만들어 클래스가 코드를 실행할지 여부, 횟수 등을 결정할때 사용한다.
- 일반적인 클래스의 대부분은 비동기적이고 모든 UI 요소들도 비동기이기 때문에 앱 코드 전체가 어떤 순서로 실행하는 아는 것은 불가능하다.결국 앱은 유저의 입력, 네트워크 활동 또는 다른 OS의 이벤트와 같이 외부적인 요소에 따라 다르게 실행된다. 아마 앱이 실행될 때마다 이런 외부적인 요소들 때문에 코드가 순서가 다를 것이다.
- 엄청 읽었는데 결국 좋은 비동기는 만들기 어렵다 라는걸 알리는 글이었다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch1-1/Asynchronous.png?raw=true" width ="800px"/>

#### 비동기 프로그래밍 용어

**1. 상태, 특히 공유된 mutable(가변) 상태**

- 노트북을 예로 들면 노트북은 처음사용할 때랑 몇일 아니 계속 사용하다보면 느려지거나 멈출 때가 있다. 노트북의 하드웨어나 소프트웨어와 같은 스펙은 그대로인데 말이다. 이것은 바로 상태 때문인데 이 상태는 노트북의 메모리에 있는 데이터, 디스크에 저장된 데이터, 네트워크를 통해 남은 흔적들을 말한다.



**2.  명령형 프로그래밍**

- 명령형 프로그래밍은 간단히 말하면 명령을 내려서 상태를 변경하는 것이다. 따라서 매우 명확하다.
- 명령형 코드는 컴퓨터가 이해할 수 있는 코드와 유사하다. 하지만 이 명령형 코드는 위에서 말한 공유된 가변 상태인 경우가 포함됐을 때는 비동기 프로그래밍을 쓰는 것은 매우 어렵다.

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    setupUI()
    connectUIControls()
    createDataSource()
    listenForChanges()
}
```

- 위에 경우를 보면 많은 메소드를 순차적으로 사용했다. 하지만 순차적이기만 하지 매우 불안한 코드이다. 누군가 이 순서를 바꾸게 되면 이 코드는 완전히 다르게 작동을 할 수 있기때문이다.



**3. 부작용**

- 위에서 살펴보면 비동기 프로그래밍의 부작용은 딱 2가지로 이부분을 통해 문제를 해결할 수 있다.
- 부작용은 현재 범위 외부의 상태로 변경된다. 위의 코드를 예로 들어보면 마지막 부분을 생각해보면 connectUIControls()는 일종의 이벤트를 UI 구성요소에 연결하는 것이다. 하지만 이것이 실행된 후 상태가 변경이 되면 연결된 이벤트의 동작은 다르게 바뀌게 된다.
- 디스크에 저장된 데이터를 수정하거나 스크린에 있는 Label에 텍스트를 업데이트할 때마다 부작용은 발생하게 된다.
- 부작용을 발생하는 문제는 컨트롤하는 방법 내에서 발생한다. 우리는 어떤 코드가 부작용을 일으키고 데이터를 단순히 처리하고 출력하는지 결정할 수 있어야 한다.
- RxSwift는 이것을 해결하기 위해 존재한다.



**4. 선언 코드**

- 명령형 프로그래밍에서는 하고 싶은대로 상태를 변경할 수 있다. 함수형 프로그래밍에서  부작용은 없다. RxSwift는 명령형과 함수형 프로그래밍을 결합한 것이다.
- 선언 코드를 사용하여 동작을 정의할 수 있고, RxSwift는 관련된 이벤트가 있을 때마다 불변의 고립된 데이터의 입력을 제공한다.
- 이 방법을 통해 비동기 코드를 사용할 수 있지만 for 루프와 같은 역할을 하게 만든다. 즉 변경 불가능한 데이터로 작업하고  순차적이고 결정론적인 방식의 코드를 실행하는 것이다.
- 이 부분은 읽기만 해서는 이해가 잘 되지 않는다. 앞으로 더 공부하면서 알게 되겠지....



**5. 반응형 시스템**

- 반응형 시스템은 추상적은 용어이다. 그리고 아래의 특성을 대부분 또는 전부 포함하는 웹 또는 iOS앱을 포함한다.
  - Responsive(반응): 항상 UI의 상태를 최신 상태로 유지한다.
  - Resilient(복원성): 각각의 동작은 격리되어 정의되고 유연한 에러 복구를 제공한다.
  - Elastic(탄성?): 코드는 다양한 작업을 다루고, 종종 lazy 기반의 데이터를 수집하고 이벤트를 조절하고 자료를 공유하는 기능을 한다.
  - 메세지 기반: 구성요소는 재사용성과 고립 그리고 라이프사이클과 클래스 분리를 향상시키는 메세지 기반의 통신을 사용한다.



#### 4. SChedulers

- **Schedulers는** Rx에서 dispatch queue와 같은 기능을 한다.
- **Schedulers는** 굳이 사용하지 않아도 미리 정의된 것으로 커버 가능하다.
- **Scehdulers는** 매우 강력하다. 예를 들어 GrandDispatch를 사용하는 GrandDispatchQueueScheduler에 주어진 큐에서 코드 실행을 딕렬화하는 이벤트를 관찰하도록 지정할 수 있다. ConcurrentDispatchQueueScheduler는 코드를 동시에 실행할 수 있다.
- OperationQueueScheduler를 사용하면 주어진 NSOperationQueue에서 구독 일정을 잡을 수 있다.
- RxSwift로 인해 다른 구독자에게 동일한 구독의 여러 작업을 예약할 수 있다.

<img src="https://github.com/simajune/RxSwift/blob/master/Documents/Ch1-2/5.png?raw=true" width="800px"/>

- 위에 다이어그램을 읽으려면 색칠된 부분을 순서대로 따라가면 됩니다. 다른 **Schedulers** 에 걸쳐 1,2,3이 실행된다.
  - 파란 네트워크의 코드 (1)은 Schduler의 기반한 커스텀 NSOperation로 작동합니다.
  - 이 블럭에 의해 출력된 데이터는 동시 백그라운드 GDC 대기열에 있는 다른 스케쥴러에서 실행되는 다음 블럭(2)의 입력으로 사용된다.
  - 마지막으로 파란색 코드(3)의 마지막 부분은 UI를 새 데이터로 업데이트하기 위해 주 쓰레드 Scheduler에서 예약된다.
