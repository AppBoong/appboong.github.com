---
title:  "[Architecture] RIBs?"
last_modified_at: 2022-06-28T19:06:00-05:00
toc: true
categories:
  - Architecture
tags:
  - Swift
  - RIBs
---

# RIBs?

## What is RIBs?
> ***Router + Interactor + Builder***  
> [RIBs](https://github.com/uber/RIBs)란 우버에서 만든 크로스플랫폼 모바일 아키텍쳐 프레임워크다  

- 기존의 디자인 패턴들이 비즈니스 로직만 있는 (뷰가 없는) 모듈의 구현이 어려웠고,
이점을 보완하여 Uber에서 VIPER를 개조하여 만든것이 RIBs이다  
- 화면 로직이나, 비즈니스 로직을 RIB(리블렛)이라는 모듈로 작성한다  
- RIB에게 UI요소(view, presenter)는 옵셔널이다 (비즈니스 로직만 가진 모듈이 가능하다는 뜻)  
- RIB들을 부모자식 관계로 이어서 논리적인 트리 구조로 앱을 작성한다.  
- **앱의 상태가 뷰가 아닌, 현재 active된 RIB이 무엇인가에 따라 결정된다**

> ***RIB는 모듈의 단위인데 립의 갈빗대를 리블렛이라고 부르기떄문에 하나의 RIB는 리블렛이라고도 부른다***

## Why RIBs?

> **Encourage Cross-Platform Collaboration**  

- RIBs를 사용하면 iOS, Android가 서로 공동으로 디자인된 단일 아키텍쳐를 공유하기 때문에 각 팀이 비즈니스 로직을 교차검토 할 수 있는 장점이 있다
  
> **Minimize Global States and Decisions** 
- Global state변경은 예측할 수 없는 동작을 유발하며, 이걸 변경했다고 해서 이 변경이 미칠 영향들을 엔지니어가 전부 알 수 없다. RIBs는 잘 격리된(well-isolated) 개별 RIBs의 deep hierarchy내에서 상태를 캡슐화하여, Global state 문제를 방지하도록 권장한다  
   
> **Testability and Isolation**
- 개별 RIB클래스에는 별도의 "책임"이 있는데, 이 책임에는 라우팅, 비즈니스 로직, 뷰 로직, 다른 RIB 클래스 생성 등이 있다. 부모 RIB 논리는 자식 RIB 논리와 분리되어(decoupled) 있다. 따라서 RIB 클래스를 쉽게 테스트하고 독립적으로 추론할 수 있다  

> **Tooling for Developer Productivity**
- non-trivial(?) Architecture 패턴을 채택한다고 해서 강력한 툴링(tooling) 없이는 소규모 어플리케이션을 넘어서 확장되지 않는다. RIBs에는 코드 생성, 정적 분석 및 runtime integrations에 대한 IDE 툴링이 함께 제공되며, 이 툴은 크고 작은 팀의 개발자 생산성을 향상시킨다

> **Open-Closed Principle**
- 개발자는 가능하면 기존 코드를 수정하지 않고 새로운 기능을 추가 할 수 있어야 한다. RIBs를 사용하면 몇 군데서 볼 수 있다. 예를들어 부모 RIB을 거의 변경하지 않고 부모의 종속성이 필요한 복잡한 자식 RIB을 attach하거나 build할 수 있다.

> **Structured around Business Logic**
- 앱의 비즈니스 로직 구조는 UI의 구조를 엄격하게(또는 절대적으로) 반영할 필요는 없다. 예를들어 애니메이션 및 view 성능을 용이하게하기 위해서 View 계층 구조는 RIB 계층 구조보다 더 얕을 수(shallower) 있다. 
또는 단일 기능 RIB이 UI의 다른 위치에 나타나는 세가지(?)View의 모양을 제어할 수 있다.

> **Explicit Contracts**
- 요구사항은 compile-time safe contracts(컴파일 타임 안전 계약?)으로 선언해야한다.
클래스 종속성과 순서(ordering) 종속성이 충족되지 않으면 클래스는 컴파일되지 않아야 한다. RIBs는 ReactiveX를 사용하여 순서 의존성을 나타내며, 클래스 의존성을 나타내기 위해 type safe dependency injection (DI) 시스템과 data invariants의 생성을 장려하기 위해 많은 DI scopes를 나타낸다.


## Elements of RIBs

![RIB parts](/images/2022-06-28-RIBs/RIB%20parts.png)

RIBs의 요소들을 살펴보기 전에 [ModernRIBs](https://github.com/DevYeom/ModernRIBs)를 프로젝트에 추가시켜 주면 RIBs의 템플릿을 쉽게 추가시킬수 있다

![ModernRIBs](/images/2022-06-28-RIBs/ModernRIBsTemplate.png)

command + N 으로 새로운 파일 추가를 하게되면 하나의 리블렛을 추가할수 있는 템플릿이 나온다  
(해당 템플릿을 쓰려면 shell script를 추가해야한다)

![ModernRIBsTemplateCreate](/images/2022-06-28-RIBs/ModernRIBsTemplateCreate.png)

추가할 리블렛의 이름을 정해주면

![ModernRIBsTemplateAdd](/images/2022-06-28-RIBs/ModernRIBsTemplateAdd.png)

하나의 리블렛이 쉽게추가된다

### 1.Router
- RIB간의 전환 ( attach, detach )을 담당하며, 앱의 논리적 트리구조를 형성한다. 

- Router가 자식RIB을 detach하면, detach된 자식RIB의 Interactor은 deactivate 된다. 반대도 마찬가지다.

- Builder에게 RIB 생성을 명령하고, 반환된 Routing을 이용해서 Routing한다.

- Interacter에게 Routing해달라고 요청받는다.

- didLoad라는 커스텀 생명주기를 가지고 있다. 비지니스 로직만을 가진 RIB이 load된 후, View를 가진 자식RIB으로 Routing 하는 등. 무언가 해야할 때 필요하기 때문.

> **example code**  

```swift
protocol HomeInteractable: Interactable {
    var router: HomeRouting? { get set }
    var listener: HomeListener? { get set }
}

protocol HomeViewControllable: ViewControllable {
    // TODO: Declare methods the router invokes to manipulate the view hierarchy.
}

final class HomeRouter: ViewableRouter<HomeInteractable, HomeViewControllable>, HomeRouting {

    // TODO: Constructor inject child builder protocols to allow building children.
    override init(interactor: HomeInteractable, viewController: HomeViewControllable) {
        super.init(interactor: interactor, viewController: viewController)
        interactor.router = self
    }
}

```

### 2. Interactor
- 비지니스 로직, 데이터 관리

- 다른 RIB들간의 통신은 오직 Interactor에서 한다. ( 부모에게 전달할 때는 Listener, 자식에게 전달할 때는 Rx Observable )

- Presenter에게 데이터나 로직을 요청받는다.

- Router에게 Routing을 요청한다.

- Presenter에게 데이터를 전달한다.

- didBecomeActive, willResignActive 같은 활성화 관련 생명주기를 소유하고 있다. Router에 의해 활성화 되었을 때 무언가 해야할 때 필요하기 때문.

> **example code**  

```swift
protocol HomeRouting: ViewableRouting {
    // TODO: Declare methods the interactor can invoke to manage sub-tree via the router.
}

protocol HomePresentable: Presentable {
    var listener: HomePresentableListener? { get set }
    // TODO: Declare methods the interactor can invoke the presenter to present data.
}

protocol HomeListener: AnyObject {
    // TODO: Declare methods the interactor can invoke to communicate with other RIBs.
}

final class HomeInteractor: PresentableInteractor<HomePresentable>, HomeInteractable, HomePresentableListener {

    weak var router: HomeRouting?
    weak var listener: HomeListener?

    // TODO: Add additional dependencies to constructor. Do not perform any logic
    // in constructor.
    override init(presenter: HomePresentable) {
        super.init(presenter: presenter)
        presenter.listener = self
    }

    override func didBecomeActive() {
        super.didBecomeActive()
        // TODO: Implement business logic here.
    }

    override func willResignActive() {
        super.willResignActive()
        // TODO: Pause any business logic.
    }
}
```

### 3. Builder
- 부모RIB의 Router로 부터 자식RIB생성을 위해 호출됨.

- 구성요소들의 생성 및 DI(외부 의존성은 Component라는 형태로 받음.) Component를 사용하여 빌드한다

- Routing(Router가 네비게이션 할 때 필요한 것을 정의한 프로토콜)을 리턴하며, 필요에 따라 interater를 listner, actionableItem형태로 함께 반환하기도 한다

> **example code**  


```swift
protocol HomeBuildable: Buildable {
    func build(withListener listener: HomeListener) -> HomeRouting
}

final class HomeBuilder: Builder<HomeDependency>, HomeBuildable {

    override init(dependency: HomeDependency) {
        super.init(dependency: dependency)
    }

    func build(withListener listener: HomeListener) -> HomeRouting {
        let component = HomeComponent(dependency: dependency)
        let viewController = HomeViewController()
        let interactor = HomeInteractor(presenter: viewController)
        interactor.listener = listener
        return HomeRouter(interactor: interactor, viewController: viewController)
    }
}
```

### 4. View(Optional)
- UI요소들. ViewController 포함.

- Presenter에게 유저 이벤트를 넘긴다.

- Presenter에게 ViewModel을 받아서 update한다.

- ViewController의 생명주기는 UI요소만 신경쓴다.

> **example code** 

```swift
protocol HomePresentableListener: AnyObject {
    // TODO: Declare properties and methods that the view controller can invoke to perform
    // business logic, such as signIn(). This protocol is implemented by the corresponding
    // interactor class.
}

final class HomeViewController: UIViewController, HomePresentable, HomeViewControllable {

    weak var listener: HomePresentableListener?
}
```

### 5. Presenter(Optional)
- View와 Interactor의 중재자.

- Interactor에게 전달받은 데이터모델을 ViewModel로 변환하여 View에게 전달하는 Stateless class이다.

- Presenter이 없는데 View가 있을 경우, ViewModel 생성은 Interacter나 View에서 담당한다.

- View에게 유저 이벤트를 전달받고 Interactor에게 필요한 데이터나 로직을 요청한다.

> **eample code**

```swift
protocol HomePresentable: Presentable {
    var listener: HomePresentableListener? { get set }
    // TODO: Declare methods the interactor can invoke the presenter to present data.
}
```

### 6. Component
- 부모RIB의 Builder가 자식RIB의 Builder에 DI하는데 필요한 것을 정의한 프로토콜
  
> **eample code**

```swift
protocol HomeDependency: Dependency {
    // TODO: Declare the set of dependencies required by this RIB, but cannot be
    // created by this RIB.
}

final class HomeComponent: Component<HomeDependency> {

    // TODO: Declare 'fileprivate' dependencies that are only used by this RIB.
}
```

## Strength & Weakness of RIBs

- **Strength**
  
    - 기존의 MVC, MVP, MVVM, VIPER는 아키텍처패턴(디자인 패턴)이고 RIBs는 프레임워크 이다  
    앞서 나열된 디자인 패턴들로 설계된 앱과 달리 RIBs는 View Tree가 아닌 비즈니스 로직이 앱을 주도한다  
    즉 화면위주에서 로직위주로 패러다임의 변화가 있다는 뜻이다

    - RIB에서 ViewController는 옵셔널이기 때문에, 새로운 생명주기가 등장한다.  
    (didLoad, didBecomeActive, willResignActive 등) -> ViewController의 생명주기는 오직 UI만 담당하게 된다.

    - 높은 Mockabilty를 위한 프로토콜 지향 프로그래밍을 사용한다.  
    프레임워크단에서 구성요소들에 필요한 것들을 모두 프로토콜화 해놨다.  
    (RIB간의 attach, detach나 생명주기 관리 등 이미 다 프로토콜화 해놨음.)

- **Weakness**
  
    - 화면위주에서 로직위주로 패러다임 변환이 있기 때문에 러닝커브가 제법 크다.  
    (프레임워크에서 정의해둔 프로토콜이 매우매우 많기 때문에, 해당 용어들의 의미와, 실행 흐름의 선행 공부가 필수적이다.)
    - 화면 기반이 아니기 때문에, 타인이 짠 소스의 의도 파악은 더 어렵다.  
    ( 머릿속에서 그려지는 형태가 없기 때문. )  
    물론, 논리 트리를 어떻게 구성했는지 참고 자료를 남겨두면 괜찮다.
    - 프로젝트에 전체적으로 적용되었을 때 힘을 발휘하는 프레임워크이다.  
    기존 프로젝트를 부분적으로 컨버팅하는 것은 무의미하고, 오히려 더 어렵다. 적용할 거면 처음부터...

---

## References

[Swfit)RIBs - EEYatHo iOS](https://eeyatho.tistory.com/79)

[RIBs란?](https://zeddios.tistory.com/937)