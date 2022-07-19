# Composition
> composition(합성, 조립)

- 아무리 복잡한 기능도 작은 객체들로 나눠 만들면 좋은 아키텍쳐를 만들수있다
- 다른 디자인패턴들을 배우는데 가장 기본이 되는 패턴이다
- 단일책임원칙을 지킬 수 있게됨
- dry(dont repeat yourself)원칙을 지킬 수 있게됨
- MVC -> massive ViewController, MVVM -> massive ViewModel 이 될수있다 
- 그렇기 때문에 composition을 잘 활용하여 massive 시리즈가 나오지 않게 하는것이 중요

> ***Favor object composition over class inheritance*** - Gang of Four, Design Pattern

> 상속을 가장 잘 이용하는 방법은 상속을 쓰지 않는 것이다 -> 상속은 코드의 가장 강한 결합,  
> 부모의 행동을 거부하는 방법은 리스코프 원칙을 위배하게됨

> ex) CocoaTouch :`UIView`의  `.frame`프로퍼티는 UIView의 상속체들의 위치와 사이즈를 조절할수 있게 해주는데 `UISwitch`는 예외적으로 `.frame`을 통해 사이즈와 위치를 바꿀수 없다 

***Swift는 composition을 잘 지원하는 언어이다***
value type은 상속을 지원하지 않기 때문에 value type의 기능을 확장 위해선 composition을 활용해야한다
ex) `SwiftUI`의 뷰들은 value type 이기 때문에 뷰의 기능을 확장하기 위해선 `viewModifier`를 활용하여 상속없이 기능을 확장해야 한다

```swift

class A: UIViewController {
    func setupViews()
    //add subviews
    }
}

class B: A {
    override func setupViews() {
        super.setupViews()
        //add B View
    }
}
```

```swift
class A: UIViewController {

}

class B: UIViewController {

}

class C: UIViewController {
    private let a: UIViewController
    private let b: UIViewController

    //add a, b child viewcontroller
}
```

위의 코드보다 아래의 코드가 중장기적 관점에서 유지보수가 훨씬쉬운 코드이다 - compose pattern

