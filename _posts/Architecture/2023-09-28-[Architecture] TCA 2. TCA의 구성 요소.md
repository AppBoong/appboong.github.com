---
title:  "[TCA] 2. TCA의 구성요소"
last_modified_at: 2023-09-28T19:06:00-05:00
toc: true
categories:
  - Architecture
tags:
  - Swift
  - SwiftUI
  - TCA
---

# TCA의 구성요소들
## State
> 앱의 데이터와 현재 상태를 나타낸다. 앱의 UI를 그릴때 필요한 모둔 정보를 보유하는 구조체이다

```Swift
struct CounterState {
  var count = 0 
}
```

## Action
> State를 변경할 수 있는 사용자의 상호작용이나 이벤트 등을 Action이라 한다

```Swift
enum CounterAction {
  case addCount
  case substractCount
}
```

## Environment
> API클라이언트나 데이터 서비스 같이 App이 필요로 하는 의존성(Dependency)를 가지고있다
```Swift
struct CounterEnvironment {
  var client: APIClient
}
```

## Reducer 
> 어떤 Action이 주어졌을 때 지금 State를 다음 State로 변화시키는 방법을 가지고 있는 함수
- Reducer는 실행할 수 있는 Effect를 반환해야 하며 보통은 .none을 반환한다

```Swift
let counterReducer = Reducer<CounterState, CounterAction, CounterEnvironment> { state, action, environment in 
  switch action {
    case .addCount:
    state.count += 1
    return .none

    case .substractCount:
    state.count -= 1
    return .none
  }
}
```

## Store
> Store는 앱의 State를 유지하고 Action에 대한 응답으로 Reducer를 실행하는 역할을 담당한다

```Swift
let store = Store(
  initialState: CounterState(),
  reducer: counterReducer,
  environment: CounterEnvironment()
)
```

## TCA의 기본적인 데이터 흐름
> 유저의 Action을 Store에 보내 Reducer와 Effect를 실행시켜, Store에서 일어나는 State의 변화를 관측하여 UI를 업데이트 한다

## Usage
### TCA를 SwiftUI프로젝트에 적용시키는 방법을 알아보자

```Swift
struct CounterView: View {
  let store: Store<CounterState, CounterAction>

  var body: some View {
    WithViewStore(self.store) { viewStore in 
        VStack {
          Button("+") { viewStore.send(.addCounter) }
          Text("\(viewStore.count)")
          Button("-") { viewStore.send(.substractCounter) } 
        }
    }
  }
}
```
- 플러스, 마이너스 버튼을 누르게되면 각각의 Action이 Store에 전달되고 Reducer에서 해당 Action의 State변경과 Effect를 방출하면 viewStore를 통해 변경된 상태값을 Text에 띄워준다 

## Reference
https://gist.github.com/pilgwon/ea05e2207ab68bdd1f49dff97b293b17  
https://elisha0103.tistory.com/26


