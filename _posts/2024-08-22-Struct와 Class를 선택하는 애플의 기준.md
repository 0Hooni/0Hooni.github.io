---
title: "[Swift] Struct와 Class를 선택하는 애플의 기준"
date: 2024-08-22 00:23:21 +0900
categories:
  - 🍎 iOS
  - Swift
tags:
  - Swift
  - DataStruct
  - CS
  - boostcamp
---
여러개의 자료형들을 묶어 하나의 타입으로 만드는 방법의 대표적인 방법에는 `Struct`와 `class`가 있다.

평소 `Class`를 사용하는것이 익숙해서 자주 이용을 하는데 이번에 이 둘의 차이에 대해 알아볼 계기가 생겨 조사할 겸 포스팅을 남겨볼까 한다.

# 🍏 우선은 개발자 문서
가장 먼저 살펴본것은 애플 개발자 문서인데 이 둘의 선택을 할때에 대해 간략하게 정리를 해둔것이 있다.
> [애플 개발자 문서](https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes)
> - 기본적으로 Struct를 활용해라
> - Objective-C와 상호 운용성이 필요하다면 Class를 사용해라
> - 모델링할 데이터의 Identity를 제어하는 경우 Class를 사용해라
> - 프로토콜을 통해 동작을 공유할때는 Struct를 사용해라

각각의 선택에 대한 이유도 같이 설명을 해줬는데 각각의 이유는 다음과 같았다.
### 기본적으로 Struct를 활용해라
Swift의 Struct는 Class에서는 제한되는 많은 기능을 포함한다. 예를 들어 속성들을 저장하거나, 계산하는 등등이 있다. 또한 Struct는 프로토콜을 채택해서 기본적인 행동을 포함시킬 수 있다.

Swift의 표준 라이브러리와 Foundation에 우리가 자주 사용하는 Number, String, Array, Dictionary 타입들 또한 Struct타입이다.

이렇듯 Struct를 사용하면 앱 전체의 상태에 신경쓰지 않고 쉽게 구조체를 만들수 있다. 왜냐하면 Struct는 Class와 다르게 값 타입이기에 로컬 변경 사항은 앱 흐름의 일부로 의도적으로 전달하지 않은 한 앱의 나머지 부분에 표시되지 않기 때문이다. 

### Objective-C와 상호 운용성이 필요한 경우 Class를 사용
데이터를 처리해야 하는 Objective-C API를 사용하거나 Objective-C 프레임워크에 정의된 기존 클래스 계층 구조에 데이터 모델을 맞춰야 하는 경우 Class 상속을 이용하여 데이터 모델링을 해야할 수 있는데. 이럴때 사용하면 좋다.

### Identity를 관리해야될 때
Class는 기본적으로 참조 타입이기 때문에 Identity에 대한 정보가 내장되어 있다. 그렇기에 만약에 두개의 Class 인스턴스가 같은 값을 가지더라도 Identity가 다르기 때문에 둘을 다르게 인식된다.

주로 Class는 파일 관리, 네트워크 연결, 공유 하드웨어 중개를 할 때 사용된다. 

### 정리
전반적으로 Swift에서는 Identity나 Objective-C와 관련된 상속이 필요한것이 아니라면 Struct의 사용을 권장하고 있는것 같다.

# 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
> - [Choosing Between Structures and Classes \| Apple Developer Documentation](https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes)
> - [[Swift] Class와 Struct의 차이점?](https://icksw.tistory.com/256)
> - [[Swift] 싱글톤 패턴에서 구조체와 클래스의 차이](https://so-kyte.tistory.com/169)