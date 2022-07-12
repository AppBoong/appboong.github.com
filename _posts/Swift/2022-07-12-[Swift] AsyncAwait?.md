---
title: "[Swift] Async/Await?"
last_modified_at: 2022-07-12T19:06:00-05:00
toc: true
categories:
  - Swift
tags:
  - Swift
  - Concurrency
---

# Async / Await

## What is Async/Await?

swift5.5에 등장한 `async/await`는 이미 Javascript, kotlin등의 다른 언어에서 제공되던 비동기 처리방식을 편하게 해주는 문법이다  
그동안 swift에서는 비동기 프로그래밍을 delegate pattern, DispatchQueue, Closure등으로 처리했지만 `async/await`를 사용하면 더욱 편하고 짧은 코드로 비동기 처리방식을 구현할수 있다

```swift
// DispatchQueue 사용한 비동기 처리
DispatchQueue.global.async {

}

// Closure를 사용한 비동기 처리
let task = URLSession.shared.dataTask(with: url) { data, response, error in

}.resume()
```
- 이외에도 RxSwift나 Combine같은 대표적인 비동기 처리방법이 있다
  
### async
`async`마크가 표기된 함수는 비동기로 동작하는 함수가 된다
```swift
func someWork() async throws -> String {
  ...
}
```
위 예제처럼 `async` 마크 위치는 함수 매개변수 선언 뒤이고 `throws`의 앞이다.  
여기서 `throws`는 반드시 필요한 것이 아니라 그저 위치 설명을 위해서 추가한 것이다.

또한 `async`는 프로퍼티에도 마크하는 것이 가능하다
```swift
struct SomeStruct {
  var value: String {
    get async throws {
      ...
    }
  }
}
```
동시에 프로퍼티에도 `throws`를 표기할 수 있게 된다

### await
`await` 는 비동기 함수의 실행을 동기하는 명령이다. 즉 현재 스레드에서 해당 비동기 함수의 실행이 끝날 때까지 대기한다는 말이다.

```swift
func doWhatWithSomeWork() async {
  ...
  try? await someWork()
  ...
}
```
`await` 는 블로킹(blocking)을 유발하므로 `async` 로 마크된 함수 등 비동기 컨텍스트 내에서만 쓸 수 있다. 당연하게도 `await` 중인 함수의 실행이 완료될 때까지 코드의 실행은 중간에 멈추게(blocking) 된다.

`await` 를 사용할 수 있는 곳은 여러 곳이 있는데 위치가 미묘하긴 하다.  
예를 들어 for 루프의 경우 공식 홈페이지 소개 내용에 아래와 같은 식의 예제를 소개하고 있다.
```swift
import Foundation

let handle = FileHandle.standardInput
for try await line in handle.bytes.lines {
    print(line)
}
```

## How to use?

- 메소드 시그니처에 `async`를 붙여 비동기 작업임을 알리는 키워드를 추가
- 메소드 내부 구현에서 비동기가 예상되는 곳에 `await` 키워드를 추가
- 해당 메소드를 호출하는 쪽에서 `Task`키워드를 붙여 async block을 호출함
  
> **example code**

```swift
@objc func didTapButton() {
  guard let baseURL = URL(string: "https://example/image.png") else { return }

  Task {
    guard let imageURL = try? await self.requestImageURL(requestURL: baseURL),
          let url = URL(string: imageURL),
          let data = try? Data(contentsOf: url)
          else { return }
  
    print(Thread.isMainThread) // true
    self.imageView.image = UIImage(data: data)
  }
}

func requestImageURL(requestURL: URL) async throws -> String {
  print(Thread.isMainThread) // false
  let (data, _) = try await URLSession.shard.data(from: requestURL)
  return try JSONDecoder().decode(MyModel.self, from: data).data.first?.profile ?? ""
}
```

--- 

## References
[[iOS - swift] Async, Await 사용 방법](https://ios-development.tistory.com/958)  
[Swift async/await](https://seorenn.github.io/note/swift-async-await.html)