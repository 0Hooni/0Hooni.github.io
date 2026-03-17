---
title: "[Xcode] SPM Package Resolution Failed 에러 해결하기"
date: 2025-04-07 11:35 +0900
description: "Xcode에서 SPM Package Resolution Failed 에러가 발생하는 원인과 해결 방법을 정리합니다."
categories:
  - Mobile
  - iOS
tags:
  - SPM
  - troubleshooting
---
최근에 패키지를 설치하거나 제거하는 일이 몇번 있었는데

그럴때마다 `Package Resolution Failed`에러가 지속적으로 발생했다.

![](assets/img/post/2025/04_07_Package_Resolution_Failed.png)_나한테 왜그래~~_

저기서 `Add Anyway`를 선택하면 어떻게든 설치는 된다

문제는 몇몇 패키지의 경우 선택해서 설치할 수 있는데

그걸 못한다는 것이다.

이번 포스팅을 통해 해당 문제를 해결했던 과정을 적어볼까 한다.

> **얼른 해결하고 싶은 사람은 [6번으로](https://0hooni.github.io/posts/SPM-Package-Resolution-Failed-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0/#6-gitconfig-%EC%88%98%EC%A0%95)!! 👍🏻**

## 🔨 시도해본것들

### 1. SPM 3신기

기존에 패키지가 있었다면 **File → Packages**에 Xcode가 제공해주는 패키지 해결 3신기가 있다.

![](assets/img/post/2025/04_07_SPM_3신기.png){: w="400"}_SPM 3신기_

`Add Anyway`를 선택한 후 해당 방식들을 사용하면 어떻게 또 패키지들이 빌드 에러가 안되게 해결이 가능하다.

하지만 지금 문제는 SPM의 설치가 문제인 것이기에 기존의 설치된 패키지를 해결하는 문제는 진정한 해결방법이 아니었다.

### 2. 신규 프로젝트에서 설치

`Package Resolution Failed`에러가 발생하면 항상 기존의 패키지들과 충돌이 발생하는것으로 보여서 새로운 프로젝트에서 설치를 해봤지만 이 또한 문제가 아니었다.

### 3. Xcode 재설치

- 특정 버전을 지정하여 설치하는 방식
- App Store에서 Xcode 설치

두가지 방식 모두 해봤지만 마찬가지로 발생했다.

### 4. 관련 파일 삭제

해당 에러를 구글링해보면 원인이 될 수 있는 파일 두개가 존재한다.

![](assets/img/post/2025/04_07_캐시_파일.png){: w="400"}_관련 캐시 파일_

두 파일 모두 `~/Library/Caches/`{: .filepath}에 위치한다. 

#### **org.swift.swiftpm**

SPM과 관련된 설정, 캐시, 의존성 정보가 저장되는 용도로 사용
#### **com.apple.dt.Xcode**

Xcode의 설정, 사용자 환경, 시뮬레이터 정보, 캐시 등 다양한 데이터를 저장하는데 사용

해당 파일을 제거해보라 해서 제거한 뒤 SPM을 설치해봤지만 

**여전히 실패🥲**

### 5. disablePackageRepositoryCache

[Xcode 16 & Package load failure \| Apple Developer Forums](https://developer.apple.com/forums/thread/763897?answerId=809113022#809113022)

해당 포럼의 내용을 보다가 `xcodebuild -resolvePackageDependencies -disablePackageRepositoryCache`명령어를 실행해보라 했다.

무슨 명령어일까?

#### **`resolvePackageDependencies`**

패키지 의존성 해결 명령어이다.

아마 SPM 3신기중 resolve를 실행하는것으로 보였다.

#### **`disablePackageRepositoryCache`**

패키지 캐싱을 비활성화하는 명령어다.

기본적으로 SPM을 설치하면 로컬 캐싱을 해두는데, 이를 비활성화하는 목적으로 보였다.

해당 명령어를 실행하게 되면 매번 원격 저장소에서 새로 패키지를 가져오도록 강제하는것으로 보였다.

_하지만 마찬가지로 문제가 발생했다.🥲_

### 6. .gitconfig 수정

슬슬 화가 나고 내 맥북을 밀어버릴까 고민도 했다.

그러던 중 아래 링크를 발견했다.

[swift - Package resolution failed. Couldn't get the list of tags - Stack Overflow](https://stackoverflow.com/questions/69555397/package-resolution-failed-couldnt-get-the-list-of-tags)

사실 해당 레퍼런스를 안봤던것은 아니다.

하지만 해당 레퍼런스에서 보여지는 내용이 나와 연관이 없다 생각했었다.

![](assets/img/post/2025/04_07_스택오버플로우_베댓.png){: w="600"}_스택 오버플로우 베스트 댓글_

_왜냐하면 소스트리가 문제였다는 듯이 얘기를 했었으니까!!!_

소스트리는 Git을 편하게 사용하게 해주는 GUI 프로그램이다. 

물론 과거에 쓴적이 있긴 하지만, 거의 설치 후 맞지않아 바로 삭제했었다.

그런데 핵심은 소스트리가 아니라 **~/.gitconfig**였다.

#### **`.gitconfig`**

`.gitconfig`는 git의 환경설정 파일이라 볼 수 있다. 

`~/`{: .filepath}에 위치하는 local을 위한 git 설정을 모아둔 것이다.

사실 SPM은 Swift로 관리한다 뭐다 하면서 git이랑 연관성을 전혀 생각하지 않았었는데,

생각해보면 결국 패키지는 GitHub로 부터 받아온다. 

그렇기에 로컬에 있는 git 설정이 SPM에도 문제를 야기할 수 있던 것이다.

#### **해당 로컬 깃 설정은 뭐임?**

> bare repo 디렉토리를 안전한 경로(safe directory)로 등록해야 작업을 허용하도록 하는 설정

즉 git에게 bare repo를 사용할 때 명백히 동의한 경우에만 허용하라는 의미

Git 2.35 이후로부터 보안상의 이유로 bare repository로 보이는 디렉토리에서 작업하는 것을 제한하기 위해 생겼다한다.

덕분에 실수로 bare repo에서 작업하는걸 막을 수 있던 것이다.

안전한 경로로 디렉토리를 등록해주려면

```bash
git config --global --add safe.directory '*'
```

이처럼 전체 디렉토리를 안전하게 만들어줄 수 있고 

all 명령어 대신 특정 디렉토리를 넣어줘서 해당 디렉토리만 등록을 해줄수도 있다.
#### **Bare Repository?**

이게 뭔가 싶겠지. 

_나도 그랬으니까!!_

**Bare repository**란 코드 파일(작업 디렉토리)이 없이 .git 디렉토리 안의 **메타데이터만 있는 Git 저장소**를 의미한다.

즉, **Bare repository**는 코드를 직접 수정하거나 열어보는 용도가 아닌거다.

보통은 push/pull을 할 때 기점이 될 역할로 사용한다고 한다.

우리가 SPM을 설치할 때 `https://github.com/repository.git`과 같은 형태로 SPM을 받아오는데, 이게 해당 레포지토리를 **Bare repository**형태로 가져온다는 의미였다.

#### **이러니까 문제지...**
정리하자면 사진에 보이는 설정은 **Bare repository**를 안전한 경로에 등록할때만 사용될 수 있도록 해주는 명령어인데

이 설정이 되려 SPM을 가져올 때 현재의 repo를 안전하지 않다고 판단해서 SPM도 설치가 안되고 문제가 발생한것이다.

`~/.gitconfig`{: .filepath}에 있는 아래 두 줄을 지워주면 문제가 해결된다.

```
[safe]
	bareRepository = explicit
```


## 🏁 마무리

![](assets/img/post/2025/04_07_SPM_설치_성공.png){: w="600"}_SPM 설치 성공!!! 🥹_

드디어 성공했다!!

처음부터 봤던 레퍼런스가 사실 정답이었는데,,,,

소스트리 단어만 보고 그냥 지나가면서 문제 해결이 더 길어졌었다😅

핵심은 git 설정이었고, 잘 해결해서 지금 SPM 설치가 매우 잘된다

이번 기회를 통해 깃 설정 파일이 로컬에도 따로 있다는것을 알 수 있었고

SPM을 추가할 때 git 주소 끝에 .git을 써주는 이유도 알게 됐다.

다들 헤매지 말고 꼭 해당 설정이 되어있는지 확인하면 좋겠다👍🏻

> Git GUI는 역시 `sourcetree`보단 `Fork`!!

## 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [Xcode "Package Resolution Failed. Couldn't update repository submodules:" - Stack Overflow](https://stackoverflow.com/questions/73767169/xcode-package-resolution-failed-couldnt-update-repository-submodules)
>- [Xcode 16 & Package load failure \| Apple Developer Forums](https://developer.apple.com/forums/thread/763897)
>- [swift - Package resolution failed. Couldn't get the list of tags - Stack Overflow](https://stackoverflow.com/questions/69555397/package-resolution-failed-couldnt-get-the-list-of-tags)