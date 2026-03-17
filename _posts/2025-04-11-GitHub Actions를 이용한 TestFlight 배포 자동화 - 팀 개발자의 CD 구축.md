---
title: "[Swift] GitHub Actions를 이용한 TestFlight 배포 자동화 - 팀 개발자의 CD 구축"
date: 2025-04-11 15:59 +0900
categories:
  - Mobile
  - iOS
tags:
  - DevOps
  - PopPool
  - Project
---

팝풀 프로젝트에서 첫 배포를 마친 뒤, 본격적으로 차기 버전을 준비하면서 **배포 과정을 자동화하고 싶다는 생각이 들었다.**

생각의 시작은 단순했다. 빌드하고, 추출하고, 올리고... 배포까지는 수작업이 너무 많았다.

특히 **애자일하게 자주 빌드하고 테스트해야 하는 상황**에서는 이런 반복 작업이 흐름을 끊기게 만들고, 팀의 개발 시간도 계속 뺏길 수 있다 생각했다.

_**(✨ 개발자의 시간은 소중하니까 ✨)**_

그래서 CD를 도입해야 겠다 하며 야심차게 시작했는데...

배포한 계정의 소유주가 아닌 팀 개발자가 CD를 야무지게 구현할만한 방법들이 거의 보이지 않았다.

그러다 우연히 [TestFlight 자동화](https://sujinnaljin.medium.com/ci-cd-github-actions-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-testflight-%EC%97%85%EB%A1%9C%EB%93%9C-%EC%9E%90%EB%8F%99%ED%99%94-8ecdbeb227a3) 글을 보게 되었고, 

"외부 개발자여도 App Store Connect API와 GitHub Actions를 이용하여 CD 구축이 가능하겠다!"

는 생각에 도전하게 되었다.

## 🧑🏻‍💻 팀 개발자가 CD를 구현하기 위해 필요한 조건

내가 팀 개발자로서 CD 환경을 구축하기 위해 준비한건 다음과 같다.

- **[App Store Connect API Key 3종](https://developer.apple.com/help/app-store-connect/get-started/app-store-connect-api/):** Issuer ID, API Key ID, Private key
- **[인증서](https://hsdev.tistory.com/entry/iOS-%EC%95%B1-%EB%B0%B0%ED%8F%AC-1-2-Certificates-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0)와 [프로비저닝 프로파일](https://hsdev.tistory.com/entry/iOS-%EC%95%B1-%EB%B0%B0%ED%8F%AC-3-Provisioning-Profile-%ED%94%84%EB%A1%9C%EB%B9%84%EC%A0%80%EB%8B%9D-%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0):** 기본 파일 2종, gpg로 암화한 파일 2종

일반적으로 앱스토어 개발자 프로그램 정책에 따라 개인 계정은 팀원을 초대하고 권한을 아무리 끌어올리더라도 못하는 것들이 좀 많다.

그 중 **API Key 발급과 인증서, 프로비저닝**이 그에 속했다. 

또한 fastlane처럼 모든 걸 알아서 해주는 도구를 쓰기 위해서는 애플 아이디를 아예 등록하는 경우도 있는데, 이런것도 팀 개발자로 불가능의 영역이었다.

그래서 애플 계정 요구를 하지 않는 범위에서 추가 파일을 요청한뒤 **GitHub Actions + App Store Connect API**를 이용해서 만들게 됐다.

_**(이쯤되니 계정 소유주 납치가 시급함 🤣)**_

## 🔁 CD 구축 과정

CD의 흐름은 아래처럼 간단히 정리할 수 있다. 

1. release/* 브랜치에 PR이 열릴 때 트리거
2. 빌드 번호 자동 설정
3. 빌드 세팅
4. 앱 아카이브 & ipa 추출
5. TestFlight 업로드

전반적인 워크플로우 구축은 [TestFlight 자동화](https://sujinnaljin.medium.com/ci-cd-github-actions-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-testflight-%EC%97%85%EB%A1%9C%EB%93%9C-%EC%9E%90%EB%8F%99%ED%99%94-8ecdbeb227a3) 자료를 많이 참고하며 만들었다. 

그리고 레퍼런스를 기반해서 팀에 맞게 개선한 코드는 [여기](https://github.com/PopPool/iOS/blob/develop/.github/workflows/deploy_on_release.yml)서 확인이 가능하다.

이 중 내가 필요해서 넣었던 추가 Step은 xcconfig 생성과 빌드 버전 세팅이었는데,,,

빌드 버전 세팅에서 매우 많은 문제를 겪었다. 💢

## 🚨 번들 버전 자동화 미적용

처음에 번들 버전에 문제가 있다는건 워크플로우를 다 구현했었을 때 쯤이었다.

번들 버전을 크게 신경쓰지 않고 있었는데, TestFlight에 업로드 하려니까 버전이 낮다는 문제로 워크플로우가 실패한 것이었다.

그래서 버전을 수정하고 있었는데, 이것도 자동화가 되지 않을까? 

하고 워크플로우에 자동 버전 범핑을 추가하려고 시도했다.

![](assets/img/post/2025/04_11_난리난_커밋.png){: width="600"}_**(그렇게 2일이 지나고 🫠)**_

진짜 자동 버저닝에 대해 열심히 알아보고 이것저것 적용해봤는데 하나같이 버전이 오르지가 않았다.

대부분의 레퍼런스에서 자동 번들 버전 범핑을 위해 

`Info.plist`의 `CFBundleVersion`을 수정하는 Run Script를 구성하거나, 

워크플로우에서 수정하는 방식이었다.

나도 처음에는 이렇게 적용을 했는데,,,,,

몇번을 해도 **추출된 ipa에서는 Bundle Version이 `Info.plist`의 버전을 따라가지 않는 것**이었다.

### 문제의 원인

나도 이 문제를 해결하기 위해 열심히 디버깅을 하던 중 알게된 핵심은

1. **프로젝트 설정의 번들 버전은 `Info.plist`의 `CFBundleVersion`과 같지 않다는 것**
2. **추출된 ipa는 프로젝트 설정의 번들 버전을 따른다는 것**

`Info.plist`는 변수를 넣어줄 수 있는데 이 때 `CFBundleVersion`에 기본으로 `$(CURRENT_PROJECT_VERSION)`을 할당해주어 둘을 같다고 인식한 것이었다.

그래서 아무리 레퍼런스의 방식을 따라하더라도 `CFBundleVersion`을 바꾸면 그냥 변수가 특정 상수로 바뀔 뿐,

원하던 빌드 버전(빌드 설정의 `CURRENT_PROJECT_VERSION`)이 오르는 결과를 내진 않던 것이었다.

_**(알고보니 레퍼런스들이 거의 5~10년 지났던 거기도 했음 🥲)**_

특히 [이 스택오버플로우의 질문](https://stackoverflow.com/questions/69261679/current-project-version-in-build-settings-not-updating-as-expected/73365699)이 내 문제 상황과 가장 유사했었다.

### 빌드 자동 범핑 완성하기

처음에는 Info.plist에 값을 바꾸는건 했으니 빌드 설정을 맞춰 바꾸면 되겠지 했었는데,

좀만 고민해보니까 이거 안좋은 방식 같았다

`.pbxproj` 파일이 빌드할 때 마다 `Info.plist`를 기반으로 수정된다고...?

파일의 중요도가 역전되는 현상이었다.

그리고 하더라도 문제는 매 빌드마다 번호가 계속 바뀌어서 Git의 Change log에도 뜨는게 거슬렸다.

#### **최종적으로**

```yaml
- name: "#️⃣ Set Build Number"
  run: |
    BUILD_NUMBER=$(TZ=Asia/Seoul date +%y%m%d.%H%M)
    cd Poppool
    agvtool new-version -all "$BUILD_NUMBER"
```

GitHub Action의 추가 Step을 만들어 빌드 번호를 설정해주는 방식을 적용했다.

이 때 빌드 번호를 설정해주기 위해 [agvtools](https://developer.apple.com/library/archive/qa/qa1827/_index.html)을 사용해주었다.

빌드번호 규칙은 항상 우상향하도록 하기 위해 `YYMMDD.HHmm` 형태를 적용했다.

_**(오히려 좀 더 직관적이기도?)**_

이 방식을 사용하면 프로젝트의 설정이 바뀌지 않다보니 change log도 뜨지 않고,

마켓팅 버전은 직접 관리할 수 있으니까 좋았던것 같다.

## 🏁 마치며

이 글을 통해 계정 소유주가 아닌 팀 개발자가 CD를 구축하는데 필요한 정보와 내가 겪은 문제들을 정리해봤다.

**이번 기회를 통해 배포에 대해 좀 더 자세히 알 수 있는 기회가 됐었고,** 

**자동화 구축에 대해서도 한번 더 경험해보는 계기가 됐던것 같다.**

중간에 빌드 자동 범핑이 생각처럼 되지 않아 수동 빌드 넘버링을 그냥 쓸까 하며 반포기 했었는데,

막상 있으니까 **TestFlight 배포를 위해서 나는 오직 `release/*` 브랜치에 PR**만 날려주면 돼서 너무 좋다 지이이인짜👍🏻

무엇보다 이번에 자동화를 통해 얻은 가장 큰 결과는 **더 집중하기 좋은 개발환경** 않을까 싶다.

![](assets/img/post/2025/04_11_테스트플라이트_웹훅.png){: width="400"}_🔔 자동화의 완성은 역시 디스코드 웹훅 🔔_

## 🔗 레퍼런스
> **해당 자료를 찾아보며 도움이 됐던 링크**
>- [TestFlight 자동화](https://sujinnaljin.medium.com/ci-cd-github-actions-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-testflight-%EC%97%85%EB%A1%9C%EB%93%9C-%EC%9E%90%EB%8F%99%ED%99%94-8ecdbeb227a3)
>- [iOS 앱 개발권한 부여](https://cheolheelee.tistory.com/415)
>- [Xcode 빌드 번호 자동 증가](https://medium.com/@mateuszsiatrak/automating-build-number-increments-in-xcode-with-custom-format-a-practical-guide-bcc90a19f716)