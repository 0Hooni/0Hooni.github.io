---
title: "[UIKit] UIKit에서의 MVVM은 왜 쓰는것일까?"
date: 2024-09-24 01:08 +0900
categories:
  - 🍎 Dev
  - UIKit
tags:
  - UIKit
  - Pattern
  - 부스트캠프
---
부스트캠프에서 3~4주차 스프린트를 진행하는 동안 정말 다양한 사람들의 코드를 볼 수 있었다. 그러던중 UIKit에서 MVVM 패턴을 사용하는것을 보기도 했고, 이번 5~6주차 스프린트 팀원과 같이 모각코를 하던중 MVVM 패턴을 이용해서 설계한다는 말을 들으면서 사람들은 왜 UIKit에서 권장하는 MVC를 뒤로하고 MVVM을 선택하는지, MVC의 문제는 무엇이며, UIKit에서 사용하는 MVVM의 장점은 무엇인지 알아볼까 한다.

## 🧓🏻 MVC 패턴
MVC는 애플의 UIKit에서 기본적으로 권장하는 아키텍처 패턴이다. 

MVC패턴은 다음 세 가지 주요 구성요소로 이주어져 있다.
- **Model:** Application의 데이터 구조와 비즈니스 로직을 담당
- **View:** 사용자에게 보여지는 UI 요소를 담당하며, 사용자와의 상호작용을 처리
- **Controller:** 모델과 뷰를 연결하는 중재자 역할. 사용자 입력을 처리하고 모델, 뷰를 업데이트함

MVC 패턴의 구조는 알겠다. 그렇다면 MVC 패턴의 문제점은 무엇일까? 우선 UIKit에서 MVC 패턴을 사용하며 겪는 문제점은 아래와 같다
### Massive View Controller - 대형 뷰 컨트롤러
MVC 패턴에서는 View와 Controller가 밀접하게 결합되어 있다. 그렇기에 복잡한 UI 로직과 비즈니스 로직이 ViewController에 집중되는 경향이 있다. 

이로 인해 ViewController의 코드가 지나치게 커지고 복잡해지며, 유지보수가 어려워진다는 문제가 있다. 또한 코드의 가독성이 떨어지고, 재사용성이 낮아지며, 테스트가 어려워진다. 

실제로 지난 스프린트 기간동안 MVC 패턴을 집중적으로 적용해보며 느낀것이 Controller가 비대해진다는 문제점과 주석과 extension을 적절히 이용하지 못하면 코드를 보기 매우 불편해진다는것을 몸소 겪었던것 같다. 

이러한 대형 ViewController의 문제점은 다양한 로직을 컨트롤러에서 처리하게되면서 오히려 책임이 불명확해진다는 부분도 존재한다.

또한 이렇게 하나의 컨트롤러로 각각의 View와 Model을 연결하고 관리하다보니 Controller의 코드는 재사용성이 어렵다는 부분 또한 존재한다.
## 👨🏻 MVVM 패턴
이렇듯 비대해지는 Controller와 오히려 역할이 모호해지는 부분으로 인해서 MVC의 한계가 들어났고, 이를 파해치기 위해 MVVM이라는 파생 아키텍처 패턴이 등장하게 된다. 

MVVM 패턴은 Model-View-ViewModel로 구성되어어 있다. 
- **Model:** MVC의 Model과 동일
- **View:** MVC의 View와 동일
- **ViewMode:** Model과 View 사이의 중개자 역할이다. 데이터를 가공하여 View에게 제공하고, View의 사용자 입력을 모델에 전달한다. 

우리가 아는 MVC에서 Controller의 다양한 로직들을 담아내고 있다고 보면 된다. 

그렇다면 이러한 MVVM 패턴은 MVC대비 어떠한 이점을 가져다 줄까?
### 관심사 분리
MVVM은 View와 Model 사이에 ViewModel을 두어 UI 로직과 비즈니스 로직을 명확히 분리한다.

이를 통해 코드의 가독성과 유지보수성이 향상된다.

### 데이터 바인딩
MVVM에서는 데이터 바인딩을 통해 ViewModel과 View 간의 동기화가 용이하다. 이를 통해 UI 업데이트를 자동으로 처리하게 할 수 있다.

### 재사용성
ViewModel을 UI에 종속되지 않기 때문에 다른 View와 쉽게 재사용할 수 있다는 장점이 있다. 

## 🤷🏻‍♂️ UIKit에서 MVVM을 적용하는 방법
이렇듯 MVVM패턴은 확실히 MVC가 가지는 단점을 시원하게 해결해주는 패턴이다. 물론 어느 프로젝트를 하느냐에 따라 각각의 패턴이 가지는 장점을 더욱 살릴 수 있기때문에 무조건 MVVM이 좋다고 할 수는 없다. 

본인의 경우 지난 스프린트를 거치며 MVC가 가지는 단점에 대해 몸소 느끼면서 MVVM 패턴에 대해 고려를 해볼까 싶다.

UIKit을 사용하면서 MVVM패턴은 어떻게 적용해야될까?

### UIKit(MVVM) 예시
[Github - UIKit MVVM](https://github.com/0Hooni/iOS/tree/main/Practice/UIKit%20MVVM%20Test)

생각보다 길어져서 깃허브에 올렸다. MVVM에 맞게 해당 코드를 분석하자면
- **Model:** User 구조체에서 사용자의 이름을 저장
- **ViewModel:** UserViewModel 클래스는 User 데이터를 관리하며, Combine 프레임워크를 사용하여 데이터 바인딩을 제공한다. 여기서는 @Published 속성을 통해 UI와 데이터를 연결한다.
- **View:** UserViewController 클래스는 UI를 구성하고 사용자 입력을 처리한다. 텍스트 필드, 버튼, 레이블을 사용하여 인터페이스를 구성하였다.

## 🎬 결론
이번에 가볍게 MVVM과 MVC에 대해 다뤄보았고 MVC를 권장하는 UIKit에서는 어떻게 MVVM을 적용해야될까에 대해서도 연습을 해보았다. 

우선 MVVM을 사용하기 위해서는 Combine 프레임워크는 필연적으로 따라오는것 같아서 공부가 좀 더 필요하다고 느꼈다.