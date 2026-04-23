---
title: "[TypeScript] type, interface, 그리고 import type 이해하기"
date: 2026-04-24 00:30 +0900
description: "TypeScript의 type과 interface는 무엇이 다르고, import 앞에 붙는 type은 왜 필요한가. 타입 소거라는 컴파일 모델에서 출발해 세 문법을 하나의 이야기로 연결합니다."
categories:
  - Language
  - TypeScript
tags:
  - tutorial
  - type-alias
  - interface
  - import-type
  - declaration-merging
---

> 이 글은 **TypeScript를 막 시작했거나, `type`과 `interface`를 둘 다 써봤지만 차이가 애매했던 개발자**를 대상으로 합니다.
> `import type`이 붙는 이유까지 한 흐름으로 엮어서 설명합니다.
{: .prompt-info }

## 🤔 왜 이 글을 쓰게 됐나

TypeScript를 쓰다 보면 자연스럽게 마주치는 세 가지 문법이 있습니다.

```ts
type User = { id: string; name: string };
interface User { id: string; name: string }
import type { User } from "./user";
```

처음엔 셋 다 비슷해 보여서, 팀 컨벤션이나 IDE 자동완성을 따라 쓰게 됩니다. 그러다 어느 순간 질문이 떠오릅니다.

- `type`이랑 `interface`는 **왜 둘 다 있는 걸까?** 이름만 다를 뿐 기능이 같아 보이는데.
- `import type`은 일반 `import`랑 **뭐가 다른 걸까?** 없어도 돌아가는 것 같은데.

이 세 질문은 사실 **한 줄기**로 연결돼 있습니다. TypeScript가 컴파일되는 방식 — **"타입은 런타임에 사라진다"** 는 사실을 이해하면, 세 문법이 왜 모두 필요한지 자연스럽게 보입니다.

## TypeScript가 푸는 문제

본론에 들어가기 전에 짚고 갑시다. **TypeScript는 무엇을 위한 도구인가?**

JavaScript는 변수에 무엇이 들어있는지 **실행해보기 전까지** 알 수 없습니다.

```js
function greet(user) {
  return "Hello, " + user.name.toUpperCase();
}

greet({ name: "Hooni" }); // "Hello, HOONI"
greet({ name: 42 });       // 런타임 에러
greet("Hooni");            // 런타임 에러
greet();                    // 런타임 에러
```

네 번의 호출 중 세 번이 **실제로 실행해야** 에러를 발견할 수 있습니다. 서비스가 커지면 이런 실수가 프로덕션에서 터집니다.

TypeScript는 이걸 **컴파일 시점으로 옮깁니다**.

```ts
function greet(user: { name: string }) {
  return "Hello, " + user.name.toUpperCase();
}

greet({ name: 42 }); // ❌ 컴파일 에러: number는 string이 아닙니다
greet();             // ❌ 컴파일 에러: 인자가 필요합니다
```

> TypeScript는 **"값의 모양에 이름을 붙이고, 그 약속을 강제하는 도구"** 입니다.

그리고 그 "모양에 이름 붙이기"를 하는 두 가지 문법이 바로 `type`과 `interface`입니다.

## type — 어떤 타입에든 이름을 붙이는 별명

`type`의 정체는 한 단어로 **"별명 짓기(alias)"** 입니다. 이미 존재하는 타입에 새 이름을 붙이는 문법이에요.

### 가장 단순한 예

```ts
type UserId = string;

const id: UserId = "hooni-001"; // OK
const wrong: UserId = 42;        // ❌ 에러
```

`UserId`는 그냥 `string`입니다. 하지만 코드 곳곳에서 `string`이라고 쓰는 것보다 `UserId`라고 쓰면 **의도가 명확**해집니다.

### 객체 모양에도 이름 붙이기

```ts
type User = {
  id: string;
  name: string;
  age: number;
};
```

### 조합 표현 — type이 특히 강한 영역

`type`은 **"이것 또는 저것"**, **"이것이면서 저것"** 같은 조합을 자연스럽게 표현합니다.

```ts
type Status = "loading" | "success" | "error"; // 유니온: 셋 중 하나
type Admin  = User & { role: "admin" };         // 인터섹션: 둘 다 만족

const s: Status = "loading"; // OK
const s2: Status = "done";   // ❌ 셋 중 하나가 아님
```

여기서 `"loading"` 같은 값은 **리터럴 타입(literal type)** 입니다. "string 중에서도 값이 정확히 `"loading"`인 것만 허용"이라는 뜻이에요. 이걸 `|`로 묶은 게 **유니온 타입**이고요.

즉 `Status`는 **"string의 부분집합"** 입니다. `Status ⊂ string` 관계죠.

> **핵심**: `type`은 아무 타입이든 골라서 거기에 이름을 붙이는 문법입니다. 객체든, 문자열이든, "A 또는 B"든, 뭐든 됩니다.

## interface — 객체 모양 전용 계약

`interface`는 **"객체(또는 클래스)의 모양을 정의하는 전용 문법"** 입니다. `type`과 달리 용도가 좁습니다 — 객체 모양 외엔 다루지 못합니다.

### 기본 사용

```ts
interface User {
  id: string;
  name: string;
  age: number;
}
```

여기까지는 `type User = { ... }`와 완전히 똑같아 보입니다. 실제로 대부분 경우 결과도 같습니다.

### 확장 — extends

객체 모양을 **상속하듯** 이어 붙일 수 있습니다.

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}
```

`type`으로도 비슷하게 조립할 수 있습니다 — `&`(인터섹션)을 쓰면 됩니다.

```ts
type Animal = { name: string };
type Dog = Animal & { breed: string };
```

### interface만의 고유 능력 — 선언 병합

같은 이름의 `interface`를 **여러 번 선언**하면, TypeScript가 **자동으로 합쳐줍니다**.

```ts
interface User {
  id: string;
}

interface User {
  name: string;
}

// 실제로는 { id: string; name: string } 인 것처럼 동작
const u: User = { id: "1", name: "Hooni" };
```

`type`은 이게 불가능합니다. 같은 이름으로 두 번 선언하면 중복 식별자 에러가 납니다.

## 둘의 실질적 차이

정면으로 비교하면 차이는 크게 세 가지입니다.

### 1. 표현 범위

| 표현 | `type` | `interface` |
|---|:---:|:---:|
| 객체 모양 | ✅ | ✅ |
| 유니온 <code>A &#124; B</code> | ✅ | ❌ |
| 튜플 `[string, number]` | ✅ | ❌ |
| 원시/리터럴 별명 (`type ID = string`) | ✅ | ❌ |
| `keyof`, 조건부 타입 등 고급 연산 | ✅ | ❌ |

`interface`는 객체 전용, `type`은 만능. **표현력 자체는 `type`이 더 넓습니다.**

### 2. 확장 문법의 차이 — 충돌 감지

결과는 비슷하지만 **에러 메시지가 다릅니다.** `interface`의 `extends`는 충돌을 즉시 잡아주고, `type`의 `&`는 조용히 `never` 타입을 만들 수 있습니다.

```ts
interface A { x: number }
interface B extends A { x: string } // ❌ 충돌 즉시 알려줌

type A2 = { x: number }
type B2 = A2 & { x: string } // 에러 없음. 하지만 B2.x의 타입은 never
```

대형 프로젝트에선 이 **즉시 에러**가 디버깅 시간을 아껴줘서 `interface`를 선호하는 팀도 있습니다.

### 3. 선언 병합 — 양날의 검

`interface`만의 고유 능력인 선언 병합은 강력한 만큼 위험합니다.

**왜 위험한가?** 앱 코드에서 같은 이름이 두 파일에 실수로 선언되면, 에러 없이 **조용히 합쳐져 버립니다.** 디버깅할 때 "분명 이 필드 없는데 왜 타입에 보이지?" 하고 하루를 날릴 수 있습니다.

**왜 유용한가?** 서드파티 라이브러리 타입을 확장할 때 필수적입니다. 대표적인 예가 Express의 `Request`에 필드를 추가하는 경우입니다.

```ts
// Express의 Request 타입에 user 필드 추가하기
declare global {
  namespace Express {
    interface Request {
      user?: { id: string };
    }
  }
}
```

`type`이었다면 원본 정의를 건드리지 않고는 불가능합니다. 선언 병합 덕분에 라이브러리를 수정하지 않고도 타입을 확장할 수 있습니다.

### 선언 병합의 스코프 규칙

선언 병합이 실제로 어디서 일어나는지는 **스코프** 기준으로 정해집니다.

| 상황 | 병합 여부 |
|------|----------|
| **같은 파일**에서 같은 이름 두 번 | ✅ 병합 |
| **다른 모듈 파일** 각자 로컬 선언 (`export` 없음) | ❌ 서로 다른 타입 |
| **다른 모듈 파일**에서 `import`해 같은 이름 재선언 | ❌ 중복 식별자 에러 |
| **`declare global`** 안에서 선언 | ✅ 병합 |
| **같은 ambient 모듈**(`declare module "x"`) | ✅ 병합 |

여기서 숨은 함정이 하나 있습니다. TypeScript는 파일을 **모듈** 또는 **스크립트**로 분류합니다.

- **모듈**: `import`나 `export`가 하나라도 있는 파일 → 그 파일만의 로컬 스코프
- **스크립트**: `import`/`export`가 전혀 없는 파일 → **전역 스코프**

스크립트 파일끼리는 동명 interface가 **전역에서 병합됩니다**. `.d.ts` 타입 선언 파일에서 특히 잘 일어나는 사고입니다. 파일에 `export {}` 한 줄을 박아 강제로 모듈로 만드는 관용구는 이 때문에 쓰입니다.

실수로 인한 병합을 막고 싶다면 ESLint 룰도 있습니다.

```json
{
  "rules": {
    "@typescript-eslint/no-redeclare": "error"
  }
}
```

## 그래서 뭘 써야 하나

TypeScript 공식 팀의 입장:

> "Prefer `interface`. Use `type` when you need features `interface` doesn't support."
> — TypeScript Handbook

반면 커뮤니티(특히 React 생태계)의 많은 팀은 다음을 선호합니다.

> "기본은 `type`, 선언 병합이 필요한 드문 경우만 `interface`."

| 입장 | 이유 |
|------|------|
| 공식 팀: `interface` 우선 | 에러 메시지, 컴파일러 캐시 효율, 확장의 의도성 |
| 커뮤니티: `type` 우선 | 표현 일관성(유니온도 모두 `type`), 선언 병합 사고 방지 |

**결론**: 팀 컨벤션을 따르되, 없으면 **"기본 `type`, 라이브러리 확장이나 선언 병합이 필요할 때만 `interface`"** 가 요즘 흐름에 가깝습니다.

## 타입은 런타임에 사라진다

이제 글의 후반부, `import type`으로 넘어가기 위한 결정적 사실을 짚을 차례입니다.

> **TypeScript는 런타임이 없습니다. 전부 JavaScript로 컴파일되어 실행됩니다.**

즉, `type`, `interface`, 타입 주석은 **컴파일 시점에만 존재하고, JS로 바뀔 때 전부 지워집니다**. 이걸 **type erasure(타입 소거)** 라고 합니다.

```ts
// user.ts
export interface User {
  id: string;
  name: string;
}

export function greet(u: User): string {
  return `Hello, ${u.name}`;
}
```

이 파일이 JS로 컴파일되면 다음과 같이 바뀝니다.

```js
// user.js
export function greet(u) {
  return `Hello, ${u.name}`;
}
// interface User? 어디에도 없습니다.
```

### 뭐가 남고 뭐가 사라지나

| 요소 | 컴파일 후 |
|------|----------|
| `type` / `interface` | 사라짐 |
| 타입 주석 (`: string`) | 사라짐 |
| `enum` | 남음 (JS 객체로 변환) |
| `class` | 남음 (JS 클래스로 변환) |
| 함수, 변수, 값 | 남음 |

핵심은 **같은 파일에서 `export`되는 것도, "타입"과 "값"은 컴파일 후 운명이 완전히 다르다**는 점입니다.

```ts
// 같은 파일
export interface User { ... }          // 타입 — 사라짐
export function createUser() { ... }   // 값 — 남음
export class UserStore { ... }         // 값 — 남음
```

이 사실이 `import type`의 존재 이유로 직결됩니다.

## import type의 정체

다음 코드를 보겠습니다.

```ts
// types.ts
export interface User { id: string; name: string; }
export class Logger { log(msg: string) { console.log(msg); } }

// app.ts
import { User, Logger } from "./types";

function greet(u: User) { ... }
const logger = new Logger();
```

컴파일러는 판단해야 합니다.

- `User`는 타입으로만 쓰이므로 → JS에서 **지워도 됨**
- `Logger`는 `new Logger()`로 실제 값으로 쓰이므로 → JS에 **남겨야 함**

TypeScript 공식 컴파일러(`tsc`)는 파일 전체를 보고 이를 잘 판단해서 불필요한 import를 제거합니다. 그런데 문제는 다음 세 가지 경우에서 생깁니다.

### 1. 다른 트랜스파일러와의 호환

Babel, esbuild, swc 같은 도구는 **파일 단위로 독립적으로** 변환합니다. `types.ts`를 보기 전에 `app.ts`만 보고 판단해야 해서, `User`가 타입인지 값인지 알 수 없습니다. 그래서 일단 import를 JS에 남겨둡니다.

```js
import { User, Logger } from "./types";  // User도 그대로 남김
```

런타임에 `User`는 존재하지 않으므로 런타임 에러가 납니다.

**해결** — `import type`으로 "이건 타입이야"를 명시합니다.

```ts
import type { User } from "./types";     // 타입이라고 명시 → 확실히 제거
import { Logger } from "./types";         // 값
```

이러면 Babel/esbuild도 파일 하나만 보고도 **"아, `User` import는 통째로 지워도 되겠구나"** 를 알 수 있습니다.

### 2. 사이드 이펙트 격리

어떤 모듈은 단순히 import하는 것만으로 **사이드 이펙트**가 일어납니다.

```ts
// analytics.ts
console.log("analytics 초기화"); // 로드되면 즉시 실행
export interface Event { ... }
```

다른 파일에서 타입만 쓰고 싶은데 일반 `import`로 가져오면, 의도치 않게 저 `console.log`까지 실행될 수 있습니다. `import type`은 컴파일 후 완전히 사라지므로 **"이 파일은 타입만 필요해, 실제 로드는 하지 마"** 를 명확히 말할 수 있습니다.

### 3. 순환 참조 회피

파일 A가 B의 타입을, B가 A의 타입을 필요로 할 때, 일반 import로 묶이면 런타임에 **순환 참조**가 일어날 수 있습니다. `import type`은 런타임에 남지 않으므로 **순환 고리를 자연스럽게 끊어줍니다.**

### 문법 정리

```ts
// ① 타입 전용 import (전체)
import type { User, Role } from "./types";

// ② 섞인 import (TS 4.5+)
import { type User, createUser } from "./user";
//        ^^^^ 타입이라고 인라인 표시

// ③ default도 가능
import type User from "./user";

// ④ export에도 동일한 규칙
export type { User };
```

### verbatimModuleSyntax — 실수 방지 장치

TS 5.0부터 `tsconfig.json`에 다음 옵션을 켜둘 수 있습니다.

```json
{
  "compilerOptions": {
    "verbatimModuleSyntax": true
  }
}
```

이 옵션이 켜지면 **"타입만 쓰이는데 `import type`을 안 썼으면 에러"** 를 냅니다. Babel/esbuild를 쓰는 프로젝트, 모노레포, RN·Expo 같은 환경에서는 켜두는 편이 안전합니다.

## 정리

세 문법은 하나의 이야기로 연결됩니다.

```
TypeScript
├─ 타입을 정의하는 법
│   ├─ type      (아무 타입에든 별명 — 만능)
│   └─ interface (객체/클래스의 모양 계약)
│
├─ 컴파일 모델
│   └─ 타입은 런타임에 사라진다 (type erasure)
│
└─ 타입 전용 import
    └─ import type — 런타임에 확실히 지워지도록 명시
       └─ Babel/esbuild 호환, 사이드 이펙트·순환 참조 회피
```

핵심을 한 줄씩 정리하면 다음과 같습니다.

| 개념 | 한 줄 정리 |
|------|----------|
| `type` | 어떤 타입이든 붙일 수 있는 만능 별명 |
| `interface` | 객체·클래스의 모양 계약 |
| 주요 차이 | `type`은 표현력, `interface`는 선언 병합·확장 충돌 감지 |
| 실무 기본 | "기본 `type`, 선언 병합이 필요할 때만 `interface`"가 커뮤니티 다수 |
| 타입 소거 | 타입 관련 문법은 JS로 컴파일되면 모두 사라진다 |
| `import type` | "이 import는 타입 전용"을 명시 → 번들러가 안전하게 제거 |
| `verbatimModuleSyntax` | 타입 전용 import를 빼먹으면 에러로 강제하는 옵션 |

한 문장으로 압축하면,

> TypeScript의 `type`과 `interface`는 **타입을 정의하는 두 가지 문법**이고, 그 타입들은 **런타임에 사라지기 때문에** `import type`으로 "이건 타입 전용 import야"를 명시해서 번들러가 안전하게 지울 수 있도록 돕는다.

`import type`을 "그냥 관습"이 아니라 **타입 소거 모델의 자연스러운 결론**으로 받아들이면, TypeScript의 컴파일 모델 전체가 한 번에 보입니다.

## 🔗 레퍼런스

> **해당 자료를 찾아보며 도움이 됐던 링크**
> - [TypeScript Handbook — Everyday Types: Differences Between Type Aliases and Interfaces](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)
> - [TypeScript Handbook — Declaration Merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html)
> - [TypeScript 3.8 Release Notes — Type-Only Imports and Export](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html#type-only-imports-and-export)
> - [TypeScript 5.0 Release Notes — `verbatimModuleSyntax`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html#verbatimmodulesyntax)
