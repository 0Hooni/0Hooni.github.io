---
title: "[SwiftUI] @State, @Binding 정리"
date: 2024-01-26 01:04 +0900
categories:
  - Mobile
  - iOS
tags:
  - SwiftUI
---
SwiftUI를 처음 시작하면 자주 보게되는 `@State` 와 `@Binding` 이라는 속성이 있다.

지금까지 이 둘의 사용법은 알고 있었지만, 이번에 블로그 포스팅을 하며 공식 문서를 뜯어보며 좀 더 자세히 알아보려 한다.

## @State

### 개요

-   `@State` 는 View 내에서 변경 가능한 상태를 나타낸다.
-   해당 속성에 변화가 생기면 SwiftUI가 자동으로 View를 업데이트 한다.

### 활용

-   주로 View 안에서 특정 상태를 추적하고 업데이트하는 데 사용된다.
-   View가 자체적으로 상태를 관리할 수 있게 한다.

### 코드

```swift
struct ContentView: View {
    @State private var counter = 0

    var body: some View {
        VStack {
            Text("Current counter is \(counter)")
            Button("Increment") {
                counter += 1
            }
        }
    }
}
```

위의 예제 코드에서 보면 `counter`변수가 `Button`을 통해 값이 증가하도록 설계하였다.

만약 `@State`로 선언되지 않았다면 버튼을 눌러도 `Text`안에 `counter` 값이 View에 반영되지 않았을 것이다.

하지만 `@State`을 이용함으로써 값의 변경이 정상적으로 View에 반영될 수 있었던 것이다.

**이렇게 `@State`는 버튼 클릭이나 사용자 입력과 같이 View의 상태를 추적하고 업데이트 하는데 사용된다.**

#### 주의사항

-   `@State` 속성은 해당 View에서만 유효하며, 다른 View 간에 공유되지 않는다.

## @Binding

### 개요

-   `@Binding` 속성은 View 간에 데이터를 공유하고 동기화하는데 사용된다.
-   부모 View의 데이터를 자식 View로 전달하면서, 자식 View에서 해당 데이터를 변경하면 부모 View에도 반영 된다.

### 활용

-   주로 부모 View와 자식 View 간에 데이터를 공유하고 양방향으로 상태를 조작하는데 사용된다.
-   자식 View에서 변경된 데이터를 부모 View에 반영할 때 유용하게 활용된다.

```swift
struct DetailView: View {
    @Binding var counter: Int

    var body: some View {
        VStack {
            Text("Counter in DetailView: \(counter)")
            Button("Increment in DetailView") {
                counter += 1
            }
        }
    }
}

struct ContentView: View {
    @State private var counter = 0

    var body: some View {
        VStack {
            Text("Counter in ContentView: \(counter)")
            DetailView(counter: $counter)
        }
    }
}
```

위의 예제에서 `DetailView`는 `@Binding` 속성을 사용하여 `ContentView`의 `counter` 값을 전달받아 사용한다.

`DetailView`에서 버튼을 누르면 `ContentView`의 `counter` 값이 업데이트되고, 이렇게 두 View 간에 데이터가 공유됩니다.

### 주의사항

-   `@Binding` 속성은 부모 View의 `@State`를 자식 View와 연결시켜 동기화한다.
-   주로 데이터의 양방향 흐름을 구현하는 데 활용된다.

## 결론

SwiftUI에서의 `@State`와 `@Binding` 속성은 View의 상태를 효과적으로 관리하고 데이터를 공유하는 데 필수적이다. 상태의 변경을 추적하고 다른 View로 데이터를 전달하는 데 사용되며, 이를 통해 간결하고 유지보수가 쉬운 코드를 작성할 수 있다.

> ### 참고 문서
> 
> -   [@State /| 애플 개발자 문서](https://developer.apple.com/documentation/swiftui/state)
> -   [@Binding /| 애플 개발자 문서](https://developer.apple.com/documentation/swiftui/binding)