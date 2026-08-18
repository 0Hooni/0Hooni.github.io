# GitBlog 프로젝트 지침

이 파일은 GitBlog 저장소에서 작업하는 AI 도구가 공통으로 따르는 프로젝트 지침이다.

## 프로젝트 개요

Jekyll 기반 개인 기술 블로그이며 `jekyll-theme-chirpy` 테마를 사용한다.

## 로컬 명령어

Ruby 3.3.4는 `mise`로 실행한다.

```bash
mise exec -- bash tools/run.sh   # 로컬 서버 실행
mise exec -- bash tools/test.sh  # 프로덕션 빌드와 내부 링크 검증
```

## 프로젝트 스킬

학습이나 트러블슈팅을 마친 뒤 블로그 포스트를 작성하는 워크플로우다. 세 스킬은 명시적으로 요청할 때만 사용한다.

| 스킬 | 설명 |
|------|------|
| `teaching-mode` | 단계별 설명, 이해도 검증, 오해 교정으로 구성된 학습 세션 |
| `troubleshoot` | 기술 문제의 증상과 원인을 검증하고 해결하는 세션 |
| `write-post` | 대화 내용을 블로그 포스트로 작성하고 로컬 결과물까지 검토 |

### 사용 흐름

1. `teaching-mode` 또는 `troubleshoot` 스킬로 세션을 시작한다.
2. 대화를 통해 학습하거나 문제를 해결한다.
3. 블로그 포스트로 남기고 싶을 때 `write-post` 스킬로 문서화한다.
4. 로컬 빌드와 화면 검토를 마친 결과를 사용자가 확인한다.

## 포스트 작성

### 파일 구조

- 위치: `_posts/`
- 파일명: `YYYY-MM-DD-제목.md`
- 템플릿: `template/New Post.md`

### Front Matter

```yaml
---
title: "[카테고리] 제목"
date: YYYY-MM-DD HH:MM +0900
categories:
  - 상위 카테고리
  - 하위 카테고리
tags:
  - 태그1
  - 태그2
---
```

### 작성 컨벤션

- 제목: `[기술명] 제목` 형식. 예: `[Swift]`, `[ReactNative]`, `[UIKit]`, `[부스트캠프]`
- `description`: 1~2문장 요약. SEO 메타 태그로 사용한다.
- 섹션: 이모지와 제목 조합. 예: `## 🤔 제목`
- 레퍼런스: 마지막에 `## 🔗 레퍼런스` 섹션을 포함한다.
- Prompt 블록: `{: .prompt-info }`, `{: .prompt-tip }`을 사용할 수 있다.

### 태그 컨벤션

- 기술 태그는 영어로 쓴다. 예: `Architecture`, `Concurrency`, `CI-CD`, `MVVM`, `TCA`, `SPM`, `Storyboard`
- 유형 태그는 영어로 쓴다. 예: `tutorial`, `troubleshooting`, `review`
- 비기술 태그는 한글로 쓴다. 예: `회고`, `후기`
- 프로젝트명은 태그에 넣지 않는다.
- 카테고리와 중복되는 태그는 사용하지 않는다. 예: `Swift`, `UIKit`

## 이미지 관리

- 경로: `assets/img/post/연도/`

## 카테고리 구조

| 상위 카테고리 | 하위 카테고리 |
|---------------|---------------|
| Mobile | iOS, Android, ReactNative |
| Web | React, ... |
| Backend | NestJS, SpringBoot, ... |
| Game | Unity, ... |
| Language | Swift, Kotlin, TypeScript, Go, ... |
| DevOps | CI/CD, Docker, ... |
| CS | ComputerScience, Algorithm, DataStructure, DesignPattern |
| Log | Boostcamp, 회고, 후기 |

## 지식 저장소

- 위치: `_notes/`. Jekyll 빌드에서 제외하고 옵시디언에서 사용한다.
- 용도: 학습 로드맵, 주제별 노트, 지식 연결
- 구조: `_notes/{카테고리}/{주제}/`

## Git 워크플로우

커밋 메시지는 `docs: 설명` 형식을 사용한다.

예시:

- `docs: "제목" 문서 추가`
- `docs: "제목" 문서 수정`
- `docs: 문서 카테고리 리네이밍`
