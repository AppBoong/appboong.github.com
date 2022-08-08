---
title:  "2022-08-08-[Network] Moya, RxSwift를 활용한 네트워킹"
last_modified_at: 2022-08-08T19:06:00-05:00
toc: true
categories:
  - Network
tags:
  - Moya
  - Alamofire
  - RxSwift
---

# Moya, RxSwift를 활용한 네트워킹  

이번 프로젝트에서 리스트를 받아 부려주는 화면의 로직을  
Moya를 활용하여 Reactive하게 짜보려고 한다

## 1. ResponseModel 작성
```swift
struct Student: Decodable {
    let id, certCount: Int
    let name, image: String
}

struct StudentsResponse: Decodable {
    let students: [Student]
}
```

## 2. API Service enum 작성
```swift
struct APIEnvironment {
    static let baseURL: URL(string: "https://baseurl.com")
}

enum StudentService {
    case getStudents
}

extension StudentService: TargetType {
    var baseURL: URL {
        return APIEnvironment.baseUrl
    }
    
    var path: String {
        return "/students"
    }
    
    var method: Moya.Method {
        return .get
    }
    
    var sampleData: Data {
        return Data()
    }
    
    var task: Task {
        return .requestPlain
    }
    
    var headers: [String : String]? {
        let accessToken = UserDefaults.standard.string(forKey: "userAccessToken")
        return ["Authorization": "Bearer \(accessToken)"]
    }
}
```

## 3. ViewModel 작성

```swift
class StudentViewModel {
    static let shared = StudentViewModel()

    private let disposeBag = DisposeBag()

    private let provider = MoyaProvider(StudentService)()

    let students = BehaviorRelay<[Student]>(value: [])

    init() {
        setupData()
    }
}
```

## 4. MoyaResponse를 Subscribe

```swift
extension StudentViewModel {
    private func requestStudents() -> Ovservable<StudentsResponse> {
        return provider.rx.request(.getStudents)
            .filterSuccessfulStatusCodes()
            .map(StudentsResponse.self)
            .asObservable()
    }

    private func setupData() {
        requestStudents().subscribe(onNext: { [weak self] response in
            guard let self = self else { return }
            self.students.accept(response.students)
        })
        .disposed(by: disposeBag)
    }
}
```

## 5. TableView에 ViewModel 데이터를 바인딩해줌
```swift
private let viewModel = StudentViewModel.shared

override func viewDidLoad() {
    super.viewDidLoad()
        
    bind(viewModel.student)
}

private func bind(_ dataSource: BehaviorRelay<[Student]>) {
    dataSource.bind(to: tableView.rx.items(
        cellIdentifier: StudentCell.identifier,
        cellType: StudentCell.self
    )) { index, student, cell in
        cell.setupData(student)
    }
    .disposed(by: disposeBag)
}
```