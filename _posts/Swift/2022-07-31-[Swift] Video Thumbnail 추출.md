---
title: "[Swift] Video Thumbnail 추출"
last_modified_at: 2022-07-31T19:06:00-05:00
toc: true
categories:
  - Swift
tags:
  - Swift
  - AVAsset
  - PHAsset
---

# Video Thumbnail 추출
게시물을 업로드하는 기능에서 라이브러리에서 동영상을 선택한후 선택한 동영상의 썸네일을 띄워줘야 할 떄 여러가지 방법으로 표현할수 있겠지만, 오늘 해볼 방법은 `AVAssetImageGenerator`를 사용하여 영상의 원하는 시간에 대한 캡쳐 이미지를 뽑아오는 방법이다

### 1. 선택한 영상의 path값 추출
`FileManager`를 이용 하여 `PHAsset`에서 path값을 추출해낸다
```swift

func videoLocalPath(_ phAsset: PHAsset) -> URL? {
   guard let resource = (PHAssetResource.assetResources(for: phAsset).filter{ $0.type == .video }).first else {
            return nil
        }
    
    let filename = resource.originalFilename
    let localPath = FileManager.default.temporaryDirectory.appendingPathComponent(filename)
    return localPath
}
```
guard 문으로 선택한 영상의 `assetResource`를 안전하게 추출한뒤 `FileManager`를 이용해 비디오의 localPath값을 리턴시켜준다

### 2. AVAssetExpoertSession을 이용한 비디오 내보내기
비디오 내보내기를 하지 않으면 추출한 localPath값으로 이미지 추출이 되지않는다
따라서 AVAssetExportSession을 활용해 비디오 내보내기를 진행해줘야 한다
이번 프로젝트에선 `RxSwift`를 사용했기 때문에 `Ovservable`로 리턴시켜줬다

```swift
func getVideoURL(_ phAsset: PHAsset) -> Observable<URL?> {

    let localPath = videoLocalPath(phAssset)

    return Observable<URL?>.create { observer in
            PHImageManager.default().requestAVAsset(forVideo: phAsset, options: nil) { avasset, avaudio, info in
            guard let avasset = avasset,
                    let exportSession = AVAssetExportSession(asset: avasset, presetName: AVAssetExportPresetMediumQuality) else {
                observer.onError(Error)
                return
            }
                
            exportSession.outputURL = localPath
            exportSession.outputFileType = .mov
            exportSession.exportAsynchronously {
                observer.onNext(localPath)
                observer.onCompleted()
            }
        }
        return Disposables.create()
    }
}
```
(비디오를 압축시키고 싶을땐 presetName프로퍼티를 조절해 압축한다)
비디오 내보내기가 완료되면 observer.onNext()를 통해 URL을 방출시켜준다

### 3. AVAssetImageGenerator로 이미지 썸네일 추출하기
localPath로  `AVURLSeet`을 만들어 `AVAssetImageGenerator`에 전달해 이미지를 만든다

```swift

func generateThumbnail(_ path: URL) -> UIImage? {
    let asset = AVURLAsset(url: path, oprion: nil)
    let imageGenerator = AVAssetImageGenerator(asset: asset)
    let time = CMTimeMake(value: 0, timescale: 1)
    let cgImage = try? imageGenerator.copyCGImage(at: time, actualTime: nil)
    let thumbnail = UIImage(cgImage: cgImage)
    return thumbnail
}
```

`CMTimeMake`로 썸네일로 추출하고싶은 시간을 생성한뒤 copyCGImage를 통해 CGImage를 복사해준다

### 4. 정리
순서대로 선택한 비디오에서 url을 추출한뒤, 비디오 내보내기를 통해 이미지 썸네일을 생성할 준비를 하고
마지막으로 썸네일을 추출하고 싶은 시간을 정해 `ImageGenerator`에 전달해주면 UIImage를 얻을수 있다!