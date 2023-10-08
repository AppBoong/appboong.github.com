---
title: "[SwiftUI] Property Wrapper?"
last_modified_at: 2022-07-06T19:06:00-05:00
toc: true
categories:
  - SwiftUI
tags:
  - SwiftUI
  - Swift
  - Property Wrapper
---

# Property Wrapper?

## What is Property Wrapper?

> [Swift 공식문서](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties): **A property wrapper** adds a layer of separation between code that manages how a property is stored and the code that defines a property.

번역해보자면 Property Wrapper는 프로퍼티가 저장되는 방식을 관리하는 코드와 프로퍼티가 정의되는 코드 사이에 분리된 계층을 추가해준다는 뜻이다

WWDC 19에서 사용한 UserDefaults예제로 활용법을 살펴보자
```swift
class UserManager {

    static var usesTouchID: Bool {  
        get { return UserDefaults.standard.bool(forKey: "usesTouchID") }
        set { UserDefaults.standard.set(newValue, forKey: "usesTouchID") }
    }

    static var myEmail: String? {  
        get { return UserDefaults.standard.string(forKey: "myEmail") }
        set { UserDefaults.standard.set(newValue, forKey: "myEmail") }
    }

    static var isLoggedIn: Bool {  
        get { return UserDefaults.standard.bool(forKey: "isLoggedIn") }
        set { UserDefaults.standard.set(newValue, forKey: "isLoggedIn") }
    }
}
```
get/set 부분의 코드가 중복돼있는걸 볼수있다

Property Wrapper를 활용하여 코드를 개선해보면

```swift
@propertyWrapper
struct UserDefault<T> {

    let key: String
    let defaultValue: T

    var wrappedValue: T {
        get { UserDefaults.standard.object(forKey: self.key) as? T ?? self.defaultValue }
        set { UserDefaults.standard.set(newValue, forKey: self.key) }
    }
} // 선언

class UserManager {

    @UserDefault(key: "usesTouchID", defaultValue: false)
    static var usesTouchID: Bool

    @UserDefault(key: "myEmail", defaultValue: nil)
    static var myEmail: String?

    @UserDefault(key: "isLoggedIn", defaultValue: false)
    static var isLoggedIn: Bool    
} // 사용
```
이렇게 Property Wrapper는 반복되는 로직들을 프로퍼티 자체에 연결 시켜줄 수 있다

그럼 이제 SwiftUI에서 사용하는 Property Wrapper들을 알아보자

## What kind?

### 1. @State

`@State` 는 특정 프로퍼티를 뷰의 상태(state)로 만들어준다. 즉 이 프로퍼티가 변경되면 자동으로 뷰의 데이터도 변경되고, 뷰의 데이터를 바꿔도 이 프로퍼티의 데이터도 자동으로 변경된다

```swift
struct ContentView: View {
    @State private var name = "World"

    var body: some View {
        VStack {
            Text("Hello, \(name)!")
                .padding()
            Button(
                action: { self.switchName() },
                label: { Text("Switch") }
            )
        }
    }

    func switchName() {
        if name == "World" {
            name = "Universe"
        } else {
            name = "World"
        }
    }
}
```

위의 예제는 버튼을 누르면 프로퍼티의 내용이 바뀌는데 이때 텍스트 뷰의 내용도 자동으로 바뀌는 것을 볼 수 있다

### 2. @Binding
`@Binding`은 다른 인스턴스 소유의 `@State` 프로퍼티를 빌려올 때 사용한다
```swift
struct MyToggleButton: View {
    @Binding var value: Bool

    var body: some View {
        Button(action: {
            self.value.toggle()
        }, label: {
            Text(self.value ? "Hello" : "World")
        })
    }
}

struct ContentView: View {
    @State private var value = false

    var body: some View {
        VStack {
            MyToggleButton(value: $value)
        }
    }
}
```
위의 예제에서 MyToggleButton 의 value 프로퍼티가 @Binding 으로 선언되어 있다. 그리고 이 프로퍼티는 나중에 ContentView 에서 뷰를 생성할 때 value 프로퍼티와 연결된다.

따라서 이 두 값은 연결되기 때문에 어느 한 쪽의 값이 바뀌면 다른 한 쪽도 값이 동일하게 바뀐다. 또한 뷰도 이 데이터의 변경을 알아채고 역시 알아서 업데이트된다.

### 3. @ObservedObject
`@State `의 대표적인 단점은 Value 타입에서만 사용이 가능하다는 점이 다. 즉 클래스 오브젝트의 경우는 `@State` 나 `@Binding` 이 불가능하다. 대신 이 경우 `@ObservableObject` 를 상속받은 클래스의 프로퍼티에 @ObservedObject 라는 Property Wrapper 를 적용해 비슷하게 뷰와 프로퍼티를 연결할 수 있다.

```swift
class MyData: ObservableObject {
    @Published var name = "World"
    @Published var buttonTitle = "Switch to Universe"

    func switchName() {
        if name == "World" {
            name = "Universe"
            buttonTitle = "Switch to World"
        } else {
            name = "World"
            buttonTitle = "Switch to Universe"
        }
    }
}

struct ContentView: View {
    @ObservedObject var data = MyData()

    var body: some View {
        VStack {
            Text("Hello, \(data.name)!")
                .padding()
            Button(
                action: { self.data.switchName() },
                label: { Text(self.data.buttonTitle) }
            )
        }
    }
}
```
다만 클래스의 모든 프로퍼티의 변화를 추적하지는 않는다. 위의 예에서 볼 수 있다시피 추적을 원하는 프로퍼티는 @Published 라는 Property Wrapper를 적용해야 한다.

### 4. @EnvironmentObject
`@EnvironmentObject` 의 경우 오브젝트라는 이름이 붙은 것처럼 클래스 오브젝트를 추적하기 위한 용도의 Property Wrapper다. 다만 차이가 있다면 공유 인스턴스 형태에 적합하게 사용할 수 있다는 점이 있다.

```swift
class SharedData: ObservableObject {
    @Published var configName = "default"
    ...
}

struct ContentView: View {
    @EnvironmentObject var sharedData: SharedData
    ...
}

struct FooView: View {
    @EnvironmentObject var sharedData: SharedData
    ...
}
```
위의 경우 ObservableObject 를 상속받은 클래스를 여러 뷰에서 @EnvironmentObject 형식으로 참조하는 것을 볼 수 있다. 따라서 이름처럼 환경설정 등 여러 곳에서 공유될 만한 데이터를 관리하는 모델로 사용하기 좋다.

다만 최초 생성을 참조가 시작되기 전에 되어야만 할 것이다. 보통은 해당 뷰를 만들기 전에 오브젝트를 생성하고 이걸 environmentObject() 로 알려주어야 한다.

```swift
var sharedData = SharedData()
...
window.rootViewController =
    UIHostingController(rootView: ContentView().environmentObject(sharedData))
```

위 코드가 SharedData 오브젝트를 생성해서 공유를 시작하는 시점이다. 이 코드를 어디에 만들어야 하나 궁금할 수 있는데, SceneDelegate.swift 라는 파일이 보인다면 이 파일 안에서 찾아보자. 아마도 비슷한 곳을 찾을 수 있을 것이다.

---

## References
[SwiftUI Property Wrappers](https://seorenn.github.io/note/swiftui-property-wrappers.html)

[Property Wrapper](https://zeddios.tistory.com/1221)