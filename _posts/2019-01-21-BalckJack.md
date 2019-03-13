---
layout: post
title: "BlackJack"
author: "Tejay"
tags: "iOS"
---

# Challenge

- 블랙잭을 구현하는 코드를 Subject를 이용하여 만들어 보자.
- 일단 아래의 코드를에 추가를 할 것이다.

```swift
exampleOf(description: "PublishSubject") {
  let disposeBag = DisposeBag()

  let dealtHand = PublishSubject<[(String, Int)]>()

  func deal(_ cardCount: UInt) {
    var deck = cards
    var cardsRemaining: UInt32 = 52
    var hand = [(String, Int)]()

    for _ in 0..<cardCount {
      let randomIndex = Int(arc4random_uniform(cardsRemaining))
      hand.append(deck[randomIndex])
      deck.remove(at: randomIndex)
      cardsRemaining -= 1
    }
    // Add code to update dealtHand here

  }
  // Add subscription to dealtHand here

  deal(3)
}
```

* 구현하는 조건은 다음과 같다.

  * 메인 Playground에 코멘트 두곳에 추가하면 된다. 처음 코멘트에는 dealhand라는 곳에서 points 메소드를 통해 결과 값을 반환한다. 만약 결과가 21이 넘는 경우 에러를 발생시키고 .next이벤트를 추가한다.
  * 두번쨰 코멘트에는 dealhand를 구독하고 이곳에서는 .next 이벤트에서는 points와 cardString의 반환값의 String값을 출력하고 에러가 발생한 경우에는 에러만 출력한다.

* 위의 조건을 통해 첫번째 코멘트에 작성한 코드이다.

```swift
if(points(for: hand) <= 21){
    dealtHand.onNext(hand)
}else{
	dealtHand.onError(HandError.busted)
}
```

* 왜냐하면 일단 3개의 카드를 뽑고 그것에 대한 결과값을 나타내는 함수인 points의 조건을 걸어 21보다 작거나 같으면 next 이벤트를 발생시키게 하고 21을 넘으면 에러를 방출한다.
* 두번째 코드엔 다음과 같은 코드를 작성했다.

```swift
dealtHand.subscribe(onNext: { element in
    print(element)
}, onError: { error in
    print(error)
}).disposed(by: disposeBag)
```

* 두번째 코드에는 위의 dealHand Subject를 구독하고 그 구동에 발생하는 next이벤트와 error이벤트에 대한 처리를 해주었다. next 이벤트가 발생하면 발생한 event의 대한 값을 출력하도록 해주었고 error가 발생하면 그 에러가 어떤 에러에 대한 것인지 출력을 하게 했다.
