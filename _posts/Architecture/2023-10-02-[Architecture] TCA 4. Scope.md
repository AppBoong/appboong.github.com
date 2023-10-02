---
title:  "[TCA] 4. Scope"
last_modified_at: 2023-10-02T19:06:00-05:00
toc: true
categories:
  - Architecture
tags:
  - Swift
  - SwiftUI
  - TCA
---

# Scope  

> Scope를 활용하면 상위 Reducer에서 특정 Action에 대한 세분화된 Effect, State관리를 하위 Reducer에서 처리할 수 있게된다.
한마디로 어떤 Action에 대해 다양한 처리를 하고싶을땐 하위 Reducer로 교체를 시켜줄 수 있게되는 것이다.
예제 코드를 통해 실제 활용법에 대해 알아보자

```swift
struct Search: Reducer {
  struct State: Equatable {
    var searchText = ""
    var searchResults = [Item] = []
    var itemDetail: ItemDetail.State?
  }

  enum Action: Equatable {
    case requestSearch(String)
    case searchResponse(<TaskResult[Item]>)
    case showItemDetail(ItemSearch.Result)
    case dismissItemDetail
    case itemDetail(ItemDetail.action)
  }

  var body: some ReducerOf<Self> {
    Reduce { state, action in 
      switch action {
        case .requestSearch(let text):
        return .run { send in 
          await send(.searchResponse(
            // search 로직 구현
          ))
          
        }
        case .searchResponse(let result):
        switch result {
          case .success(let results):
          state.searchResults = results
          return .none

          case .failure(_):
          state.searchResults = []
          return .none
        }
        case .showItemDetail(let result):
        state.itemDetail = .init(result: result)
        return .none

        case .dismissItemDetail:
        state.itemDetail = nil

        case .itemDetail:
        return .none
      }
      .ifLet(\.itemDetail action: /Action.itemDetail) {
        ItemDetail() // State 의 itemDetail 이 nil 이 아니라면 Reducer 교체
      }
      }
  }
}

struct ItemDetail: Reducer {
  struct State: Equatable {
    let result: Item.Result

    init(result: Item.Result) {
      self.result = result
    }
  }

  enum Action: Equatable {
    case onAppear
    case onDisAppear
  }

  var body: some ReduceOf<Self> {
    Reduce { state, action in 
    switch action {
      case .onAppear:
      return .none
      case .onDisappear: 
      return .none
    }
    }
  }
}
```
- 자식 State 를 부모 State 에 포함 시킴으로서 부모Reducer가 자식 Reducer의 State 객체를 치환 혹은 초기화가 가능하다

- ifLet 스코프는 말그대로 State의 어떤 값이 optional 일때 해당 값이 들어오면 Reducer를 갈아끼워 주고싶을때 용이하다
  - 위 코드는 어떤 정보를 찾고 리스트에서 해당 객체를 눌렀을때 detail을 띄워주는 로직이다

- ifLet 이 아닌 원래의 Scope는 다음과 같이 사용할 수 있다
```swift

var body: some ReduceOf<Self> {
  Scope(state: \.someState, action: /Action.someAction) {
    AnotherReducer()
    // someAction이 들어왔을때 AnotherReducer로 처리하겠다
  }

  Reduce { state, action in 
   //Action switch문
  } 
}

```


