---
title: "[Swift] Swiftlint 자동 수정 자동화 - 팝풀 CI 적용기"
date: 2025-04-03 11:22 +0900
categories:
  - Mobile
  - iOS
tags:
  - Swift
  - Project
  - PopPool
  - DevOps
---
처음 팀에 들어오고 가장 먼저 맡았던 일은 코드 리뷰였다.

당시 팀은 전반적인 **리팩토링**을 원하고 있었기에, 프로젝트가 어떤 구조로 되어 있는지, 어떤 부분을 개선해야 할지 꼼꼼히 살펴보려 노력했다.

그런데…

**후...** 코드 스타일이 이렇게 중요하다는 걸 실감한 건 그 리뷰 과정을 거치면서였다.

클래스는 잘 나눠져 있고 기능도 나쁘지 않았지만, 들여쓰기, 공백, 주석, 줄 바꿈...

**사소한 스타일이 제각각**이었고, 그 작은 차이들이 코드 가독성을 떨어뜨리고 리뷰 시간도 길어지게 만들고 있었다.

그래서 팀원에게 **SwiftLint 도입**을 가장 먼저 제안했다.

## 🎨 SwiftLint 도입

처음 린트를 도입하는 건 어렵지 않았다.

팀원들도 코드 스타일 문제에 대해 공감하고 있었고, 도입 자체에 거부감은 없었다.

그래서 바로 `.swiftlint.yml`을 구성하고 빌드를 돌려봤는데…

![린트 도입 직후](assets/img/post/2025/04_03_린트_도입_직후.png){: w="200"}_린트 도입 직후_

> 🔴 Error: **42개**
> 🟡 Warning: **4000개** 근접

상당히 충격적인 숫자였다.

대부분은 들여쓰기, 불필요한 공백, 줄 바꿈 같은 **스타일 위주의 문제**였다.

그래서 1차 코드 스타일 회의를 통해 SwiftLint 룰을 커스터마이징했다.

회의를 거친 다음 아래와 같은 규칙으로 수정했다.

```yml
# 기본 활성화된 룰 중에 비활성화할 룰을 지정
disabled_rules:
  - redundant_optional_initialization

# 기본(default) 룰이 아닌 룰들을 활성화
opt_in_rules:
  - sorted_imports
  - direct_return
  - file_header
  - weak_delegate

# 기본 활성화된 룰 중에 조건을 변경할 룰
file_length: 400

function_body_length:
  warning: 100
  error: 150

cyclomatic_complexity:
  warning: 15
  error: 30
```

이후 다시 돌려본 결과는…

![1차 린트 수정 후](assets/img/post/2025/04_03_린트_수정_후.png){: w="200"}_1차 린트 수정 후_

> 🔴 Error: **33개**
> 🟡 Warning: **4000개 돌파**

줄어들긴 했지만, 여전히 **경고는 너무 많았다**.

이걸 전부 수작업으로 고친다는 건 말이 안 됐다.

## 🔄 SwiftLint Autocorrect

다행히 SwiftLint는 자동 수정 기능을 지원한다.

명령어는 `swiftlint --fix`로 간단했다

하지만 문제는 그걸 **매번 개발자가 수동으로 해야 한다는 것**이었다.
- 매번 `--fix`를 기억해야 하고
- 고친 뒤 커밋도 직접 해야 하며
- 자칫하면 린트 수정 자체가 누락되기도 쉽다

이런 반복 작업은 결국 **누락되거나 무시**되기 마련이다.

그래서 생각했다

> "아예 자동화하면 안 될까?"

## ⚙️ GitHub Actions로 자동 수정 자동화

이전에 CI를 써봤을 때 린트 검사를 했던 흐름을 참고해서, `autocorrect`를 자동으로 돌리는 워크플로를 만들었다.

```yaml
name: 🤖 Autocorrect Workflow
runs-on: macos-15  # 최신 macOS 15 환경에서 실행
if: github.actor != 'github-actions[bot]'  # Actions 봇 커밋은 무시

steps:
  # 내용 생략
  - name: 🎨 Run SwiftLint Autocorrect  # SwiftLint 자동 수정 실행
    run: swiftlint --fix

  - name: 🚀 Commit and Push Changes  # 변경 사항 자동 커밋 및 푸시
    run: |
      git config user.name "github-actions[bot]"
      git config user.email "github-actions[bot]@users.noreply.github.com"

      # 내용 생략

      if [ -n "$(git status --porcelain)" ]; then
        git add .
        git commit -m "style/#${ISSUE_NUMBER}: Apply SwiftLint autocorrect"
        git push --set-upstream origin "${GITHUB_HEAD_REF}"
      fi
```

이런 저런 것들을 많이 생각했던것 같다. 

자동 수정은 꼭 커밋으로 남기게 만들고 싶었고, 그 커밋이 컨벤션도 맞게 했으면 좋겠어서 좀 고생했던것 같다.

최종적으로 자동 커밋 메시지는 `"style/#이슈번호: Apply SwiftLint autocorrect"` 형식으로 남기도록 했다. 

덕분에 이제는 PR이 열리면 알아서 스타일이 정리되고, 불필요한 추가 작업을 생략하게 할 수 있었다.

##  🚀 CI 도입

자동 수정을 셋업하면서, 자연스럽게 **CI 파이프라인**도 도입하게 됐다.

코드를 보던 중, 꽤 오래된 PR이 몇 달째 열려 있는 걸 봤다.

그 상태로 방치된 브랜치가 실수로 병합된다면?

→ **빌드 에러**, **기능 중단**, **디버깅 지옥**으로 이어질 수 있다.

그래서 PR마다 **기본적인 빌드 테스트와 린트 검사가 자동으로 실행**되도록 CI를 구성했다.


``` yaml
name: 🏗️ Build Workflow
runs-on: macos-15  # 최신 macOS 15 환경에서 실행
if: github.actor != 'github-actions[bot]'  # Actions 봇 커밋은 무시

steps:
  # 내용 생략
  - name: ⚙️ Generate xcconfig
    run: # 내용 생략

  # 내용 생략      
  - name: 🎨 Run SwiftLint  # SwiftLint 코드 스타일 검사 실행
    run: swiftlint
	
  # 내용 생략
  - name: 🏗️ Build the project  # 자동 검지된 Scheme과 Simulator로 빌드 수행
    run: # 내용 생략
```

CI도 마찬가지로 Scheme이 변경될 수도 있기에 유지보수성을 고려해서 설계했던것 같다.

CI 도입을 통해 팀원들이 “내 코드가 문제 없이 작동한다”는 **확신**을 가질 수 있게 했다.

코드 충돌도 더 빨리 파악할 수 있어서 merge 과정도 훨씬 원활해질것이라 보인다.

## 🔧 추가 작업: Secrets.swift → xcconfig 전환

아무래도 Secrets이 없으면 빌드 파일에 기록은 있지만 GitHub Actions 환경 기준으로는 파일이 없으니 에러가 날것이라 생각했다.

그래서 CI 빌드 세팅을 위해 기존의 `Secrets.swift`를 `debug.xcconfig`/`release.xcconfig` 구조로 전환했다.

또한 CI에선 GitHub의 Repository secret을 사용해 임시 `.xcconfig` 파일을 생성해 빌드 시 필요한 키를 주입하도록 구성했다.

덕분에 **민감 정보가 레포에 남지 않으면서도 CI 빌드가 가능**해졌다.

##  🏁 마무리
자동 수정을 적용한 후 warning수를 4000개에서 무려 100개까지 줄여냈다.

![린트 자동 수정 후](assets/img/post/2025/04_03_CI_돌아간다.png){: w="400"}_감격스러운 순간👍🏻_

![린트 자동 수정 후](assets/img/post/2025/04_03_린트_자동_수정_후.png){: w="100"}_감격스러운 순간✌🏻_

스타일 통일, 반복 작업 제거, 빌드 안정성 확보.

이 모든 걸 단지 **린트 자동화와 CI 구성**만으로 이뤄냈다는 건 꽤 인상 깊은 결과였다.

특히 SwiftLint의 `autocorrect`를 GitHub Actions로 자동화한 레퍼런스는 생각보다 흔하지 않아 조금 고생했지만, 도입 난이도 대비 만족도는 아주 높았다.

[팝풀 iOS - PR #98: CI & Autocorrect 적용](https://github.com/PopPool/iOS/pull/98)

해당 링크는 팝풀의 Swiftlint 적용과 관련된 PR이다. 참고가 되면 좋겠다.

#### **04/22 소소한 근황**
![](assets/img/post/2025/04_03_AutoCorrect_자동화_성과.png){: width=200}_열일하는 GitHub Actions bot_

> 종종 팀원보다 많은 커밋을 보낼 정도로 자동화의 기여도가 높아서 기분이 좋습니다👍🏻

## 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [SwiftLint GitHub](https://github.com/realm/SwiftLint)
>- [Fastlane + Github Actions로 CI/CD 파이프라인 구축하기 (6편 - Fastlane, github actions로 tuist project의 CI/CD 구현)](https://jazz-the-it.tistory.com/85)
>- [[CI/CD] GitHub Actions를 이용한 테스트 자동화](https://ho8487.tistory.com/126)