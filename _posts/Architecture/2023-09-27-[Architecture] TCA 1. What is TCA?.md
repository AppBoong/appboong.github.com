---
title:  "[TCA] 1. What is TCA??"
last_modified_at: 2023-09-27T19:06:00-05:00
toc: true
categories:
  - Architecture
tags:
  - Swift
  - SwiftUI
  - TCA
---

# TCA?(The Composable Architecture)  

> TCA는 단방향 아키텍쳐로서, 기존의 단방향 아키텍쳐인 Redux의 영향을 많이 받았다  
> Swift의 프레임워크로는 ReSwift와 ReactorKit과 비슷하다
## 단방향 아키텍쳐란(Uni-directional Flow, Architecture) ?
>많은 최신 앱 프레임워크에서 사용되는 디자인 원칙 및 아키텍처 패턴이다. 핵심 아이디어는 데이터가 애플리케이션을 통해 일반적으로 단일 진실 공급원(SSOT:Single Source Of Truth)[애플리케이션 상태]에서 사용자 인터페이스로 한 방향으로 흐른다는 것이다.
- 단방향 아키텍쳐에 관해선 따로 알아보겠다  

### TCA
![TCA1](/images/TCA/TCA1.png)
[출처] https://motosw3600.tistory.com/63
### MVC
![MVC](/images/TCA/MVC.png)
[출처] https://developer.mozilla.org/ko/docs/Glossary/MVC

TCA 와 MVC 의 도식을 보고 비교해봤을때의 차이점은  
TCA의 데이터 흐름에 양방향으로 겹치는 화살표가 없다는 것이다.
  
즉 MVC에서는 State가 여러 곳에서 영향을 받게되며 규모가 커질수록 side effect 때문에 발생하는 오류를 예측할 수 없고, 디버깅도 어려워진다. 반면 TCA와 같은 단방향 아키텍쳐들은 State에 전달되는 변경사항들이 한 방향으로만 수정된다. 따라서 개발자가 State의 흐름만 인지한 채로 코드를 짜기 때문에 디버깅도 쉬워지고 코드 정리가 잘 되는 것이다

## Why TCA?
기존의 단방향 아키텍쳐에는 대표적으로 ReactorKit 이 있는데 ReactorKit은 UIKit을 활용해 앱을 만들때 많이 사용되고 TCA는 SwiftUI와 활용될때 더 빛을 내게 된다.  
그리고 기본적으로 TCA는 Combine의 의존성을, ReactorKit은 RxSwift의 의존성을 갖고있기 때문에 SwiftUI에겐 TCA가 더욱 잘 어울릴 수 밖에 없다.

### TCA에서 강조되는 3가지 원칙
#### Composition(합성)
- 큰 기능을 작은 부분으로 쪼개서, 격리된 모듈로 분리시킬 수 있고, 또한 이 모듈을 다시 합쳐서 큰 기능을 작은 컴포넌트의 집합으로 구성하는 방법을 제공한다.

#### Testing(테스트)
- 아키텍처에 포함된 기능만 테스트하는 게 아니라, 많은 부분으로 구성된 기능에 대한 통합적 테스트가 가능하다.
- 사이드 이펙트가 앱에 어떤 영향을 줄지, 비즈니스 로직이 개발자가 예상한 대로 작동하는지 보장할 수 있게 해주는 end-to-end 테스트가 가능하다.

#### Ergonomic(인체공학)
- 위에서 설명한 합성, 테스팅에서의 특징을 심플한 API 만으로 달성할 수 있도록 하여 생산적인 개발 경험을 제공하는 것을 목표로 한다

> TCA의 중요 요소들에는 State, Action, Reducer, Store, Environment 등이 있다.
다음 글에서 이들에 대해 자세히 다뤄보도록 하겠다.


### Reference
https://medium.com/@Jager-yoo/swiftui-the-composable-architecture-tca-%EC%9E%85%EB%AC%B8-95b69b8c6c16