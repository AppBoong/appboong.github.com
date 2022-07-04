---
title: "[SwiftUI] SwiftUI?"
last_modified_at: 2022-06-30T19:06:00-05:00
toc: true
categories:
  - SwiftUI
tags:
  - SwiftUI
  - Swift
---

# SwiftUI?

## What is SwiftUI?
![swiftui_slogan](/images/2022-06-30-SwiftUI.1/swiftui_slogan.png)
 
[Apple Developer](developer.apple.com/kr/xcode/swiftui/)의 SwiftUI 소개 페이지에서는 SwiftUI를 다음과 같이 소개한다
> SwiftUI에서는 놀랍게도 최소한의 코드만으로 Swift의 성능을 사용하여 모든 Apple 플랫폼 전반에서 멋진 앱을 빌드할 수 있습니다. 어떤 Apple 기기에서나 단 하나의 도구 및 API 세트를 사용하여 모든 사용자에게 더 나은 경험을 제공할 수 있습니다.

SwiftUI는 애플에서 2019년 6월에 있었던 WWDC 2019에서 발표한 UI 프레임워크로, 슬로건은  
 "The Shortest Path to a Great App", "Less code, Better code, EveryWhere" 이다  
말그대로 SwiftUI의 장점은 **적은 코드로 좋은앱을 만들기위한 가장 짧은길**이라는 의미이다  

## Special Feature of SwiftUI

### 1. 선언적 구문(declarative syntax)  

SwiftUI는 기존 UIKit과 인터페이스 빌더를 사용하여 사용자 인터페이스 레이아웃을 설계하고 필요한 동작을 구현하는 것과 다른 방법인 선언적구문를 도입했다  
기존 UIKit은 화면을 구성하는 컴포넌트들(라이브러리 패널에 있는 버튼, 텍스트 필드, 슬라이더 등 화면을 구성하는것들)의 레이아웃과 모양에 대한 복잡한 세부 사항을 직접 설계하는 대신, SwiftUI는 단순하면서 직관적인 구문을 이용해 화면을 만들게 해준다

![example1](/images/2022-06-30-SwiftUI.1/example1.png)

- 애플이 제공하는 공식 SwiftUI 튜토리얼이다 해당 튜토리얼은 따로 정리하도록 하겠다
  
위에서 보이는 것 처럼 레이아웃이 실제로 구축되는 방식의 복잡함에 대해 고민할 필요 없이 선언하는 방식만으로 레이아웃을 생성할 수 있고 프리뷰로 바로 확인이 가능하다(맵은 왜안뜨지....)

또한 레이아웃을 구성하는 컴포넌트들을 선언해주는데 이때 VStack, HStack, Form, List와 같은 매니저 종류를 명시하고 텍스트, 색상, 이벤트 시 호출되는 메서드등을 설정하는 수정자를 사용하면 레아이웃의 위치와 컨스트레인트, 렌더링등을 SwiftUI가 자동으로 처리해준다

```swift
// 기존의 명령형 코드
let button = UIButton(type: .system)
button.setTitle("SwiftUI", for: .normal)
button.setTitleColor(.black, for: .normal)
button.titleLabel?.font = .preferredFont(forTextStyle: .title1)
button.addTarget(self, action: #selector(buttonDidTap(_:)), for: .touchUpInside)
view.addSubview(button)

button.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
])

@objc func buttonDidTap(_ sender: UIButton) {
print("hello, swiftUI!")
}
```

이런 코드가 SwiftUI에선

```swift
// SwiftUI의 선언형 코드
Button(action: {
    print("hello")
}) {
    Text("SwiftUI")
        .font(.title)
        .foregroundColor(.black)
}
```

이렇게 짧아지는 것이다.....


### 2. 데이터 주도적

SwiftUI도 여전히 사용자가 이벤트를 처리하는 로직을 짜야하지만, 앱 데이터와 인터페이스, 로직 사이의 관계에서는 데이터 주도적이다

이전에는 앱내에서 데이터의 현재 값을 검사하려면 코드를 작성해 확인했고, 데이터의 최신 상태를 확인하려 해도 계속 코드를 작성했어야 했다

하지만 SwiftUI에서는 앱의 데이터 모델과 사용자의 인터페이스의 컴포넌트, 그리고 기능을 구현한 로직을 바인딩하는 방법을 이용해 이런 복잡함을 해결했다

물론 기존 UIkit에서 사용하던 RxSwift, RxCocoa라이브러리의 데이터 바인딩의 방식과는 달라 따로 공부가 필요한 부분이지만 데이터 바인딩을 라이브러리에 의존하지않고, 라이브러리를 서서 제공하던 방식보다 더 간편하게 가능해졌기 때문에 굉장한 이점으로 보인다.

나머지 SwiftUI에 관한 내용은 [SwiftUI Tutorials](https://developer.apple.com/tutorials/swiftui/building-lists-and-navigation)를 진행하며 다뤄보겠다

---

## References
[[SwiftUI] SwiftUI란? UIKit와의 차이란? 대체 얼마나 좋길래 개발자분들 모두 칭찬을 하는거지?](https://boidevelop.tistory.com/115)

[[IOS][SWIFTUI] SwiftUI 기본 설명!](https://iosminjae.tistory.com/8)