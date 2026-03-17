---
title: "[UIKit] UIViewController의 View를 갈아끼워도 될까? - loadView() 이해하기"
date: 2025-04-27 17:03 +0900
categories:
  - Mobile
  - iOS
tags:
  - UIKit
  - ViewController
  - PopPool
---
최근 프로젝트에서 ViewController에 새로운 컨벤션을 적용하는 과정에서 

```swift
self.view.addSubview(mainView)
```

이런 코드를 자주 보게 됐는데요.

그런데 문득 이런 방식이 기존 ViewController의 `self.view` 인스턴스를 낭비하는 건 아닐까 하는 생각이 들었습니다.

왜냐하면 기존 `self.view`는 덮어씌워진 채 메모리에 남아 있게 되고, 이게 쌓이면 앱 전체 메모리 사용량이 불필요하게 늘어날 수 있기 때문이죠.

그럼 여기서 궁금한 점은, `addSubview()`를 쓰는 대신 `self.view = mainView`처럼 직접 할당하는 건 어떨까 하는 건데요. 이 방법이 ViewController의 생명주기나 드로잉에 어떤 영향을 줄지 궁금해서 좀 더 자세히 알아보게 됐습니다.

## 🖼️ UIViewController의 View는 뭘까?

[공식 문서](https://developer.apple.com/documentation/uikit/uiviewcontroller/view)를 살펴보면, ViewController가 관리하는 루트 View라는 점을 알 수 있어요. 그리고 이 프로퍼티는 { get set }으로 되어 있어서 외부에서 인스턴스를 바꾸는 것도 가능하다고 합니다.

초기값은 nil인데, 만약 `self.view`가 nil일 때 접근하면 ViewController가 자동으로 자신의 `loadView()` 메서드를 호출해서 View를 만들어주는 것 같아요.

그럼 여기서 또 궁금해지는 게 있습니다. 

_**(과연 ViewController의 `loadView()` 메서드에서는 어떻게 기본 View를 정의할까?)**_

## 🤔 loadView()는 어떻게 View를 만들어낼까?

`loadView()` 메서드는 ViewController의 View 계층 구조를 설정하는 역할을 합니다.

앞서 말했듯이, 일반적으로 ViewController의 View에 접근하는 시점에 이 메서드가 자동으로 호출되는데요.

먼저, 만약 `loadView()`가 오버라이드되어 있다면, **그 안에서 정의한 방식대로** View가 만들어집니다.

그렇지 않다면, ViewController가 초기화될 때 [`init(nibName:bundle:)`](https://developer.apple.com/documentation/uikit/uiviewcontroller/init(nibname:bundle:)) 생성자를 통해 번들 내 스토리보드 파일을 찾아, **스토리보드에 정의된 대로** View를 그려줍니다.

마지막으로, 오버라이드도 없고 스토리보드 파일도 없는 경우에는 **시스템이 기본 UIView 인스턴스를 생성해 `self.view`에 할당**하게 되죠.

## 🔧 그렇다면 View를 직접 교체해도 괜찮을까?

그럼 이쯤에서 정리해보면, ViewController의 기본 View는 별도의 스토리보드나 오버라이드된 `loadView()` 메서드가 없다면 그냥 빈 UIView 인스턴스라고 볼 수 있어요.

그래서 만약 mainView의 Constraints를 잘 잡고, autoresizingMask도 적절히 설정한다면, 오버라이드된 `loadView()`에서 `self.view = mainView`처럼 직접 할당하는 것도 크게 문제되지 않을 것 같습니다.

## 🏁 마무리

오늘은 UIViewController의 기본 View가 어떻게 생성되는지 과정을 좀 더 자세히 살펴보았는데요.

이를 통해 ViewController의 `self.view`를 직접 교체해도, 올바른 Constraints 설정과 autoresizingMask 조정만 잘 이루어진다면 특별히 문제가 발생하지 않는다는 점을 확인할 수 있었습니다.

## 🔗 참고 자료
> 이 내용을 살펴보면서 도움이 되었던 링크들입니다.
>- [view \| Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiviewcontroller/view)
>- [loadView() \| Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiviewcontroller/loadview())