---
title: "[UIKit] DiffableDataSource의 동작과 Hashable 프로토콜 이해하기"
date: 2025-05-08 14:36 +0900
description: "UIKit DiffableDataSource의 동작 원리와 Hashable 프로토콜의 역할을 이해합니다."
categories:
  - Mobile
  - iOS
tags:
  - tutorial
---
최근에 팝풀의 검색화면에서 발생하는 플리커링 문제를 해결하는 과정에서 기존의 DataSource를 Diffable로 교체한 경험이 있었습니다.

DiffableDataSource는 흔히 반복적인 변경이 일어나는 TableView나 CollectionView에서 많이 도입한다고 말하죠. 잦은 변경에 용이한 구조라 하면서.

하지만 실제로 DiffableDataSource는 어떻게 기존의 DataSource보다 빠르게 데이터를 처리할 수 있는지, DiffableDataSource의 snapshot은 무엇인지, Section의 Item은 왜 Hashable을 채택하는지 결과를 보면 얼추 이해는 가지만 내부적으로는 이해하기 어려운것들이 많았습니다.

그래서 오늘은 DiffableDataSource의 동작을 이해해보고 또 단순 비교만 필요했다면 Equatable만 채택해줘도 될텐데 Hashable이어야 했던 이유 등을 알아가보려고 합니다.

## 📌 DiffableDataSource란?

DiffableDataSource는 WWDC 2019에서 발표된 데이터 소스 객체입니다. 일반적인 프로토콜 형태로 채택하던 DataSource와는 별개로 별도의 객체를 생성하여 delegate를 설정해주는 방식으로 선언부터 조금 다른 형태를 띄고 있어요.

그렇다면 애플은 왜 기존의 DataSource 프로토콜을 놔두고, 사용방식이 달라진 DiffableDataSource를 발표했었을까요?

### 기존 DataSource의 문제

![](assets/img/post/2025/05_08_데이터_소스_에러.png)_(나야 데이터 소스 에러)_

아마 기존의 DataSource를 사용했을 때 많이들 경험했을 에러일것입니다. Controller로부터 상태 변경을 전달받고 UI를 업데이트 할 때 빈번히 나타나는 에러죠.

해당 에러를 좀 더 상세하게 설명드리자면, 업데이트 이전의 섹션의 수는 10개였고, 업데이트 후 섹션의 수도 10개다. 그런데 시스템에서는 섹션을 1개 삭제했다고 판단한다. 

UI를 업데이트할 때 DataSource의 섹션 수와 실제 UI 업데이트 내용이 불일치하여 발생하는 오류인 것입니다. 각자가 자신의 Truth를 갖고 있는데 두 개의 Truth가 불일치하면서 에러를 뱉어낸것이죠.

그래서 클래식한 DataSource에서는 이러한 문제를 해결하기 위해 자주 흑마법을 사용합니다.

**🪄 reloadData()**

하지만 흑마법의 대가로 아무런 UI 애니메이션이 일어나지 않습니다. 사용자의 경험을 해치는 주요 요인이었던 것이죠.

실제로 이는 애플에서도 많이 인지를 하고 있던 문제였고, [Advances in UI Data Sources](https://developer.apple.com/videos/play/wwdc2019/220/) 영상의 초반부에서도 언급을 해주고 있습니다.

그래서 이를 개선하기 위해 새로운 DataSource가 등장합니다.

### DiffableDataSource의 등장

![](assets/img/post/2025/05_08_데이터_소스_핵심_변경.png){: width=400}

기존의 DataSource에서 여러 Update를 처리를 해주기 위해 `performBatchUpdates()`를 사용하였습니다. 또한 내부적으로는 여러 비교를 통해 적절한 업데이트 처리도 개발자의 몫이었죠. 하지만 DiffableDataSource에서는 간단하게 `apply()`만 실행해주면 모든게 해결해주도록 해주었습니다.

그러면 뭘 apply한다는 걸까요?

## 🧩 Snapshot

그건 바로 Snapshot을 DataSource에 적용해주는것입니다. 

다들 DiffableDataSource를 사용하면서 처음 등장하는 친구에 어떻게 써야되는건지 많이 당황을 합니다.

이 친구는 왜 있는걸까요?

기존 DataSource의 문제는 쉽게 말해 두 개의 Truth가 충돌한다는 것입니다. 그러면 이걸 해결하기 위해선 진짜 하나의 Truth를 만들어줘야겠죠?

Snapshot은 그 역할을 합니다. UI 상태의 진짜 단 하나의 Truth를 목표하는 객체입니다.

또한 Section과 Item을 IndexPath 형태로 관리하던 방식에서 **고유한 ID를 갖도록 해줍니다.**

덕분에 개발자는 변경을 처리하기 위해 매번 IndexPath를 직접 비교해주던 상황에서 변경을 해시값을 이용해 알아서 비교해주는 편리함을 얻을 수 있게 됩니다.

이제 저희는 DiffableDataSource가 왜 등장했는지, Snapshot은 무슨 용도인지 알 수 있게 됐습니다.

그렇다면 왜 DiffableDataSource는 기존의 DataSource보다 더 변경에 유리한 것일까요?


## ⚙️ DiffableDataSource가 왜 변경에 유리한가?

지금까지 학습한 내용을 근거로 봤을때 우선 가장 큰 장점은 복잡한 변경을 'performBatchUpdates()'가 아닌 snapshot을 'apply()'하는 것 만으로도 쉽게 적용을 할 수 있기에 기존의 DataSource 대비 변경이 편하다라는 것이 있습니다.

더 나아가 성능적인 장점도 존재하는데요. 크게 두 가지 정도를 얘기할 수 있습니다.

### 비교 알고리즘

기존의 DataSource에서는 개발자가 변경에 대해 직접 비교를 하며 변경을 처리해주어야 했습니다. 알고리즘에 대해 익숙하지 못한 개발자라면 이 과정에서 비교를 매우 비효율적으로 처리하게 되겠죠.

하지만 DiffableDataSource에서는 이러한 비교를 자체적으로 처리하여 최소한의 변경만 해도 되는 환경을 만들어줍니다.

### 백그라운드 이용

Diff를 처리하는 모든 아이템들을 비교해야 되기 때문에 그 과정에서 자연스럽게 O(N)의 시간복잡도를 가지게 됩니다.  만약 아이템이 매우 많아진다면 이 시간도 분명 적지 않게 걸리게 되겠죠. 

DiffableDataSource는 이 때 작업이 오래 걸릴것을 대비하여 비교 연산은 백그라운드 스레드에서 처리해주고, 연산이 끝나 UI 업데이트가 필요해지면 자동으로 메인 스레드에서 처리해줍니다.

덕분에 기존 DataSource보다 DiffableDataSource가 얻는 이점들이 매우 많아지는 것이죠.

## 🤔 Diffable을 공부하면서 궁금했던것들

> **(여기서부터는 제 개인적은 궁금점과 그를 어느정도의 추측성을 겸한 얘기입니다)**

### Section, Item이 Hashable이어야 하는 이유?

![](assets/img/post/2025/05_08_itemIdentifier.png){: width=400}_itemIdentifier(for: ) 공식 문서_

처음에는 DiffableDataSource가 비교에 유리하다는 얘기를 보고, 만약 단순한 비교만 필요했다면 Equatable만 채택해도 되지 않았을까 싶었습니다. 그렇지만 Section과 SectionItem은 반드시 Hashable이어야 하죠.

저는 이 Hashable의 채택 이유를 탐색의 이점을 살리기 위해서라고 생각합니다.

DiffableDataSource에서 제공되는 몇몇 탐색 메서드들의 디스크립션을 보면 O(1)의 시간이 소요된다고 적혀있습니다. O(1) 시간에 탐색되는 자료구조는 흔히 Dictionary와 같은 해시 테이블 형태가 있죠.

그렇기 때문에 SectionItem은 Hashable을 채택하고, SectionItem이 갖고 있는 고유한 데이터들 중 어떤 값들을 기반으로 SectionItem의 해시값을 구할것인지 `hash(into: )` 메서드를 정의하며, 내부적으로는 이 해시값을 Key로 SectionItem을 값으로 사용하는 해시테이블 형태로 구성되어있지 않을까 싶습니다.

그 덕분에 여러 탐색을 위한 메서드들이 O(1)의 시간으로 값을 찾아낼 수 있는 것이고, 빠른 탐색이 있기에 비교에 있어서도 유리한 이점을 가져갈 수 있었던 것입니다.

### Hashable은 왜 Equatable이어야 하는가?

Hashable을 채택하면 자연스럽게 Equatable이 채택됩니다. 내부적으로 이미 채택이 되어있기 때문이죠.

그렇다면 왜 Hashable은 Equatable이 필요한것일까요?

그건 아마 해시 테이블의 해시 충돌 문제와 깊은 연관이 있다고 생각합니다. 

일반적으로 해시테이블에 Key로 들어갈 수 있는 값은 무한합니다. 반면 해싱 함수의 결과는 유한하죠. 그래서 자연스럽게 해시 테이블은 해시 충돌이라는 문제가 발생합니다. 

해시 충돌이 발생하게 되면 다른 Key-Value임에도 내부적으로 해시 결과가 동일해지기 때문에 의도치 않은 데이터 변경과 같은 오류가 발생할 수 있습니다. 

그래서 이러한 해시 충돌을 해결하기 위해 체이닝 방식이나 개방 주소법 방식을 이용합니다. 

두 가지 해결 방식에 대해 간단하게 설명드리자면 체이닝 방식은 해시 충돌이 발생하는 값을 링크드 리스트 구조로 해시 충돌 시 값들을 연결시켜두고, 개방 주소법 방식은 비어있는 해시값을 찾아서 위치시켜주는 방식입니다.

그래서 추후 해시 충돌이 발생한 값을 찾기 위해서 필연적으로 비교 연산을 해야되고, 비교를 어떻게 할것인지 정의해주기 위해 Equatable이 채택되어있다고 생각합니다.

## 🏁 마무리

이렇게 DiffableDataSource의 등장과 동작방식, 추가적으로 궁금했던 것들을 해결했습니다.

Diffable을 처음 접했을 때 그냥 `apply()`하면 된다길래 묻지도 따지지도 않고 난발했었는데, 이렇게 정리함으로써 앞으로 DiffableDataSource를 어떻게 사용해나가야될지 명확히 알 수 있었습니다.


## 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [Advances in UI Data Sources](https://developer.apple.com/videos/play/wwdc2019/220/)