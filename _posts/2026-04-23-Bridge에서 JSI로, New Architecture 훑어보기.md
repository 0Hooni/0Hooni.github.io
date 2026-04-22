---
title: "[ReactNative] Bridge에서 JSI로, New Architecture 훑어보기"
date: 2026-04-23 00:30 +0900
description: "Expo로 RN을 처음 시작한 개발자 관점에서 Bridge, JSI, New Architecture가 어떤 문제를 풀기 위해 등장했고 어떻게 동작하는지 정리합니다."
categories:
  - Mobile
  - ReactNative
tags:
  - Architecture
  - Bridge
  - JSI
  - Fabric
  - Expo
---

> 이 글은 **Expo로 React Native를 처음 시작한 개발자**를 대상으로 합니다.
> Bridge, JSI, New Architecture라는 이름만 들어봤거나, 공식 문서를 훑어봤지만 큰 그림이 잘 안 잡히는 분에게 도움이 됩니다.
{: .prompt-info }

## 🤔 왜 이 글을 쓰게 됐나

Expo SDK 52~53 정도에 React Native를 처음 만졌습니다.

그런데 공식 문서, 블로그, 라이브러리 릴리스 노트 곳곳에서 **Bridge, JSI, Fabric, TurboModules** 같은 낯선 단어가 계속 튀어나왔습니다. "New Architecture가 기본 활성화됐다"는 얘기도 자주 나왔고요.

지금 만들고 있는 앱은 이미 **New Architecture 위에서 돌아가는데**, 정작 그게 뭔지 모르고 쓰고 있다는 게 찜찜했습니다.

그래서 **"우리가 서 있는 이 구조의 과거는 어땠고, 무엇을 바꿔서 지금에 이르렀는가"**를 한번 정리했습니다.

## 🌉 RN이 풀어야 했던 근본 문제

React Native의 핵심 아이디어는 단순합니다.

> 한 번 짠 JS/React 코드로 iOS와 Android를 동시에 돌리자.

그런데 여기서 바로 문제가 생깁니다. JS 엔진은 혼자서 **버튼을 그릴 수 없습니다.**

화면에 보이는 `<View>`, `<Text>`는 실제로는
- iOS에서는 `UIView`
- Android에서는 `android.view.View`

같은 **네이티브 객체**로 존재해야 합니다. 즉, **JS가 네이티브에게 "버튼 하나 만들어줘"라고 말을 걸어야 하는 상황**이 됩니다.

![RN 일반 구조](/assets/img/post/2026/rn-new-architecture/rn-general.webp)
_JS 세계와 Native 세계는 기본적으로 분리되어 있고, 무언가로 연결해야 한다_

그 연결 방식이 초기에는 **Bridge**였고, 지금은 **JSI**입니다.

## 🏗️ Old Architecture — Bridge의 세계

### 3개의 스레드

Bridge 시대의 RN은 **3개 스레드**로 돌아갑니다.

| 스레드 | 역할 |
|--------|------|
| **JS 스레드** | React 컴포넌트 실행, 상태 관리 |
| **Shadow 스레드** | Yoga 엔진으로 레이아웃 계산 (flex, width 등) |
| **Native(Main) 스레드** | 실제 네이티브 뷰 그리기, 터치 이벤트 처리 |

이 3개는 서로 다른 스레드라서 **직접 함수 호출을 할 수 없습니다.** 그래서 사이에 Bridge라는 **메시지 큐**를 둡니다.

![Bridge 아키텍처](/assets/img/post/2026/rn-new-architecture/bridge-architecture.webp)
_JS와 Native 사이에 Bridge(JSON 기반 메시지 큐)가 끼어 있는 구조_

### Bridge 통신의 3가지 특징

Bridge를 통해 오가는 메시지는 다음 세 가지 성질을 가집니다.

1. **비동기(Async)** — 보낸 즉시 응답을 받을 수 없습니다.
2. **직렬화(Serialized)** — 모든 메시지는 JSON 문자열로 변환됩니다.
3. **일괄 전송(Batched)** — 개별로 보내지 않고 묶어서 전달합니다.

즉, 이런 흐름입니다.

```
[JS] "버튼 만들어줘"
   → JSON 변환
   → Bridge 큐
   → [Native] JSON 파싱
   → UIView 실제 생성
```

### 왜 이런 설계였을까

2015년 당시 설계 전제는 다음과 같았습니다.

- JS 엔진(JavaScriptCore)과 네이티브는 **완전히 분리된 세계**였다
- 서로 메모리나 객체를 공유할 수 없으니, **문자열(JSON)**이라는 공통 포맷으로 우회한다
- 비동기로 보내면 UI 스레드가 막히지 않아 앱이 덜 버벅거릴 것이라고 기대했다

> **원인을 혼동하지 않기**
>
> JSON 때문에 Bridge를 쓴 것이 아니라, Bridge라는 구조적 제약 때문에 JSON을 써야만 했습니다.
{: .prompt-tip }

## 😵 Bridge가 남긴 5가지 고통

이 구조는 꽤 오래 버텼지만, 결국 다섯 가지 한계가 드러났습니다.

### 1. 동기 호출이 불가능

JS가 네이티브에 무언가를 묻고 **즉시 답을 받아야** 할 때가 있습니다. 예를 들어 화면 크기 같은 값입니다.

```js
const { width, height } = Dimensions.get('window');
```

Bridge는 비동기라서, 이런 값을 동기적으로 쓰려면 앱 시작 시점에 값을 **JS 쪽에 미리 복사**해 두는 꼼수가 필요했습니다.

### 2. 애니메이션과 제스처 병목

60fps를 유지하려면 한 프레임을 **16ms 안에** 그려야 합니다.

사용자가 뷰를 드래그할 때의 흐름을 따라가 보면,

```
[Native] 터치 발생 → JSON → Bridge → [JS] 새 위치 계산 → JSON → Bridge → [Native] 반영
```

매 프레임마다 이 왕복을 돌면 Bridge 큐가 밀리면서 **끊김**이 생깁니다.

### 3. JSON 직렬화 비용

큰 데이터, 예를 들어 이미지 바이트 배열 같은 것을 보내려면 JSON 문자열로 변환하는 비용 자체가 무겁습니다.

### 4. 모든 모듈을 앱 시작 시 일괄 초기화

Bridge는 JS가 호출할 **네이티브 모듈 목록을 처음부터 전부 알고** 있어야 합니다. 그래야 JSON 메시지에 적힌 모듈 이름을 보고 어디로 전달할지 결정할 수 있습니다.

결과적으로,

```
앱 시작 ─┬─ CameraModule 인스턴스 생성 (안 써도)
        ├─ LocationModule 인스턴스 생성
        ├─ BluetoothModule 인스턴스 생성
        ├─ ... (수십 개)
        └─ 이제 JS 실행 시작
```

**앱 콜드 스타트가 느려지고**, 안 쓰는 모듈까지 메모리에 상주합니다.

### 5. 타입 안전성이 없다

JS는 네이티브를 **문자열 이름으로** 호출합니다.

```js
NativeModules.Camera.takePhoto({ quality: 'high' });
```

`"Camera"`, `"takePhoto"` 둘 다 문자열이라 **오타가 있어도 런타임에야** 터집니다. 네이티브 쪽에서 메서드 이름이 바뀌어도 컴파일 타임에 잡을 방법이 없습니다.

## 🔌 JSI — 다리 대신 직접 연결

이 다섯 가지 고통을 뿌리부터 뽑기 위해 등장한 것이 **JSI(JavaScript Interface)**입니다.

> JSI는 JS 엔진과 C++ 세계를 **직접 연결**하는 얇은 C++ 인터페이스 계층입니다.

"Bridge를 없앴다"기보다는 **Bridge를 대체할 더 낮은 레벨의 통로**를 새로 깐 것입니다.

### 잠깐, JS 엔진이 C++라고?

JSI를 이해하려면 한 가지 사실을 먼저 받아들여야 합니다.

> JavaScript라는 언어는 **C++로 작성된 프로그램(엔진)**에 의해 실행됩니다.

| 엔진 | 만든 곳 | 구현 언어 |
|------|---------|-----------|
| **Hermes** | Meta (RN 기본) | C++ |
| **V8** | Google (Chrome, Node) | C++ |
| **JavaScriptCore** | Apple (Safari) | C++ |

Python이 CPython이라는 **C 프로그램**에 의해 실행되는 것과 같은 구조입니다. JS 값 `{ name: "hoon" }`도 엔진 내부에서는 **C++ 객체**로 저장됩니다.

그래서 C++ 수준에서 JS 엔진 내부에 접근하면 **JSON 변환 없이도 JS 값을 직접 읽고 쓸** 수 있습니다. 이게 JSI의 핵심 마법입니다.

### HostObject — JS처럼 생긴 C++ 객체

JSI에서 가장 중요한 개념은 **HostObject**입니다.

```js
// JS 코드
const camera = global.CameraModule;
camera.takePhoto();
// JS 함수 호출처럼 보이지만, 실제로는 C++ 함수가 즉시 실행됨
```

**Bridge 방식과 JSI 방식 비교**:

| 항목 | Bridge | JSI |
|------|--------|-----|
| 통신 방식 | 메시지 큐 | 직접 함수 호출 |
| 직렬화 | JSON 변환 | 없음 (객체 참조 공유) |
| 동기/비동기 | 비동기만 | 둘 다 가능 |
| 타입 기반 | 문자열 | C++ 타입 |

### Shadow 스레드는 어떻게 바뀌었나

Shadow 스레드 자체가 사라진 것은 아닙니다. 달라진 것은 **그 안에서 다루는 Shadow Tree가 C++로 재구현**되어, Yoga 엔진과 함께 **JSI를 통해 JS에서 동기적으로 접근 가능**해졌다는 점입니다.

Bridge 왕복이 사라지면서, 스레드는 같은 수만큼 남아 있어도 **스레드 간 통신 비용이 크게 줄어들었습니다.**

```
[Old]
JS 스레드 ─Bridge─ Shadow 스레드 ─Bridge─ UI 스레드
       (비동기 JSON 왕복)

[New]
JS 스레드 ═JSI═ Shadow 스레드 (C++ Shadow Tree + Yoga) ──→ UI 스레드
       (동기 C++ 호출)
```

## 🏛️ New Architecture의 3대 기둥

JSI는 **하위 레이어**일 뿐이고, 그 위에 실제 기능을 담당하는 세 가지 구성 요소가 쌓여 있습니다.

![Fabric 아키텍처](/assets/img/post/2026/rn-new-architecture/fabric-architecture.webp)
_JSI 위에 Fabric, TurboModules가 올라가고, C++ 코어가 양 플랫폼에 공유되는 구조_

### 1. TurboModules — Lazy 네이티브 모듈

TurboModules는 기존 **NativeModules를 대체**합니다. 동작은 비슷해 보이지만 실행 방식이 다릅니다.

```
[Old]
앱 시작 → 모든 네이티브 모듈 일괄 등록/초기화
       → 첫 호출부터 Bridge 대기
       → 매 호출마다 JSON 직렬화

[New]
앱 시작 → JSI만 준비, 모듈은 아직 생성 안 함
       → JS가 모듈을 처음 호출할 때 C++ 객체 생성 (Lazy)
       → 이후 호출은 C++ 함수 직접 실행 (동기도 가능)
```

핵심 이점은 다음과 같습니다.

- **콜드 스타트 단축**: 시작 시 모듈 초기화를 건너뜁니다.
- **메모리 절감**: 안 쓰는 모듈은 메모리에 올라가지 않습니다.
- **동기 호출 가능**: `Dimensions.get()` 같은 API가 꼼수 없이 동작합니다.

### 2. Fabric — C++로 내려간 렌더러

Fabric은 기존 **UIManager + Shadow 스레드**를 대체하는 새 렌더러입니다.

렌더링 파이프라인이 이렇게 바뀝니다.

```
[Old]
JS 상태 변경
  → Bridge → Shadow 스레드에서 레이아웃 계산
  → Bridge → UI 스레드에서 실제 렌더

[New]
JS 상태 변경
  → JSI로 즉시 C++ Shadow Tree 업데이트 (동기)
  → 같은 C++ 공간에서 Yoga 레이아웃 계산
  → Commit
━━━━ 여기까지 한 프레임 안에 ━━━━
  → UI 스레드가 Mount
```

Old에서 자주 보이던 다음과 같은 현상이 Fabric에서 많이 개선됩니다.

- 키보드가 올라올 때 레이아웃이 **한 프레임 튀는** 현상
- 화면 전환 시 이전 화면이 순간적으로 남는 **깜빡임**
- 빠른 스크롤 중 FlatList가 **빈 공간**을 보이는 현상

원인은 전부 "JS 상태 변경 → 레이아웃 → 렌더링"이 여러 프레임에 걸쳐 어긋나서 발생했는데, Fabric은 이 단계를 **C++ 안에서 한 번에 묶어** 어긋남을 제거합니다.

### 3. Codegen — 컴파일 타임 타입 안전성

JSI만 있으면 여전히 **이름 기반 호출**이라 타입 에러는 런타임까지 못 잡습니다. 이 문제를 해결하는 것이 **Codegen**입니다.

개발자는 TypeScript로 모듈 명세(Spec)를 한 번만 작성합니다.

```ts
// NativeCameraModule.ts
export interface Spec extends TurboModule {
  takePhoto(options: { quality: string }): Promise<string>;
}
```

빌드 타임에 Codegen이 이 Spec을 읽어서 **iOS / Android / C++ 바인딩 코드를 자동 생성**합니다.

```
[개발자가 쓰는 것]  Spec.ts 하나
         ↓ (빌드 시 Codegen 자동 실행)
[Codegen이 만드는 것]
  ├─ iOS 헤더 (.h)
  ├─ Android 인터페이스 (Kotlin)
  └─ C++ 공통 바인딩
```

결과적으로,

- JS / Obj-C++ / Kotlin 바인딩을 손으로 동기화할 필요가 없습니다.
- 이름이 바뀌면 모든 플랫폼 바인딩이 한꺼번에 갱신됩니다.
- 타입 불일치를 **컴파일 타임에** 잡습니다.

## 📱 그래서 지금 Expo에서는?

### New Architecture는 이미 기본값

- **Expo SDK 52+**: New Architecture가 **기본 활성화**된 상태로 새 프로젝트가 만들어집니다.
- **Expo SDK 55+**: New Architecture가 **항상 켜져 있고, 끌 수 없습니다.**
- **RN 0.76+**: 순수 RN에서도 New Architecture가 기본값입니다.

### 확인하는 법

`app.json` 또는 `app.config.js`를 열어봅니다.

```json
{
  "expo": {
    "newArchEnabled": true
  }
}
```

- `true`: New Architecture
- `false`: Old Architecture (SDK 54까지만 가능)
- 생략: SDK 기본값

런타임에서도 확인할 수 있습니다.

```js
import { TurboModuleRegistry } from 'react-native';

const isNewArch = TurboModuleRegistry.get('PlatformConstants') !== null;
```

### 내 코드는 뭐가 달라지나

**대부분 안 달라집니다.** `<View>`, `<Text>`, `useState`, 대부분의 Expo 라이브러리는 그대로 씁니다.

다만 서드파티 라이브러리가 New Architecture 호환인지는 확인해야 합니다. Expo 공식 모듈은 SDK 51부터 전부 호환되며, 아래 명령으로 의존성 호환성을 점검할 수 있습니다.

```bash
npx expo-doctor@latest
```

### 체감 가능한 변화

- **앱 콜드 스타트가 빨라집니다** — TurboModule의 Lazy 로딩 덕분입니다.
- **애니메이션과 제스처가 덜 끊깁니다** — JSI의 직접 호출이 Bridge 왕복을 대체합니다.
- **키보드 튐, 화면 전환 깜빡임이 줄어듭니다** — Fabric의 동기 렌더링 덕분입니다.

## 📝 마치며

한 줄로 요약하면 이렇습니다.

> **JSI라는 C++ 직접 통로를 깔고, 그 위에 모듈(TurboModule) / 렌더러(Fabric) / 타입(Codegen)을 얹어서 Bridge 시절의 고통을 지웠다.**

SDK 52~53으로 처음 시작한 입장에서는 이 구조 대부분이 **이미 완성된 상태**로 주어져 있습니다. 하지만 `useNativeDriver`, `InteractionManager` 같이 Old 시절의 흔적이 라이브러리 문서 구석에 남아 있고, 커뮤니티 글에도 "Bridge"라는 단어가 자주 등장합니다.

그때마다 이 글의 큰 그림을 떠올리면, **왜 그 우회책이 존재했고 왜 지금은 덜 중요해졌는지**가 같이 읽힙니다.

## 🔗 레퍼런스

> **이 글을 쓰며 참고한 자료**
> - [About the New Architecture · React Native 공식 문서](https://reactnative.dev/architecture/landing-page)
> - [React Native's New Architecture · Expo Documentation](https://docs.expo.dev/guides/new-architecture/)
> - [React Native Architecture: From Bridge to Fabric · Codeminer42 Blog (본문 아키텍처 다이어그램 출처)](https://blog.codeminer42.com/react-native-architecture-from-bridge-to-fabric/)
