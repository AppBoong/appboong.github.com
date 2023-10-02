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
    var itemDetail: ItemDetail.Result?
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
        state.itemDetail = result
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
```



## Reference
https://medium.com/naverfinancial/%EB%84%A4%EC%9D%B4%EB%B2%84%ED%8E%98%EC%9D%B4-%EC%9B%8C%EC%B9%98%EC%95%B1-tca-%EC%A0%81%EC%9A%A9%EA%B8%B0-655f1d1b8c23


