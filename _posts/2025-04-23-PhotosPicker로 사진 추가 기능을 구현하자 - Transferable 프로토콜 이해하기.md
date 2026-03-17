---
title: "[SwiftUI] PhotosPicker로 사진 추가 기능을 구현하자 - Transferable 프로토콜 이해하기"
date: 2025-04-23 18:24 +0900
description: "SwiftUI PhotosPicker와 Transferable 프로토콜을 이용한 사진 추가 기능 구현 방법을 알아봅니다."
categories:
  - Mobile
  - iOS
tags:
  - Transferable
---

후배와 함께 진행하는 프로젝트 중 사용자 프로필 설정 화면을 만들면서 사진을 추가하거나 제거하는 기능을 구현해야 하는 일이 있었습니다.

SwiftUI를 최대한 활용하는 것이 목적이었기에, 대다수의 레퍼런스에서 사용되는 `ImagePicker`를 래핑하는 방식을 지양하고자 했습니다.

그래서 이를 대체하기 위해 iOS 16부터 지원되는 **`PhotosPicker`를 활용하여 프로필 사진 추가 및 제거 과정을 MVVM 패턴에 따라 정리**해보았습니다.

> **\[GitHub PR] 프로필 사진 추가 및 제거 UI와 기능 구현 PR**
> 
> [[FEAT] 프로필 설정 UI 그리기 및 기능 구현](https://github.com/Percevy/iOS/pull/5)

## 🖼️ PhotosPicker 띄우기

`PhotosPicker`를 띄우는 방법은 두 가지가 있습니다.

### 1. PhotosPicker Initializer

첫 번째는 `PhotosPicker`의 생성자를 이용해 Label에 `PhotosPicker`가 Present되도록 동작을 연결하는 방식입니다.

#### **코드**
```swift
PhotosPicker(
    "사진 선택하기",
    selection: $viewModel.selectedPhoto,
    matching: .images
)
```

#### **동작**

![](assets/img/post/2025/04_23_PhotosPicker_생성자.gif)

### 2. PhotosPicker ViewModifier

두 번째는 `PhotosPicker`를 ViewModifier에 두어 `@State` 값이 변경될 때 Present되도록 하는 방식입니다.

#### **코드**

```swift
.photosPicker(
    isPresented: $isPhotosPickerPresented,
    selection: $viewModel.selectedPhoto,
    matching: .images
)
```

#### **동작**

![](assets/img/post/2025/04_23_PhotosPicker_Modifier.gif)

### ViewModifier를 선택한 이유

개인적으로 `confirmationDialog`(ActionSheet)를 이용해 프로필 변경 동작을 여러 개 선택하기에, 바로 Present되는 방식이 아닌 동작 선택의 콜백으로 `PhotosPicker`를 띄우고 싶어 두 번째 방식을 선택했습니다.

### Selection 파라미터

두 방식 모두 공통으로 사용하는 파라미터가 있습니다. 바로 `selection`입니다.

`selection`은 `PhotosPickerItem` 타입을 요구하는 파라미터로, `PhotosPicker`에서 선택한 사진이 담길 변수를 의미합니다.

따라서 사진을 `PhotosPicker`로 받아내기 위해서는 해당 타입의 변수를 선언하고 바인딩해줘야 합니다.

## 📥 전달받은 사진 처리하기

선택된 사진을 `PhotosPickerItem`으로 전달받으면 우리가 일반적으로 사용하는 `Image` 타입이 아니기 때문에, 이를 `Image`에 사용할 수 있도록 변환해줘야 합니다.

### 1. loadTransferable(type:)를 통해 Data로 변환

`PhotosPickerItem`에서 비동기적으로 `Transferable` 타입의 데이터를 로드하는 메서드입니다.

```swift
import SwiftUI
import PhotosUI
import OSLog

class SetupProfileViewModel: ObservableObject {
    @Published private var profile: Profile
    @Published var selectedPhoto: PhotosPickerItem? {
        didSet { loadImageData(from: self.selectedPhoto) }
    }

	...

    private func loadImageData(from selectedPhoto: PhotosPickerItem?) {
        Task {
            do {
                let data = try await selectedPhoto?.loadTransferable(type: Data.self)
                await MainActor.run { self.profile.updateAvatarImageData(to: data) }
            } catch {
                os_log(.error, "\(#file) \(#function) \(error.localizedDescription)")
            }
        }
    }
}
```

`PhotosPicker`로부터 받아낸 `PhotosPickerItem`을 지정한 타입(여기서는 `Data`)으로 변환해줍니다.

이로써 사진 앱에서 사진을 선택한 후 `Data` 형태로 변환까지 성공했습니다.

### 2. 모델의 데이터를 View에 바인딩

이제 남은 일은 이렇게 처리된 `Data`를 View에 바인딩하는 것입니다.

```swift
// 상위 View에서 호출
AvatarView(imageData: $viewModel.avatarImageBinding)

// 프로필 View 구현
struct AvatarView: View {
    @Binding var imageData: Data?

    var body: some View {
        Circle()
            .fill(Color.neutralOffWhite)
            .overlay {
                if let image = Image(data: imageData) {
                    image
                        .resizable()
                        .scaledToFill()
                        .clipShape(Circle())
                } else { Image(.avatar) }
            }
    }
}
```

이렇게 하면 ViewModel에 선언된 `@Published` 모델이 변경될 때 `@StateObject`로 선언된 ViewModel이 갱신되면서 최종적으로 사진이 추가됩니다.

### **🥳 최종 완성**

![](assets/img/post/2025/04_23_프로필_사진_추가_최종.gif)

## ❓ 궁금증 해결하기 - loadTransferable(type:)

이번 구현의 핵심은 `PhotosPickerItem`의 `loadTransferable(type:)` 메서드를 적절히 활용하는 것이었습니다.

이 메서드가 어떻게 `PhotosPickerItem`의 값을 `Data` 타입으로 변환하는지 궁금해져 알아보았습니다.

우선 [메서드 문서](https://developer.apple.com/documentation/photosui/photospickeritem/loadtransferable(type:))와 메서드의 원형을 봤을 때 다음과 같이 특성을 확인할 수 있습니다.

> **\[loadTransferable(type: ) 메서드의 조건]**
> 
> - 파라미터로 넘긴 타입(T)은 `Transferable` 프로토콜을 채택하고 있어야 합니다. 
> - 아이템이 지원하는 content type 중 T와 호환되는 첫 번째 representation을 찾습니다. 
> - 해당 representation을 비동기적으로 로드하여 T 인스턴스로 반환합니다.
{: .prompt-info }

_**(Transferable...? representation...?)**_

이게 대체 뭔지 싶은것들 투성입니다 🤣

하나씩 알아가보죠 ㅎㅎ

### Transferable 프로토콜

`Transferable`은 Apple 플랫폼에서 데이터를 안전하고 효율적으로 공유하거나 전송할 수 있도록 설계된 프로토콜입니다.

주로 드래그 앤 드롭, 복사/붙여넣기 등 전송 API와 타입이 상호작용할 수 있도록 만들어졌습니다.

`Transferable`을 채택한 타입은 이러한 전송 API에 사용될 수 있습니다.

이 때 저희가 사용한 `PhotosPicker`는 OS의 사진 앱과 내 앱 사이를 연결하는 전송 API라 볼 수 있습니다.

그래서 여기에 사용될 타입을 정의하기 위해 `Transferable` 프로토콜 채택이 필요합니다.

이번에 사용한 `Data` 타입은 [공식 문서](https://developer.apple.com/documentation/foundation/data) 하단에 `Transferable`을 준수한다고 명시되어 있어, 해당 메서드의 파라미터로 사용할 수 있었습니다.

### transferRepresentation

`Transferable` 프로토콜을 채택할 때 구현해야 하는 정적 프로퍼티입니다.

`Transferable` 타입을 어떻게 포함시키거나 전송할지 정의하는 역할을 합니다.

> **\[애플에서 제공되는 대표적인 Representation 종류]**
> 
> - **Codable:** 모델이 Codable을 채택해 JSON 형태로 직렬화가 가능할 때
> - **Data:** 모델이 Data로 변환 가능할 때
> - **File:** 파일 URL을 통해 대용량 전송이 필요할 때
> - **Proxy:** Transferable을 채택한 일반 타입들(String, Data, ...)을 전송할 때
{: .prompt-info }

다양한 Representation을 통해 최종적으로 모델을 어떻게 전송할지 정의할 수 있습니다.

### loadTransferable(type:) 동작 정리하기

이제 이해가 좀 더 명확해졌습니다.

`PhotosPickerItem`은 시스템 앱으로부터 전송받은 이미지입니다.

외부에서 전달된 것이기에, 이를 앱 내에서 사용하려면 해당 이미지를 받을 타입이 필요합니다.

애플은 `PhotosPickerItem`의 메서드로 `Transferable` 타입으로 변환하는 기능을 제공합니다.

따라서 우리는 `Transferable` 프로토콜을 채택한 타입을 정의하고, 해당 타입의 전송 방식을 `transferRepresentation`에 명시하여 외부에서 전달된 이미지를 앱에서 사용할 수 있게 되는 것입니다.

_**(아 이해했다~!~!~!!!)**_

## 🏁 마무리

이번 포스팅을 통해 SwiftUI에서 `PhotosPicker`를 이용해 사진을 가져오는 방법을 정리했고,

나아가 어떻게 이를 전송받는지 `Transferable` 프로토콜을 이해하는 데 도움이 되었습니다.

앞으로 애플의 기본 시스템을 활용해 데이터를 전달하거나 전송할 때 `Transferable` 프로토콜이 자주 등장할 것으로 예상됩니다.


## 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [Bringing Photos picker to your SwiftUI app \| Apple Developer Documentation](https://developer.apple.com/documentation/photokit/bringing-photos-picker-to-your-swiftui-app)
>- [Transferable \| Apple Developer Documentation](https://developer.apple.com/documentation/coretransferable/transferable)
>- [TransferRepresentation \| Apple Developer Documentation](https://developer.apple.com/documentation/coretransferable/transferrepresentation)
>- [Choosing a transfer representation for a model type \| Apple Developer Documentation](https://developer.apple.com/documentation/coretransferable/choosing-a-transfer-representation-for-a-model-type)