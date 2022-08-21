---
title:  "[Network] Moya RenewalToken"
last_modified_at: 2022-08-21T19:06:00-05:00
toc: true
categories:
  - Network
tags:
  - Moya
  - RxSwift
  - RenewalToken
---

# Moya RenewalToken

Moya를 활용해 네트워크로직을 구성했을때 401 에러에 대한 처리 로직을를 매번 API를 호출할때 마다
작성 해 줄 수는 없기 때문에 RxSwift를 활용해 오퍼레이터를 만들어주려고한다

## 1. AuthAPI 작성
```swift

enum AuthAPI {
    case renewalToken(accessToken: String, refreshToken: String)
}

extension AuthAPI: TargetType {
    var baseURL: URL {
        return APIEnvironment.baseUrl
    }
    
    var path: String {
        switch self {
        case .renewalToken:
            return "/auth/refresh"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .renewalToken:
            return .post
        }
    }
    
    var sampleData: Data {
        return Data()
    }
    
    var task: Task {
        switch self {
        case .renewalToken(let accessToken, let refreshToken):
            return .requestParameters(parameters: ["accessToken" : accessToken, "refreshToken" : refreshToken], encoding: JSONEncoding.default)
        }
    }
    
    var headers: [String : String]? {
        let accessToken = Userdefaults.standard.string(forKey: "accessToken") ?? ""
        return ["Authorization": "Bearer \(accessToken)"]
    }
}
```

## 2. TokenReseponse 작성
```swift
struct TokenResponse: Decodable {
    var accessToken: String
    var refreshToken: String?
}
```

## 3. PrimitiveSequence extension 을 활용해 토큰 만료처리 오퍼레이터를 만들어줌
```swift

extension PrimitiveSequence where Trait == SingleTrait, Element == Moya.Response {
    private func filter401StatusCode() -> Single<Moya.Response> {
        return flatMap {
            if $0.statusCode == 401 {
                throw MoyaError.statusCode($0)
            } else {
                return .just($0)
            }
        }
    }
    
    func retryWithAuthIfNeeded() -> Single<Moya.Response> {
        
        let access = UserDefaults.standard.string(forKey: "accessToken") ?? ""
        let refresh = UserDefaults.standard.string(forKey: "refreshToken") ?? ""
        
        return filter401StatusCode().catchError { tokenExpiredError in
            APIService.shared.authAPI.rx.request(.renewalToken(accessToken: access, refreshToken: refresh))
                .filterSuccessfulStatusCodes()
                .catchError { refreshTokenError in
                    
                    if (refreshTokenError as! MoyaError).response?.statusCode == 401 {
                        //RefreshToken 도 만료되었으므로 로그아웃 처리 
                    }
                    
                    return Single.error(refreshTokenError)
                }
                .flatMap { response -> Single<Moya.Response> in
                    if let token = try? response.map(TokenResponse.self) {
                        UserDefaults.standard.set(token.accessToken, forKey: "accessToken")
                        // 새로 발급받은 AccessToken 로컬에 저장
                    }
                    return Single.error(tokenExpiredError)
                }
            
        }.retry(2)
    }
}
```

## 4. MoyaProvider를 활용해 API호출시 토큰만료 오퍼레이터를 붙여준다

```swift

let provider = MoyaProvider<SomeAPI>()

func someRequest() -> Single<Response> {
    return provider.rx.request(.getSomeData)
            .retryWithAuthIfNeeded() // 401에러가 뜨게되면 토큰만료 처리로직이 실행됨
            .filterSuccessfulStatusCodes()
            .map(SomeResponse.self)

}