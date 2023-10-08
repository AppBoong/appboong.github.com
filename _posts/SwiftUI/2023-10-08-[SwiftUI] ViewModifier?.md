---
title: "[SwiftUI] View Modifier?"
last_modified_at: 2023-10-08T19:06:00-05:00
toc: true
categories:
  - SwiftUI
tags:
  - SwiftUI
  - Swift
  - View Modifier
---

# View Modifier?  

## What is View Modifier?
> [Swift 공식문서](https://developer.apple.com/documentation/swiftui/viewmodifier): A modifier that you apply to a view or another view modifier, producing a different version of the original value.
  
번역해보자면 뷰 또는 다른 뷰 수정자에 적용하여 원래 다른 값의 버전을 성하는 수정자 라는 뜻이다

## Custom View Modifier
UI개발을 하다보면 중복되는 디자인과 반복되는 스타일 같은 부분들이 반드시 발생한다. UIKit 에선 SubClassing으로 이를 해결했지만 SiwtUI에서 ViewModifier 를 만들어줌으로서 재사용성 높은 UI코드를 만들 수 있다

> Text에 곡선의 테두리를 넣고, 뒷배경과 라인에 색상을 넣어 박스형태로 만드는 ViewModifier를 만들어보자

```swift
struct RectStyleModifier: ViewModifier {
    var fillColor: Color?
    var lineColor: Color?
    var cornerRadius: CGFloat = 15
    
    func body(content: Content) -> some View {
        content
            .padding(5)
            .background {
                ZStack {
                    if let fillColor {
                        RoundedRectangle(cornerRadius: cornerRadius)
                            .foregroundColor(fillColor)
                    }
                    if let lineColor {
                        RoundedRectangle(cornerRadius: cornerRadius)
                            .strokeBorder(lineColor)
                    }
                }
                
            }
            
    }
}
```
## Usage
위에서 만든 RectStyleModifier를 Text에 적용시켜보자

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "globe")
                .imageScale(.large)
                .foregroundStyle(.tint)
            Text("Hello, world!")
                .modifier(RectStyleModifier(fillColor: .red,
                                            lineColor: .red,
                                            cornerRadius: 15))
        }
        .padding()
    }
}
```

![ViewModifier1](/images/SwiftUI/ViewModifier1.png)

요런식으로 나오게된다

잘 활용하면 중복되는 디자인에 적용하여 생산성을 엄청 높일 수 있겠다
