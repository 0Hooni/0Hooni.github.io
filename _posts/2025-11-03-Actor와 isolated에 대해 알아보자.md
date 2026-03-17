---
title: "[Swift] Actor와 isolated에 대해 알아보자"
date: 2025-11-03 16:13 +0900
categories:
  - Language
  - Swift
tags:
  - Swift
  - Concurrency
---
최근 간단한 비디오 데모 앱을 개발하며 **컴파일 타임 Thread Safety**를 직접 경험하기 위해 Swift 6를 도입했습니다.

이전까지는 Swift가 스레드 간 전환을 자동으로 처리해 주었기 때문에, 세밀하게 동시성 문제를 신경 쓰지 않아도 되는 경우가 많았습니다.

하지만 Swift 6에서는 Thread Safety를 강화하면서 스레드 관련 경고를 설정할 수 있고, 이를 **Strict 모드**로 두면 다수의 경고와 오류를 마주하게 됩니다.

이 과정에서 자연스럽게 `actor`, `@MainActor`, `isolated`, `nonisolated` 같은 새로운 키워드를 접하게 됩니다.

이번 글에서는 Swift 동시성 모델의 핵심인 **Actor 시스템**을 중심으로, 이 키워드들이 어떻게 스레드 안전성을 보장하는지 살펴보겠습니다.

## 🦺 Actor - Thread safety type

### 정의

Actor는 Class와 마찬가지로 참조 타입이지만, 내부 동작 방식이 다릅니다.

**Actor**는 **race condition**을 방지하며, **한 시점에 하나의 스레드만 내부 상태에 접근**하도록 보장합니다.

아래 예시를 보겠습니다.

~~~swift
// ❌ Class: 데이터 레이스 가능
class BankAccount {
    var balance: Int = 0

    func deposit(_ amount: Int) {
        balance += amount  // 🚨 여러 스레드에서 동시에 접근하면 문제 발생
    }
}

let account = BankAccount()
Task { account.deposit(100) }
Task { account.deposit(100) }
~~~

위 예제에서 `deposit()`을 여러 스레드가 동시에 호출하면 `balance` 프로퍼티에 race condition이 발생합니다.

하지만 이 문제를 **Actor**로 해결할 수 있습니다. 어떻게 가능한걸까요?

### 작동 원리 1 - Isolation

Actor를 사용하면 내부 프로퍼티는 **자신의 고유한 격리 영역(isolation)** 안에 존재하게 됩니다.

아까 Class를 Actor로 바꿔보겠습니다.

~~~swift
// ✅ Actor: 데이터 레이스 방지
actor SafeBankAccount {
    var balance: Int = 0

    func deposit(_ amount: Int) {
        balance += amount  // ✅ 한 번에 하나의 작업만 접근 가능
    }
}

let account = SafeBankAccount()
Task { await account.deposit(100) }
Task { await account.deposit(100) }
~~~

이제 내부 프로퍼티는 `SafeBankAccount`의 격리 영역에 존재하게 됩니다.

이러한 격리 영역을 만듦으로써 Actor는 **Thread safety**를 위한 기반을 얻습니다.

### 작동 원리 2 - Serial Executor

Actor는 **한 시점에 오직 하나의 Task만** 내부 상태에 접근할 수 있는데요.

이는 Actor가 내부적으로 **serial executor**를 가지고 있기 때문입니다.

serial executor는 **Actor 내부의 코드를 순차적으로 처리**하도록 만듭니다.

그렇기에 한 시점에 오직 하나의 Task만 들어오고, 격리 영역에 대한 접근 또한 동시에 발생하지 않는 것이죠.

위 예제 코드를 보면 이제 외부에서 actor의 상태를 바꾸기 위한 `deposit()` 함수를 사용하기 위해 await를 사용하는 모습을 볼 수 있는데요.

이 또한 외부에서 접근 시 이를 직렬적으로 처리해주기 위함입니다.

### 결과적으로

Actor는 다음과 같은 방식으로 Thread Safety를 달성합니다.

- 객체 상태를 내부적으로 격리하고
- isolation을 다룰때는 직렬적으로 접근하여
- race condition을 방지하고
- Thread Safety를 달성한다

## 🌍 @MainActor와 Global Actor

### @MainActor란?

방금 이해한 Actor가 백그라운드 스레드를 위한 것이었다면 

`@MainActor`는 메인 스레드를 위한 Actor라고 볼 수 있습니다.

일반적으로 UI 요소는 항상 메인 스레드에서만 동작해야 합니다. 

그래서 이를 보장해주기 위해 UIKit, SwiftUI의 대부분의 화면들은 @MainActor를 채택해주고 있습니다.

하지만 추가적으로 메인 스레드에서 동작하도록 보장하게 만들고 싶다면 직접 `@MainActor` 키워드를 붙여주면 되는데요.

아래의 예시를 보겠습니다.

~~~swift
@MainActor
class ContentViewModel: ObservableObject {
    @Published var displayText: String = ""

    func updateText() {
        displayText = "Updated"  // ✅ 메인 스레드에서만 실행됨
    }
}
~~~

만약 `@MainActor` 어노테이션을 따로 붙여주지 않았다면 `displayText`의 값이 변하는 시점의 스레드가 어디인지 보장받을 수 없습니다.

이를 위해 여러 방법이 있겠지만, 예시와 같이 해당 ViewModel에 `@MainActor` 어노테이션을 붙여준다면 내부의 동작들은 모두 메인 스레드에서 실행한다는 보장을 해줄 수 있는것이죠.

### Global Actor란?

Global Actor는 **전역적으로 공유되는 단일 Actor**입니다. `@MainActor` 또한 Global Actor입니다.

일반 Actor가 독립된 isolation 영역을 가지는 반면, Global Actor는 앱 전체에서 하나의 isolation 영역을 공유합니다.

~~~swift
@MainActor
class ViewModel1 { var data = "" }

@MainActor
class ViewModel2 { var data = "" }

// ViewModel1과 ViewModel2는 같은 MainActor 격리 영역을 공유
~~~

## 🫥 isolated와 nonisolated

### isolated — 격리 영역 내부 (기본 상태)

Actor 내부의 모든 프로퍼티와 메서드는 기본적으로 **isolated 상태**입니다.

즉, Actor의 격리 영역 안에서만 접근할 수 있으며 외부에서는 `await`를 통해서만 접근 가능합니다.

~~~swift
actor Counter {
    var count = 0  // isolated

    func increment() {  // isolated
        count += 1
    }
}

let counter = Counter()
await counter.increment()  // ⚠️ await 필요
~~~
### nonisolated — 격리 영역 외부

actor 인스턴스의 모든 코드들이 serial executor에 의해 직렬적인 수행을 필요로 하지 않을 수 있습니다.

이럴 때는 `nonisolated` 키워드를 붙이면 해당 메서드는 Actor의 격리 영역 밖에서 실행됩니다.

아래 예시와 같은 상황에서 유용할 수 있습니다.

~~~swift
actor Calculator {
    var history: [Int] = []  // isolated

    func addToHistory(_ value: Int) {
        history.append(value)
    }

    nonisolated func calculate(_ a: Int, _ b: Int) -> Int {
        return a + b  // ✅ 상태 접근 없음, await 불필요
    }
}
~~~

예시처럼 `calculate(_:, _:)` 함수는 전혀 Actor 내부 상태를 변경하는데 관심이 없습니다. 

이렇게 Actor의 상태에 접근하지 않거나, 순수 계산 로직일 때 유용합니다.

### 차이 정리

| 구분 | isolated (기본) | nonisolated |
|------|------------------|--------------|
| Actor 상태 접근 | ✅ 가능 | ❌ 불가능 |
| 외부 호출 시 await | ⚠️ 필요 | ✅ 불필요 |
| 실행 순서 보장 | 순차적 | 병렬 가능 |
| 사용 예 | 상태 변경, 내부 로직 | 순수 계산, 불변 데이터 |

## 🏁 마무리
### Swift 동시성 핵심

| 키워드           | 설명                     | 사용 시점             |
| ------------- | ---------------------- | ----------------- |
| `actor`       | 스레드 안전한 참조 타입          | 공유 상태를 안전하게 관리할 때 |
| `@MainActor`  | 메인 스레드 전용 Global Actor | UI 업데이트 로직        |
| `isolated`    | Actor의 격리 영역 내부        | 상태 변경 메서드         |
| `nonisolated` | Actor 격리 영역 외부         | 순수 계산, 불변 데이터     |

정리해보면

Actor는 Class가 가질 수 있는 race condition에 대한 문제를 해결해주기 핵심 도구입니다.

내부 상태를 격리하고, 직렬적으로 코드를 수행하여

최종적으로는 Thread safety한 코드를 만드는 것이죠.

또한 상황에 따라 `isolated`와 `nonisolated`를 조합한다면 더 유연한 동시 제어가 가능해집니다.

이제는 자신있게 actor와 `@MainActor`를 사용할 수 있고, 상황에 따라 제어자를 두어 적절한 concurrency를 사용할 수 있을것 같습니다.

## 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [Actor \| Apple Developer Documentation](https://developer.apple.com/documentation/swift/actor)
>- [SerialExecutor \| Apple Developer Documentation](https://developer.apple.com/documentation/swift/serialexecutor)
>- ['iOS/Concurrency' 글 목록](https://dev-with-precious-dreams.tistory.com/category/iOS/Concurrency)