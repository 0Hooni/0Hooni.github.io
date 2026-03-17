---
title: "[Xcode] AppProject 및 속성"
date: 2022-10-21 21:15 +0900
categories:
  - Mobile
  - iOS
tags:
  - Xcode
---
## **Project**

>  An Xcode project is a repository for all the files, resources, and information required to build one or more software products. A project contains all the elements used to build your products and maintains the relationships between those elements. It contains one or more targets, which specify how to build products. A project defines default build settings for all the targets in the project (each target can also specify its own build settings, which override the project build settings).  
>   
> 출처 : 애플공식문서

-   Project란 하나 이상의 소프트웨어 제품을 구축하는데 필요한 모든 파일, 리소스 및 정보의 저장소
-   Project는 하나 이상의 Target을 포합하고 있으며 모든 Target에 대한 기본 빌드 설정을 정의

## **AppProject 속성**

### 1\. Info

![](assets/img/post/2022/10_21_info.png)

-   Deployment Target : App이 지원하는 최소 iOS 버전을 설정
-   Configurations : 개발버전, 릴리즈버전, 무료버전, 등등 다양한 환경에서 테스트를 하고싶을때 Configurations에 추가하여 테스트할 수 있음
-   Localizations : 지원하는 언어를 설정

### 2\. Build Setting

![](assets/img/post/2022/10_21_build_setting.png)
_Build Setting 최상단에서 보이는 전문, 밑으로 더 많은 Setting이 존재한다_

-   Build Setting을 저장하고 있는 파일이 project.pbxproj이며 아주 민감한 값들을 지니고 있음
-   기본값이 아닌 Build Setting은 Customized 필터 옵션에서 확인
-   현재 적용되어 있는 Build Setting이 기본값을 쓰고 있는지, 프로젝트에 적용된 값을 쓰고 있는지, Target별로 Override를 한것인지 등을 확인 가능
-   **불필요한 Build Setting 변경을 최소화 하자**

### 3\. Package Dependencies

![](assets/img/post/2022/10_21_package_dependencies.png)

-   Swift Package Manager이며 외부의 Swift package를 추가해줄 수 있다.