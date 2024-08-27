---
title: "[UIKit] 스토리보드 Component들 알아보기"
date: 2022-10-21 23:20 +0900
categories:
  - 🍎 Dev
  - UIKit
tags:
  - UIKit
  - Storyboard
---
## **Storyboard**

> Storyboard란 앱의 흐름을 나타내며, 시각적으로 화면을 구성하는 곳입니다. 좌측 프로젝트 리스트에서 Main.storyboard로 되어있는 파일이 Storyboard이며, 앱의 전반적인 형태와 앱의 화면 전환, 다양한 Object들을 관리해줍니다. Xcode 우측 상단의 **+버튼**을 누르거나 **Cmd + Shift + L**을 동시에 누르면 다양한 Object(Component)가 보이는 화면이 나타납니다.

## **Component(Object)**

위에 설명된 방법으로 Component List를 볼 수 있으며, 각 Object의 이름과 해당 Object가 수행하는 역할에 대해 간략하게 적혀있다.아래에 List에 존재하는 각 Component에 대하여 간략하게 정리해보았다.

### 1\. Label & Button

![](assets/img/post/2022/10_21_label_and_button.png)
_Cmd + Shift + L 을 통해 보여지는 Component List_

-   Label : 임의의 텍스트를 포함하고 있는 오브젝트
-   Button : 눌렀을 때 액션을 취하는 제어 오브젝트
-   Gray, Tinter, Filled Button : 눌렀을 때 액션을 취하는 제어 오브젝트, 그 여러 스타일 중 하나
-   Pull Down Button : Pull Down 메뉴를 보여주는 오브젝트
-   Pop Up Button : Pop Up 메뉴를 보여주는 오브젝트
-   Segmented Control : 수평(가로)의 버튼 목록들을 나타내는 제어 오브젝트
-   Text Field : 텍스트를 편집할 수 있는 텍스트 오브젝트
-   Slider : 일정 범위내에서 어떠한 값을 고를 수 있는 슬라이드 오브젝트
-   Switch : On과 Off의 상태를 나타내는 스위치 오브젝트
-   Activity Indicator View : 작업이 진행될 때 회전하며 나타나는 오브젝트
-   Progress View : 진행율을 나타내는 막대 오브젝트
-   Page Control : 페이지 목록을 점으로 표시해주는 오브젝트
-   Stepper : 특정 값의 증가 또는 감소를 하는 오브젝트
-   Color Well : 눌렀을 때 색상판이 나와 색상을 선택하는 오브젝트
-   Paste Control : 붙여넣은 콘테츠를 수신하는 오브젝트(??? 자료가 잘 안 보임)

### 2\. View

> View는 UIView 클래스의 인스턴스로, 윈도우 위에서 컨텐츠를 보여줍니다. 즉, 화면에 나타나는 거의 모든 요소를 뷰의 컨텐츠라고 봐도 무방합니다. 뷰는 컨텐츠를 나타내고 터치, 하위 뷰의 배치 등의 역할을 수행합니다.
![](assets/img/post/2022/10_21_view.png)

-   View : 비어있는 기본 View 오브젝트
-   Container View : 하위 View 컨트롤러를 호스팅 하는 오브젝트
-   Horizontal Stack View : View를 수평으로 정렬해주는 인터페이스 View 오브젝트
-   Vertical Stack View : View를 수직으로 정렬해주는 인터페이스 View 오브젝트
-   Table View : Data를 테이블 형태의 리스트로 표현해주는 View 오브젝트
-   Table View Cell : Table View의 단일 행을 보여주는 오브젝트
-   Image View : 이미지를 표시하는 View 오브젝트
-   Collection View : 테이블처럼 일정한 가로 세로 개수를 가진 목록 View 오브젝트
-   Collection View Cell : Collection View에서 한 개의 Cell을 나타내는 오브젝트
-   Collection Reusable View : 캐싱 동작을 구현하는 오브젝트(재사용이 가능한 View의 속성과 동작을 정의)
-   Text View : 여러 줄의 텍스트 라인을 편집하는 오브젝트
-   Scroll View : 스크롤 가능한 영역을 정하는 오브젝트
-   Date Picker : 날짜와 시간을 선택하게 해주는 오브젝트
-   Picker View : 행과 구성요소로 구성된 회전 휠을 제공하는 오브젝트
-   Visual Effect View with Blur : 흐린 배경을 제공하는 View 오브젝트
-   Visual Effect View with Blur and Vibrancy : 흐린 배경과 생동감 있는 효과를 제공하는 오브젝트

### 3\. Kit View

![](assets/img/post/2022/10_21_kit_view.png)

-   Map Kit View : Map 컨텐츠를 보여주는 오브젝트
-   MetalKit View : Metal로 렌더링된 그래픽을 제공해주는 오브젝트
-   GLKit View : OpenGL ES로 렌더링된 그래픽을 제공해주는 오브젝트
-   SceneKit View : SceneKit로 렌더링된 그래픽을 제공해주는 오브젝트
-   SpriteKit View : SpriteKit로 렌더링된 그래픽을 제공해주는 오브젝트
-   ARKit SceneKit View : SceneKit로 렌더링되고 ARkit으로 증강된 그래픽을 제공해주는 오브젝트
-   ARKit SpriteKit View : SpriteKit로 렌더링되고 ARkit으로 증강된 그래픽을 제공해주는 오브젝트
-   Web View (deprecated) : 상호작용하는 웹 컨텐츠를 표시하는 오브젝트. 구버전을 지원하는 View
-   WebKit View : 상호작용하는 웹 컨텐츠를 표시하는 오브젝트
-   RealityKit AR View : RealityKit로 렌더링된 그래픽을 제공해주는 오브젝트
-   Room Capture View : Room을 캡처하고 정의하는 오브젝트
-   Core Location Button : 장치의 현재 위치에 대해 일회성으로 권한을 부여하는 오브젝트

### 4\. Segue

> 세그웨이는 스토리보드에서 뷰 컨트롤러 사이의 연결관계 및 화면 전환을 관리하는 역할을 한다. 세그웨이는 화면과 화면을 연결하기 위해 아무런 소스 코드도 필요하지 않다는 특징이 있다. 뷰 컨트롤러와 뷰 컨트롤러 혹은 화면 전환의 매개체가 되는 버튼과 뷰 컨트롤러 사이를 직접 연결하는 식으로 화면 전환 관계를 구성한다. 스토리보드 상에서 세그웨이는 뷰 컨트롤러 사이에 연결된 화살표로 표시된다.

![](assets/img/post/2022/10_21_segue.png)

-   Navigation Bar : Title과 옵션 버튼을 제공하는 오브젝트
-   Navigation Item : Naviagtion Bar 컨텐츠 오브젝트
-   Toolbar : 버튼들이 있는 가로 Bar를 나타내는 오브젝트
-   Bar Button Item : Toolbar 또는 Navigation Bar 안에 있는 버튼 오브젝트
-   Bar Button Item Group : 하나 이상의 bar button이 묶여있는 오브젝트
-   Fixed Space Bar Button Item : Toolbar 내의 고정된 분리 오브젝트
-   Flexible Space Bar Button Item : Toolbar 내의 유동적인 분리 오브젝트
-   Tab Bar : 선택 가능한 아이템들이 담긴 행을 제공하는 오브젝트
-   Tab Bar Item : Tab Bar 안에 있는 Item 오브젝트
-   Search Bar : 검색 영역이 있는 bar를 제공하는 오브젝트
-   Menu Command : Action 혹은 설정을 나타내는 오브젝트
-   Main Menu : 기본 메인 메뉴 오브젝트
-   Sub Menu : 하위 메뉴를 보여주는 오브젝트
-   Inline Section Menu : Menu 또는 Command를 일렬로 그룹화시키는 오브젝트

### 5\. View Controller

> 뷰 컨트롤러는 앱의 근간을 이루는 객체로 모든 앱은 최소한 하나 이상의 뷰 컨트롤러를 가지고 있다. 주된 역할은 화면 구성 요소들, 즉 뷰를 관리하는 것이지만, 화면과 데이터 사이의 상호작용도 관리까지 한다.

![](assets/img/post/2022/10_21_view_controller.png)
-   View Controller : 한 개의 View를 관리하는 오브젝트
-   Storyboard Reference : 다른 스토리보드를 참조하는 오브젝트
-   Navigation Controller : Navigation bar와 View Controller 스택을 관리하는 오브젝트
-   Table View Controller : Table View를 관리하는 오브젝트
-   Collection View Controller : Collection View를 관리하는 오브젝트
-   Tab Bar Controller : Tab bar 항목인 View Controller를 관리하는 오브젝트
-   Split View Controller : 두 개의 View Controller를 반반 나눠서 보여주는 오브젝트
-   Page View Controller : 일련의 View Controller를 통한 페이지 오브젝트
-   Hosting View Controller : Content가 SwiftUI View인 UIViewController
-   GLKit View Controller : 모든 표준 View Controller 기능을 제공하고, OpenGL ES 렌더링 루프를 구현하는 오브젝트
-   AVKit Player View Controller : 시청각 컨텐츠를 제공하는 오브젝트
-   Look Around View Controller : Map에서 로드뷰같은 기능을 제공하는 오브젝트(기본 설명 참조)
-   Object : 오브젝트를 인스턴스화 시키는 오브젝트

### 6\. Gesture Recognizer

![](assets/img/post/2022/10_21_gesture_recognizer.png)

-   Tap Gesture Recognizer : 간단한 Tab에 반응하는 오브젝트
-   Pinch Gesture Recognizer : Pinch(두 손가락으로 오므리는) 동작에 반응하는 오브젝트
-   Rotation Gesture Recognizer : Rotation(두 손가락으로 회전하는) 동작에 반응하는 오브젝트
-   Swipe Gesture Recognizer : Swipe(쓸어내리는) 동작에 반응하는 오브젝트
-   Pan Gesture Recognizer : Pen(드래그하는) 동작에 반응하는 오브젝트
-   Screen Edge Pan Gesture Recognizer : 가장자리에서 Pen 동작에 반응하는 오브젝트
-   Long Press Gesture Recognizer : 길게 누르는 동작에 반응하는 오브젝트
-   Custom Gesture Recognizer : 사용자 정의 동작에 반응하는 오브젝트