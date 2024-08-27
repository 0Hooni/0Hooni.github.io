---
title: "[UIKit] ViewController는 메모리 관리를 위해 ARC를 어떻게 활용할까?"
date: 2024-08-22 21:04:52 +0900
categories:
  - 🍎 Dev
  - UIKit
tags:
  - UIKit
  - ViewController
  - ARC
  - CS
---
부스트캠프에서 JK의 수업을 듣던 중 궁금해졌던 순간이 있다. 

어떤 멤버의 질문이 시작이었는데 "ViewController는 계속 쌓이나요?"와 같은 질문이었고 대답은 정확히 기억은 잘 안나지만 "ViewController는 호출되기 전에도 쌓여있을 수 있다." 뭐 아무튼 이런 대답인것으로 기억한다(바로 받아적어둘걸 😅)

핵심은 대답이 아닌 이 대답으로 시작한 내 궁금증이다. 

분명 ViewController는 자체 메소드로 메모리에서 해제될 때 호출되는 함수도 있기도 하고, 이미 참조 타입인 Class이기에 잘 관리하면 ARC가 수거해가지 않을까 싶었다. 그런데 저렇게 말을 하셔서 갑자기 궁금해졌다. 

실제로 UIKit은 아니긴 했지만 SwiftUI에서 지속적으로 View가 Stack되는 상황을 개선해보려 시도했던 기억도 있었고, 분명히 메모리에 영향을 줄 수 있는 부분이기에 사용되지 않는 View를 메모리에서 할당 해제를 하는것은 중요하기에 이번 기회를 통해 자세히 파헤쳐볼까 싶다.

# View가 메모리에 추가되는 상황
메모리 할당이 되기 전에 분명 View는 메모리에 추가된다. 그렇다면 어느 타이밍에 추가될까? 

생각을 해보면 우리가 Class를 사용하기도 전에 초기화를 하며 생성하듯, View도 분명 앱이 생성되면서 어딘가 초기화될것 같다는 생각은 든다.

무작정 찾으면 갈피가 안잡힐 것 같아 간단히 가설을 세워봤다. 

우선 첫번째 찾아본 레퍼런스는 역시 [애플 문서](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/ManagingMemory/Articles/MemoryAlloc.html)

처음부터 아카이브 문서라니 ㅠ


# 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [Tips for Allocating Memory (apple.com)](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/ManagingMemory/Articles/MemoryAlloc.html)