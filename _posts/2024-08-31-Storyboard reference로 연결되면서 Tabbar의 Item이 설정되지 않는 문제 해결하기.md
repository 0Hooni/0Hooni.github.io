---
title: "[UIKit] Storyboard reference로 연결되면서 Tabbar의 Item이 설정되지 않는 문제 해결하기"
date: 2024-08-31 00:41 +0900
categories:
  - 🍎 Dev
  - UIKit
tags:
  - UIKit
  - Storyboard
  - TroubleShooting
  - ViewController
---
팀 동료와 미션 수행한 내용을 공유하는 과정에서 `Stroryboard`도 파일 분리를 할 수 있다는 것을 알게 되었다.

그래서 기존에 `Main.storyboard`에서 관리되고 있던 **스토리보드 컴포넌트들을 별도의 파일로 분리**를 했었다.

그런데 이 과정에서 `Tabbar`의 **아이템들을 설정하던 코드가 정상 작동을 하지 않았다.**

이전의 스토리보드는 `Main`에 있고 `하위 ViewController`들이 `Main`의 탭바와 연결될 때 자동적으로 `하위 ViewController`들의 `Tabbar UI`가 추가가 됐었다. 

하지만 `Storyboard reference`로 파일을 나눈 뒤 분리하게 되면 분리된 곳으로 간 `하위 ViewController`들에는 `TabbarController`가 연결됐는지 모르기에 자동적으로 `하위 ViewController`에 `Tabbar UI`가 추가되지 않는다.

이유가 이것 때문인지는 모르겠지만 아무튼 이러한 변경으로 인하여 `Tabbar`를 초기화하는 코드가 정상 작동하지 않아서 이 문제를 해결해보며 그 과정을 포스팅으로 기록해보려 한다.

## ⏮️ 이전까지의 방식
이전까지는 위에 언급된것 처럼 `Main.storyboard`에서 `하위 ViewController`들을 스토리보드를 통해 연결해주고 있었다.

그런 다음 `SceneDelegate`에서 아래와 같은 방식으로 `Tabbar`의 아이템들을 초기화 해줬었다.
```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
	guard let _ = (scene as? UIWindowScene) else { return }

	guard let tabBarController = self.window?.rootViewController as? UITabBarController else { return }
	guard let tabBarItems = tabBarController.tabBar.items else { return }
	
	// 첫번째 View의 tabbar 설정
	tabBarItems[0].title = "게임"
	tabBarItems[0].image = UIImage(systemName: "gamecontroller")
	tabBarItems[0].selectedImage = UIImage(systemName: "gamecontroller.fill")

	// 두번째 View의 tabbar 설정
	tabBarItems[1].title = "설정"
	tabBarItems[1].image = UIImage(systemName: "gear.circle")
	tabBarItems[1].selectedImage = UIImage(systemName: "gear.circle.fill")
}
```

### 동작 이해하기
일단 이 코드가 어떻게 동작했는지 부터 알아볼 필요가 있다.

해당 코드는 `SceneDelegate`에서 관리되고 있던 코드이다. `SceneDelegate`는 UI의 상태 변화를 메소드들을 통해 `application`에게 알리는 역할을 한다. 

그중 해당 코드가 선언된 메소드(`scene(_:willConnectTo:options:)`)의 역할은 아래와같다.
- 앱이 UI 인스턴스를 만들거나 복원할 때 호출되는 함수
- 화면에 앱이 등장했을때 실행됨
- 해당 메소드가 실핼될때는 아직 `ViewController` 객체들의 `viewDidLoad()`가 호출되지 않는 시점

그러면 일단 내가 원했던 해당 코드의 동작은 앱이 실행될 때 Tabbar의 item들의 인터페이스를 설정하는 것이었고, 호출 타이밍에는 이상이 없다.

아래 코드들 또한 이전의 `Storyboard` 환경에서는 전혀 문제가 없었기에 이부분의 문제는 아닐것이다.
## 🔄 변화된 것들



|                          변경 전                           |                          변경 후                           |
| :-----------------------------------------------------: | :-----------------------------------------------------: |
| ![](assets/img/post/2024/08_31_이전_스토리보드.jpg){: w="360"} | ![](assets/img/post/2024/08_31_이후_스토리보드.jpg){: w="300"} |



위와 같이 하나의 파일에서 처리하던 `Storyboard`의 컴포넌트들을 `Storyboard reference`로 나눠서 연결을 해주었다.

흠.... 뭐가 문제일까...

바뀐것은 레퍼런스를 나눠준것 단 하나인데....

이런저런 테스트를 해봤을 때 탭바에 설정은 제대로 들어간다. 

그렇다면 문제가 생기는것은 UI가 업데이트 되지 않는다는것

이 메소드가 끝나기 전에 UI가 업데이트가 이미 끝나버린것 아닐까? 하며 SceneDelegate의 함수가 끝나는 시점에 `print`도 찍어보고, `tabbar`의 `viewDidLoad()`, `첫번째 ViewController`의 `viewDidLoad()`도 모두 찍어봤지만 당연하게도 해당 탭바 설정 코드가 이들보다 더 먼저 시작해서 잘 끝난다.

## 💡 문제점 발견
**드디어 찾았다 ㅠㅠㅠㅠㅠㅠㅠㅠㅠㅠ**

분명 탭바에서

![](assets/img/post/2024/08_31_스토리보드_분리_이전_탭바_설정.png){: w="400"}_스토리보드 분리 이전 상황_

분명 분리 이전에는 `TabbarController`에서 `하위 ViewController`의 탭바를 체크했을 때 속성이 잘 들어갔던것을 볼 수 있다. 

반면에




![](assets/img/post/2024/08_31_스토리보드_분리_이후_탭바_설정.png){: w="400"}_스토리보드 분리 이후 상황_


분리 이후에는 `TabbarViewController`에서 설정은 들어갔지만 `하위 ViewController`들에서는 속성이 들어가있지 않는것을 볼 수 있었다.

## 🤔 이유 알기
시작은 스토리보드 레퍼런스를 분리하면서 일어낫던 것이다.

그렇다면 나누는것은 어째서 이런 문제를 야기하는 것일까?

우선 [UITabBarController \| Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uitabbarcontroller)문서에 의하면 `tabbarItem`을 구성하고 싶다면 꼭 각각의 ViewController에서 UITabbarItem 인스턴스를 만들어줘야된다. 

레퍼런스로 TabbarController과 이어진 하위 ViewController가 있다면 main.storyboard때와는 다르게 자동으로 UITabbarItem이 생성되지 않기에 아이템 설정이 먹지 않았을 수 있다. 

위 사진에서 하위 탭바의 아이템의 타이틀이 nil로 뜨던 이유는 이처럼 하위 ViewController에 UITabbarItem 인스턴스가 만들어져있지 않았기 때문이다.

하지만 이를 할당해주더라도 TabbarItem의 `title`은 여전히 세팅이 되지 않은체 기본상태로 유지되고 있다.

그러면 이건 무슨 이유때문일까?

이 비교를 보기 위해 여러 출력을 띄운 콘솔 로그를 비교해보자


|                     이 전                     |                     이 후                     |
| :-----------------------------------------: | :-----------------------------------------: |
| ![](assets/img/post/2024/08_31_이전_콘솔로그.png) | ![](assets/img/post/2024/08_31_이후_콘솔로그.png) |


사실 처음에는 하위 `ViewController`에 자동적으로 생성되지 않는 `TabbarItem`의 문제라고 생각했고, 이 문제로 탭바에서 하위 tabbar에 대한 참조가 없어서 제대로 값이 들어가지 않는 것이라 생각했다.

하지만 이것보다는 `Storyboard reference`로 구성하게 되면서 **View가 보여지는 시점**이 바뀌게 되었고, 이 문제로 인해서 제대로 View에 보여지지 않게 되는 것이었다.

객체가 `init`됐다면 인스턴스값에 할당을 해줘도 될 것이라 생각한다. 실제로 우리가 어떠한 인스턴스를 생성한 이후부터는 별일 없다면 할당할때 문제가 없었으니까.

**하지만 View의 경우는 달랐다.**

View가 전부 그려지기 전까진 그 어떤 값도 어떻게 될지 모르게 되는 것이다. 실제로 `AutoLayout`의 `Contraint`같은 경우도 [viewWillAppear()](https://developer.apple.com/documentation/appkit/nsviewcontroller/1434415-viewwillappear/) 시점까지도 잘 모른다.

그래서 [viewIsAppearing(_:)](https://developer.apple.com/documentation/uikit/uiviewcontroller/4195485-viewisappearing/) 라는 메소드도 추가로 생겼지만, 이 시점까지도 완벽하진 않다고 한다.

## 🧑🏻‍🔧 해결하기

위에서 알아본 사실을 기반하자면 우리가 지금같이 View의 요소를 변경하고 싶다면 적어도 해당 View가 전부 그려진 시점에 실행되는 [viewDidAppear(\_:)](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621423-viewdidappear/), [sceneDidBecomeActive(_:)](https://developer.apple.com/documentation/uikit/uiscenedelegate/3197915-scenedidbecomeactive/) 와 같은 메소드들에서 View의 데이터 변경, 셋팅을 해주는것이 좋다 한다. 

확실한것은 AppCycle, ViewCycle에 대해서는 무조건 이른것도, 무조건 늦는것도 좋지 않다는 것이었다. 각각의 설정에 어울리는 ViewCycle이 존재하는것이란거고, 그냥 무작정 `ViewDidLoad()`에 때려넣지는 말자.



# 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [[iOS] AppDelegate & SceneDelegate](https://sueaty.tistory.com/135)
>- [scene(_:willConnectTo:options:) \| Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiscenedelegate/3197914-scene)
>- [[iOS - SceneDelegate] iOS13이상 버전의 SceneDelegate](https://ios-development.tistory.com/53)
>- [ios - UItabbar item not showing storyboard reference - Stack Overflow](https://stackoverflow.com/questions/33754320/uitabbar-item-not-showing-storyboard-reference)
>- [UITabBarController \| Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uitabbarcontroller)