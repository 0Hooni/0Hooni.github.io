---
title: "[Xcode] Target 및 속성"
date: 2022-10-21 00:04:00 +0900
categories:
  - 🍎 Dev
  - Xcode
tags:
  - Xcode
---
## **Target**

> A target specifies a product to build and contains the instructions for building the product from a set of files in a project or workspace. A target defines a single product; it organizes the inputs into the build system—the source files and instructions for processing those source files—required to build that product. Projects can contain one or more targets, each of which produces one product.  
>   
> 출처 : 애플 공식문서

-   Target은 Build하고자 하는 하나의 Product를 지정하고, Product를 빌딩하는데 필요한 다양한 설정을 포함하고 있다.

## **Target의 속성**

### 1\. General

-   Support Destinations : 지원 기기를 설정
-   Minimum Deployments : 최소 지원하는 iOS 버전을 설정
-   Identity
    -   App category : 앱의 카테고리를 설정
    -   Display Name : iOS 홈 화면에 표시되는 번들의 이름
    -   Bundle Identifier : Apple 생태계에서 앱을 고유하게 식별할 수 있는 ID
    -   Version : 사용자가 보게될 나의 앱 버전
-   Deployment Info
    -   iPhone Orientation : 앱 실행 화면의 회전 가능한 방향을 설정
    -   iPad Orientation : 아이패드에서 회전 가능한 방향을 설정
    -   Status Bar Style : 상태바의 스타일 설정
-   Supported Intents : App이 SifiKit와 상호작용을 하려면 Intents가 필요하다. 이럴 경우 Supported Intents에 추가해줘야 한다.
-   Frameworks, Libraries, and Embedded Content : 앱에서 사용하는 라이브러리를 embedded 시킬 지, 시키지 않을지에 대한 설정. 시키는 경우에는 앱에 library가 포함된 채로 배포되면, 반대는 포함되지 않으며 library가 참조된다.
-   Development Assets : Product에는 넣지 않고 Development 단계에서만 쓰고 싶은 Assets들을 넣을 수 있는 곳이다.

### 2\. Signing & Capability

-   인증 관련 항목들을 관리한다. Capability를 통해 여러 권한을 설정할 수 있다.

### 3\. Resource Tags

-   필요시에 다운로드하는 앱 컨텐츠를 의미한다. 앱의 크기를 줄이고, 빠른 다운로드를 위해 사용된다.

### 4\. Info

-   Custom iOS Target Properties
    -   Bundle name : 사용자가 볼 수 있는 번들의 짧은 형식의 이름
    -   Bundle identifier : 번들의 고유 식별자
    -   InfoDictionary version : info.plist 구조의 현재 버전
    -   Main storyboard file base name : 앱의 스토리보드 리소스 파일 이름
    -   Bundle version : 번들의 버전
    -   Launch screen interface file base name : 아마 앱 초기실행시 나타나는 초기화면의 파일 이름을 지정하는 것으로 보임
    -   Executable file : 번들의 실행 파일 이름
    -   Application requires iPhone environment : 앱이 iOS에서 실행되어야 하는지 여부를 나타내는 Bool 값
    -   Supported interface orientations (iPhone) : 아이폰 앱에서 지원하는 인터페이스 방향
    -   Application supports indirect input events : 앱이 일반적으로 간접 입력 메커니즘을 지원함을 나타내는 Bool 값
    -   Application Scene Manifest : 앱의 Scene 기반 Lifecycle 지원에 대한 정보
    -   Bundle OS Type code : 4글자로 이루어진 번들 타입 - 앱: APPL; 프레임워크: FMWK; 번들: BNDL
    -   Status bar style : 상태바의 스타일을 지정
    -   Development localization : 아마도 앱의 언어와 관련된 부분인 것으로 추정
    -   Supported interface orientations (iPad) : iPad 앱에서 지원하는 인터페이스 방향
    -   Bundle version string (short) : 번들의 릴리즈 또는 버전 번호
-   Document Types : 앱에서 만들고 편집할 수 있는 문서 유형을 지정하고 iOS 또는 Mac OS에서 해당 문서 유형에 대한 표시되는 사용자 정의 아이콘을 제공한다.
-   Exported Type Identifiers, Imported Type Identifiers : 앱에서 내보내거나 가져올 수 있는 모든 파일 유형에 대해 내보내고 가져온 UTI를 추가. 일반적으로 앱에서 고유한 문서 유형과 달리 UTI는 일반 텍스트 또는 .png. 예를 들어 UTI는 앱간에 클립보드에 복사 및 붙여넣기를 지원한다.
-   URL Types : URL Types 설정을 사용하면 사용자 지정 프로토콜을 사용하여 다른 앱과 데이터를 교환하기 위한 사용자 지정 스키마를 지정 가능하다.

### 5\. Build Setting

-   대상 제품을 빌드하는 데 필요한 다양한 설정, 정보를 제공한다.

### 6\. Build Phases

-   빌드 시 수행하는 작업들을 설정하는 곳이다. 소스 코드 컴파일(Compile sources), PRODUCT에 리소스 복사(Copy bundle resources) 등이 있다.

### 7\. Build Rules

-   빌드 프로세스 중에 특정 유형의 파일을 처리할 때 빌드 시스템이 사용하는 규칙을 사용자 정의한다.