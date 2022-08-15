---
title:  "[Network] Moya를 활용한 Error 처리"
last_modified_at: 2022-08-15T19:06:00-05:00
toc: true
categories:
  - Network
tags:
  - Moya
  - Alamofire
  - RxSwift
---

# Moya를 활용한 Error 처리   

MoyaProvider를 이용하여 네트워크 로직을 짜던중 status code 에 따른 에러처리를 나눠서 해줘야 하는 경우가 생겨 한번 처리해 보려고 한다

## 1. ErrorResponse 구조체 생성
```swift
struct ErrorResponse: Decodable {
    var message: String?
    var code: Int?
}
```
- message: 서버에서 보내주는 메세지를 직접 유저에게 띄워줘야하는 경우 사용
- code: status code 외에 따로 분기처리가 필요한 경우 사용


## 2. Moya.Response를 리턴하는 함수 생성
```swift
private let provider = MoyaProvider<UserAPI>()

func updateUsername(_ name: String) -> Single<Moya.Response> {
    let request = provider.rx
            .request(.putUsername(name: name))

    return request
}
```

- 미리 구현해놓은 Moya UserAPI enum type의 provider를 생성해준다


## 3. subscribe를 통해 에러처리 로직 추가
```swift
private func didTapDone() {
    let username = "jeremy"
    updateUsername(username)
        .subscribe(onSuccess: { response in
        // success action
        }, onError: { error in
            if let moyaError = error as? MoyaError,
               let statusCode = moyaError.response?.statusCode {
                    guard let data = moyaError.reseponse?.data,
                          let errorResponse = try? JSONDecoder().decode(ErrorReseponse.self, from data),
                          let message = errorResponse.message else {
                            // docoding error action
                            return
                          }
                    print(message)
                    // 서버에서 내려주는 메세지 활용    
            } else {
                // network error action
            }

        })
        .disposed(by: disposeBag)
}
```

- statusCode에 따라 분기처리가 필요할때도 활용 가능하다
