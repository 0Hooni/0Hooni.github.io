---
title: "[SwiftUI] TCA 아키텍처를 ALABOZA"
date: 2025-04-05 21:43 +0900
description: "The Composable Architecture(TCA)의 핵심 개념과 구조를 알아봅니다."
categories:
  - Mobile
  - iOS
tags:
  - Architecture
  - TCA
---
예전에 [SwiftUI 아키텍처에 대한 학습](https://0hooni.github.io/posts/MVVM-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%ED%8C%A8%ED%84%B4-(1%ED%83%84-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)/)을 하다가 우연히 TCA를 알게 된 일이 있었다. 

간단하게 설명하자면 SwiftUI는 이미 MVVM 구조로 구성이 되어있기에, 여기에 MVVM을 적용하는 것은 날 수 있는 스케이트보드에 바퀴를 달아준 모양이라나. 

어느정도 공감이 되긴 했는데...

지금 생각해보면 사실 아키텍처 패턴이란 단순 코드의 흐름을 잡아주는것 뿐만 아니라 비즈니스 로직 분리와 같은 역할의 분리와 같은 중요점도 있으니 뭐.

_주절히 주절히 각설하고 👻_

그 당시 공부를 하다가 TCA가 SwiftUI에서 단방향 흐름을 만들어줄 수 있는 좋은 대안이었다는 것을 보고 계속 흥미만 갖고 있었는데

이번에 새롭게 만들어보는 토이 프로젝트에서 혼자 iOS를 담당하게 돼서 겸사겸사 하고 싶었던 것들을 다 저질러 볼 까 한다.

이번 글에서는 1차적으로 [TCA GitHub](https://github.com/pointfreeco/swift-composable-architecture)에 적혀있는 내용을 시작으로 정리하면서 TCA에 대해 알아가볼까 싶다.

## 📦 TCA

**The Composable Architecture**는 SwiftUI 애플리케이션을 **일관되고, 테스트 가능하며, 조합 가능한 방식으로** 구축하기 위한 아키텍쳐다.

> 이름에서 알 수 있듯 **Composable(조합 가능한)**은 TCA의 핵심 철학이다

TCA는 앱을 구성할 때, 하나의 거대한 구조 안에서 모든 걸 처리하지 않는다.

대신, 화면이나 기능 단위로 작은 로직들을 나누고, 이걸 필요할 때 조립하듯 이어붙일 수 있는 구조를 제공한다.

덕분에 조직화되고 재사용 가능한 설계를 쉽게 만들 수 있다.

또한 TCA의 또 다른 핵심 특징이 있다.

> 흐름 제어를 위한 단방향 데이터 통신을 지원한다

SwiftUI는 자체적으로 MVVM 구조를 따르고 있지만, 실제 복잡한 앱을 개발하다 보면 MVVM 만으로는 구조적 한계에 부딪힌다. 

예를 들어, 상태(State)의 흐름이 양방향으로 퍼지게 되면 어느 시점에서 어떠 값이 바뀌었는지 추적하기 어려워지고, 이로 인해 디버깅이 어려워지거나 사이드 이펙트가 어디에서 발생했는지 감을 잡기 힘들어진다.

TCA는 이런 복잡함을 막기 위해, 모든 데이터의 흐름을 한 방향으로만 흐르게 강제한다. 

_심지어 비동기 작업의 흐름마저도 😮_

그렇다면 TCA에서는 어떻게 앱을 조직화하고 또 데이터의 흐름을 강제할까

## ⚙️ TCA의 구성 요소 및 흐름

![TCA 데이터 흐름](assets/img/post/2025/04_05_TCA_데이터_흐름.png){: w="800"}_TCA 데이터 흐름_

> **위의 사진과 아래의 설명을 번갈아 보면 좀 더 이해가 쉽다**
{: .prompt-tip }

### State - 상태
> **앱의 현재 상태를 나타내는 순수 데이터 구조**

- Struct로 정의되며, View와 직접 바인딩되어 화면을 구성하는 데 사용
- 간단한 예시로 현재 카운트 값, 로딩 중 여부, 네트워크 응답 결과 등
- 모든 State는 Reducer에 의해 변경되고, 변경되면 View는 자동으로 리렌더링

```swift
struct CounterState: Equatable {
    var count: Int = 0
    var isLoading: Bool = false
}
```

### Action - 행동/이벤트
> **사용자의 입력이나 외부 이벤트를 나타내는 열거형**

- enum으로 정의되며, View에서 발생하거나 Effect의 결과로 전달
- Reducer는 이 Action을 받아서 적절한 처리를 수행

```swift
enum CounterAction: Equatable {
    case incrementButtonTapped
    case decrementButtonTapped
    case fetchData
    case dataResponse(Result<Data, Error>)
}
```

### Reducer - 상태 변경 처리
> **Action을 받아 State를 변경하고, 필요 시 Effect를 반환하는 순수 함수**

- Reducer<State, Action, Enviroment> 형태로 구성
- 모든 상태 변화는 Reducer를 통해서만 발생
- View 로직과 분리되어 있어 테스트가 용이

```swift
let counterReducer = Reducer<CounterState, CounterAction, CounterEnvironment> { state, action, env in
    switch action {
    case .incrementButtonTapped:
        state.count += 1
        return .none
    case .fetchData:
        return .run { send in
            let result = try await env.apiClient.fetchData()
            await send(.dataResponse(result))
        }
    }
    case .dataResponse(let result):
        state.data = result
        state.isLoading = false
        return .none
}
```

### Effect - 작업/비동기 처리
> **비동기 작업, 외부 시스템과의 상호작용, 사이드 이펙트를 명시적으로 표현하는 값**

- 주로 네트워크 요청, 타이머, 알림, 파일 I/O등 외부와의 상호작용을 표현
- 실행 후 다시 Action을 발생시켜 Reducer로 흘러들어감
- 그 결과로 새로운 Action이 발생
- 이 과정을 통해 **비동기 작업도 완전히 단방향 흐름 안에서 통제**

```swift
Effect.run { send in
	let result = try await env.apiClient.fetchData()
	await send(.dataResponse(result))
}
```

### Store - 상태 및 로직 관리 허브
> **View와 Reducer, State를 연결해주는 중심 객체**

- View는 Store를 통해 Action을 보내고, State를 구독
- 내부적으로는 State를 보관하고, Action이 오면 Reducer를 호출하고, 그 결과를 처리
- 필요하다면 Environment를 이용하여 외부 동작을 수행할 수 있음
- 마치 앱의 RunLoop와 같이 전체 흐름을 중앙에서 통제해주는 역할을 해준다 볼 수 있음

```swift
let store = Store(
    initialState: CounterState(),
    reducer: counterReducer,
    environment: CounterEnvironment()
)
```

### Environment - 외부 의존성
> 네트워크, 디스크, 타이머 등 외부 시스템과의 연결을 담당하는 구조체

- Reducer는 외부 의존성을 직접 알지 않고, Environment를 통해 간접적으로 사용
- 테스트 환경에서는 이를 쉽게 Mocking할 수 있어 테스트 작성이 쉬워짐

```swift
struct CounterEnvironment {
    var apiClient: APIClient
    var mainQueue: AnySchedulerOf<DispatchQueue>
}
```

## 🏁 마무리
이번 글을 통해 TCA의 기본 개념과 구성 요소를 간단히 정리해봤다. 

SwiftUI가 MVVM을 자연스럽게 따르고 있는 구조라고는 하지만, 실제로 복잡한 앱을 개발할수록 **역할 분리, 상태 관리, 비즈니스 로직의 예측 가능성**이 점점 더 중요해지기 마련이다.

그런 면에서 TCA는 SwiftUI에 단방향 흐름을 강제함으로써, **예측 가능하고 테스트 가능한 구조**를 만들어준다. 

물론 러닝커브가 있긴 하지만, 일단 익숙해지면 오히려 상태와 로직을 정리하고 관리하는데 큰 도움이 될 수 있다 생각한다.

> 아~ 벌써 재밌어 보인다 ㅋㅅㅋ 🤩

## 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [TCA GitHub](https://github.com/pointfreeco/swift-composable-architecture)
>- [[iOS] 내가 보려고 정리하는 TCA (The Composable Architecture) 기초](https://mini-min-dev.tistory.com/320)
>- [SwiftUI TCA - 3. State, Action, Reducer, Effect — Clamp](https://clamp-coding.tistory.com/516)