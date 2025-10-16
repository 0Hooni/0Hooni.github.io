---
title: "[Swift] PropertyWrapper를 이용한 DIContainer 만들기 - Layer간 의존성 흐름 개선하기"
date: 2025-04-17 17:17 +0900
categories:
  - 🍎 iOS
  - Swift
tags:
  - Swift
  - PopPool
---
지금 참여하고 있는 프로젝트에서 3-layer 클린 아키텍처를 도입하고 있었다.  

하지만 논리적으로만 분리되어 있다 보니, 실제 구현에서는 ViewModel이 UseCase나 Repository 같은 구체 타입을 직접 의존하는 구조가 많았다.

ViewModel에서 구체 구현체를 직접 생성하거나 의존하다 보니 **Presentation → Domain → Data로 가는 방향이 아닌, 모든 계층이 서로 얽히는 문제**가 있었다.

덕분에 물리적으로 모듈을 분리하려는 시도조차 할 수 없는 상황이 되어버렸다.

기존 코드를 모두 뜯어고치는 데에는 부담이 컸기 때문에, 단계적으로 Presentation Layer에서부터 의존성 주입 방식을 도입하기로 했다.

하지만 이미 출시된 앱이고, ViewModel의 수가 상당하다 보니 ViewModel에 간편하게 DI를 할 수 있는 구조가 필요했다.

그래서 선택한 방법이 바로 **PropertyWrapper + DIContainer** 조합이었다.

> **\[팝풀] DIContainer 및 의존성 제거** 
> 
> [[GitHub PR] DIContainer 구현 및 Presentation layer에 외부 의존성 제거](https://github.com/PopPool/iOS/pull/115)

## 🤔 DI란 무엇인가

DI(Dependency Injection)는 객체가 필요로 하는 의존성을 외부에서 주입받는 패턴이다. 

이 방식은 **코드의 결합도를 낮추고, 테스트 가능성과 유연성을 높인다.**

예를 들어 아래 코드를 보자

```swift
class MyViewModel {
	let useCase = UseCaseImpl()
}
```

이 구조에서는 `MyViewModel`이 `UseCaseImpl`이라는 구체 타입에 강하게 의존한다.

하지만 다음과 같이 DI를 적용하면

```swift
class MyViewModel {
	let useCase: UseCase

	init(useCase: UseCase) {
		self.useCase = useCase
	}
}
```

`MyViewModel`은 `UseCase`라는 추상 타입만 알게 되고, 실제 어떤 구현체가 들어올지는 외부에서 주입받기 때문에 결합도가 낮아진다.

DI의 장점은 이제 알겠으니 그러면 왜 DIContainer를 만들었을까?

## 📦 DIContainer란 무엇인가

의존성을 하나하나 생성자에 주입하는 방식은 명확하지만, 규모가 커질수록 불편하고 반복적인 코드가 많아진다.

_**(특히 지금과 같은 상황이면 그 작업이 배로 늘어난다)**_

이를 보완하기 위해 **DIContainer**를 만들었다. 

컨테이너는 **앱 시작 시점에 구현체를 등록**해두고, **필요한 시점에 타입 기반으로 꺼내** 쓸 수 있게 해준다.

```swift
// 등록
DIContainer.register(SampleProtocol.self) { return SampleImpl() }

// 주입
let sample: SampleProtocol = DIContainer.resolve(SampleProtocol.self)
```

> DIContainer는 내부적으로 
> - `ObjectIdentifier`를 사용하여 타입별로 저장된 클로저 관리
> - `DispatchQueue`를 이용해 스레드 안전성 확보

_**(나중에 알았지만 데이터의 읽기시에는 주소를 참조하기에 데이터 레이스는 발생하지 않는다 한다. 🥲)**_

이렇게 만들어진 DIContainer를 사용하면 이전의 의존성 주입을 생성자에서 처리해줄때보다 더 간편하게 의존성을 관리해줄 수 있다.

## ✨ PropertyWrapper로 의존성 주입하기

하지만 결국 의존성을 가져올때는 계속 `DIContainer.resolve(SampleProtocol.self)`을 계속 쳐줘야된다.

의존성을 꺼내는 코드를 계속 반복해서 작성해야 되는 내 상황에서는 생각보다는 번거로운 작업이었다.

그래서 `@Dependency`라는 propertyWrapper를 만들어 DIContainer와 연결되도록 했다.

#### **@Dependency 구현**
```swift
@propertyWrapper
public final class Dependency<T> {
    public private(set) var wrappedValue: T

    public init() {
        self.wrappedValue = DIContainer.resolve(T.self)
    }
}
```

#### **@Dependency 사용**

```swift
class MyViewModel {
    @Dependency var sample: SampleProtocol
}
```

1차적으로는 외부에서 생성자에 DIContainer를 이용해 의존성을 주입해준다.

하지만 의존성을 바로 꺼내써야 되는 경우, 위처럼 선언만 하면 **DIContainer로부터 자동으로 해당 타입을 꺼내서 주입**해준다.

덕분에 ViewModel 내부는 훨씬 가볍고 깔끔해지고, DI 적용도 자연스럽게 흘러들어갈 수 있었다.

근데 이제 PropertyWrapper로 만들면서 생긴 문제가 있다.

## 🔒 PropertyWrapper 불변성을 위한 고민

Swift에서 PropertyWrapper를 사용할 때는 반드시 var로 선언해야 한다.

> 이는 프로퍼티 래퍼가 내부적으로 값을 읽고 쓸 수 있어야 하기 때문이다.

```swift
@Dependency var sample: SampleProtocol
```

하지만 외부에서 이 프로퍼티를 수정하는 건 원치 않았다.

ViewModel에서 UseCase를 의도치 않게 바꾸거나 바뀌어질 수 있음을 고려했다.

_**(물론 그런 코드는 없긴 했지만...😅)**_

그래서 propertyWrapper 내부의 wrappedValue를 private(set)으로 제한했다.

이렇게 하면 값은 초기화 시 한 번만 설정되고, 외부에서는 읽기만 가능해서 **불변성을 유지**할 수 있었다.

## 🏁 마무리

DIContainer와 propertyWrapper를 함께 사용하면서, 기존의 구조는 거의 그대로 두고 ViewModel 단에서 추상화된 의존성을 사용하게 만들 수 있었다.

또한 강한 결합도를 낮출 수 있었고, ViewModel 내부에서 반복적으로 생성되는 구현 객체들을 DIContainer에 등록된 하나의 객체를 이용해서 처리할 수 있게 되었다. 

_**(이런게 다 리소스 낭비에요...!!!)**_

단계적 리팩토링이 필요했던 프로젝트에서 가장 현실적인 선택이었고, 앞으로 진행할 모듈화 작업에 큰 도움이 될거라 생각된다.

## 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
> - [[DI] DI Container, IOC Container 개념과 예제](https://eunjin3786.tistory.com/233)
>- [Apple - Property Wrappers](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/#Property-Wrappers)
>- [ObjectIdentifier \| Apple Developer Documentation](https://developer.apple.com/documentation/swift/objectidentifier)
