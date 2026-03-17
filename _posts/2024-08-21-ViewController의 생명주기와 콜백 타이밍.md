---
title: "[UIKit] ViewController의 생명주기와 콜백 타이밍"
date: 2024-08-21 01:42 +0900
description: "UIKit ViewController의 생명주기 메서드와 콜백 타이밍을 자세히 알아봅니다."
categories:
  - Mobile
  - iOS
tags:
  - ViewController
---
뷰의 라이프사이클은 앱을 생성할 때 가장 먼저 보이는 것중 하나이다.

언제 나오냐고요?

여러분이 바로 앱을 생성하자마자 나오는 코드중 `ViewDidLoad()`함수가 뷰 라이프사이클을 관리하는 함수 중 하나이다.

그만큼 뷰의 생명주기에 대해서 이해하는 것은 앱을 설계하는데 있어서 중요한 요소중 하나이고, 오늘은 이 **ViewController의 생명주기**에 대해서 스스로 다시 학습해볼겸 내용을 정리해볼까 한다.

## 🤔 ViewController?
`ViewController`의 생명주기를 알기 이전에 `ViewController`라는 친구에 대해 간단하게 알아볼 필요가 있다.

> #### [Apple Developer Document - View Controller](https://developer.apple.com/documentation/uikit/view_controllers)
 View Controller는 단일 Root View를 관리하며, 이는 그 자체로 여러 개의 하위 View를 포함할 수 있다. 해당 View 계층 구조와의 사용자 상효 작용은 필요에 따라 앱의 다른 개체와 조정하는 View Controller에 의해 처리된다.
 모든 앱에는 콘텐츠가 메인 창을 채우는 적어도 하나의 View Controller가 있다. 앱에 한 번에 화면에 들어갈 수 있는 것보다 더 많은 컨텐츠가 있다면, 여러 View Controller를 사용하여 해당 컨텐츠의 다른 부분을 관리해라.

뭔가 내용이 좀 추상적인것 같아 다른 문서를 좀 확인하던 중 `View Controller`의 역할이라는 이름으로 정리된 아카이브 문서가 있어서 들고와봤다. 해당 문서에 따르면 `View Controller`의 역할은 크게 4가지로 나뉘는 것 같았다.

> #### [The Role of View Controllers](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/index.html#//apple_ref/doc/uid/TP40007457)
- View 관리
- 데이터 객체와 View 사이를 관리
- 사용자와의 상호작용
- 리소스 관리

어느정도 `ViewController`의 개념을 잡을 수 있는 좋은 문서들이었다. 결국 `ViewController`는 `View`를 관리하기 위한 존재이고, `View`를 위한 추가적인 역할을 하는 존재임을 파악할 수 있었다.

### 정리
- RootView와 하위 View를 가질 수 있고 이들은 관리
- 이러한 View와 사용자의 상호작용을 관리
- View와 데이터의 상호작용을 관리
- 이런 역할의 ViewController는 앱마다 최소 하나 이상이 존재

## 🔄 ViewController의 생명주기
이렇게 알아본 내용에 따르면 `ViewController`는 앱이 시작하고부터 무조건 하나 이상이 존재하는 상태일 것이다. 하지만 분명히 앱이 실행됐을 때 앱 안에있는 모든 `ViewController`들이 동작하는 것은 아닐것이다. 

만약 그렇다 한다면 엄청나게 메모리를 먹기도 할 것이고, 만약 CPU의 성능이 부족한 아이폰이라면 앱의 동작속도 또한 저하될것이다. 그렇기에 이러한 한정된 리소스 안에서 앱을 최적화하기 위해서 우리가 갖고 있는 이 앱속에 있는 무수히 많은 `View Controller`에 대해 관리할 줄 알아야된다. 

![](assets/img/post/2024/08_21_View_Controller_생명주기.jpeg)


일반적으로 UIViewController의 생명주기라 한다.

네이밍이 매우 직관적이어서 어떠한 의미인지 바로 느낌이 오긴 하지만 간략하게 정리해보자
#### [init](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621359-init)
- ViewController의 인스턴스를 초기화할 때 호출
- 객체가 생성될 때 필요한 초기 설정을 할 때 사용
- nibName과 bundle을 받는 생성자는 인터페이스 빌더 파일을 사용해서 ViewController를 초기화 할 때 사용

#### [loadView](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621454-loadview)
- ViewController의 View 계층을 메모리에 로드하는 함수
- 일반적으로는 시스템이 자동으로 loadView를 함
- 직접 호출할 경우 검은 화면을 띄움
- 대신 override 하는것은 가능함

#### [viewDidLoad](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621495-viewdidload)
- ViewController의 View가 메모리에 로드된 후 호출
- 보통 View의 초기 설정을 할 때 사용

#### [viewWillAppear](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621510-viewwillappear)
- View가 화면에 나타나기 직전에 호출
- View가 표시될 준비를 할 때 주로 사용

#### [viewIsAppearing](https://developer.apple.com/documentation/uikit/uiviewcontroller/4195485-viewisappearing)
- viewWillAppear과 viewDidAppear 사이에 호출
- viewWillAppear는 단순히 View가 추가될 예정을 알리는 메소드이지만 타이밍이 정확하지 않다는 점이 있음
- 하지만 viewIsAppearing은 superView가 View의 배치를 끝낸 뒤 호출되기에 View geomerty를 정확히 파악할 수 있음
- View가 보일 때 UI를 업데이트 하기 위한 최적의 장소

#### [viewDidAppear](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621423-viewdidappear)
- View가 화면에 완전히 나타난 후 호출
- 화면이 나타난 후 추가 작업을 할 때 사용

#### [viewWillDisappear](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621485-viewwilldisappear)
- View가 화면에서 사라지기 직전에 호출
- View가 사라질 때 필요한 정리 작업을 할 때 사용

#### [viewDidDisappear](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621477-viewdiddisappear)
- View가 화면에서 완전히 사라진 후 호출
- View가 사라진 후 추가적인 작업을 수행할 때 사용

#### [viewDidUnload](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621383-viewdidunload)
- iOS6 이전 버전에서 메모리 경고를 처리하기 위해 사용
- 메모리 부족으로 인해 View를 메모리에서 해제할 때 호출
- 현재는 더이상 사용되지 않고, 대신 [didReceiveMemoryWarning()](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621409-didreceivememorywarning) 메소드를 이용해 메모리 경고를 처리하는 방법이 변경


## 📞 해당 메소드들은 언제 콜백이 되는것일까?
생명주기에 대해 학습하던중 도대체 이 메소드들을 `UIViewController`가 콜백하는지 알고싶어졌다. 분명 `UIViewController`도 해당 `ViewCycleMethod`를 특정 트리거에 의해 호출되는 시점이 있을것이라 생각하는데, 열심히 찾아보면 결국 **"UIViewController가 자동으로 인지해서 호출한다"**로 귀결이 되서 열심히 찾아봤다.


### loadView()
- UIViewController의 view 프로퍼티의 lazy loading 과정에서 호출
- 기본적으로 UIViewController는 스토리보드나 NIB 파일을 사용하여 view를 로드
- 그렇지 않은 경우 해당 메소드를 override하여 programming하게 view를 생성

### viewDidLoad()
- loadViewIfNeeded() 또는 view property에 접근할 때 loadView()가 호출되고, 그 후 viewDidLoad()가 호출된다.

### viewWillAppear(\_:)
- UIViewController의 [beginAppearanceTransition(\_:animated:)](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621387-beginappearancetransition) 메소드에서 호출

### viewDidAppear(\_:)
- UIViewController의 [endAppearanceTransition()](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621503-endappearancetransition) 메소드에서 호출

### viewWillDisappear(\_:)
- UIViewController의 [beginAppearanceTransition(\_:animated:)](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621387-beginappearancetransition)메소드에서 호출

### viewDidDisappear(\_:)
- UIViewController의 [endAppearanceTransition()](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621503-endappearancetransition) 메소드에서 호출


## 👍 마무리
오늘은 이렇게 `ViewController`의 생명주기에 대해 알아봤는데 이렇게 알아보는 과정에서 생명주기에 대한 내용을 다시한번 익힐 수 있었고, 추가로 `ViewController`의 계층 구조라던지, 역할에 대해 더 자세하게 알 수 있었다. 

좀 더 알아봤던 `view cycle`의 `method`들이 어느 함수에서 콜백되는지 알아볼 수 있었고, 생각보다 애니메이션에 의존한다는 사실도 알 수 있었다.

## 🔗 레퍼런스
> - [Apple Developer Document - View Controller](https://developer.apple.com/documentation/uikit/view_controllers)
- [View Controller Programming Guide for iOS](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/index.html#//apple_ref/doc/uid/TP40007457)
- [Apple Developer Document - UIViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller)
- [iOS) View Controller의 생명주기(Life-Cycle)](https://zeddios.tistory.com/43)
- [viewIsAppearing- ZeddiOS](https://zeddios.tistory.com/1390)