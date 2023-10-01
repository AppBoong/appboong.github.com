---
title:  "[TCA] 3. Dependency"
last_modified_at: 2023-10-01T19:06:00-05:00
toc: true
categories:
  - Architecture
tags:
  - Swift
  - SwiftUI
  - TCA
---

# Dependency  
> API Client나 Data Service 같은 외부 의존성을 주입해야하는 부분에 사용됩니다.
 이는 Client 를 완전히 분리해서 구현 할 수 있는 장점이 있고 테스트 환경에도 쉽게 적용이 가능해 로직이 집중되어 있는 곳을 정리할 수 있습니다.

## Usage

### 1. DependencyValue에 필요한 타입을 등록한다
```swift
extension DependencyValues {
  var someClient: SomeClient {
    get { self[SomeClient.self] }
    set { self[SomeClient.self] = newValue }
  }
}
```

### 2. DependencyKey 프로토콜을 사용해 liveValue 를 구현한다
```swift
struct SomeClient {
  // API 정의
  var fetch: @sendable () async -> Result<[SomeItem], ErrorTypeValue>
}

extension SomeClient: DependencyKey {
  static let liveValue: Self = {
    let client = SomeClient()

    return client 
  }
}
```

### 3. previewValue, testValue 를 구현하면 테스트나 프리뷰등의 상황에서 쉽게 데이터 전환이 가능하다
```swift
extension SomeClient: DependencyKey {
  static var testValue: Self(
    someParam: SomeParam
  )
}
```

### 4. 사용할 Reducer내부에 선언하여 사용해준다
```swift
@Dependency(\.someClient) var someClient
```



## Reference
https://medium.com/naverfinancial/%EB%84%A4%EC%9D%B4%EB%B2%84%ED%8E%98%EC%9D%B4-%EC%9B%8C%EC%B9%98%EC%95%B1-tca-%EC%A0%81%EC%9A%A9%EA%B8%B0-655f1d1b8c23


