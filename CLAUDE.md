# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

Jekyll 기반 개인 기술 블로그 (jekyll-theme-chirpy 테마)

## 명령어

```bash
bundle exec jekyll serve  # 로컬 서버 실행
bundle exec jekyll build  # 빌드
```

## 커스텀 명령어 (테크니컬 라이팅)

학습/트러블슈팅 후 블로그 포스트를 작성하는 워크플로우:

| 명령어 | 설명 |
|--------|------|
| `/teaching-mode [주제]` | 티칭 모드. 단계별 설명 + 이해도 검증 + 오해 교정의 구조화된 학습 세션 |
| `/troubleshoot [문제]` | 트러블슈팅 세션 시작. 문제 분석 및 해결 |

### 사용 플로우

1. `/teaching-mode` 또는 `/troubleshoot`로 세션 시작
2. 대화를 통해 학습/문제 해결 진행
3. 블로그 포스트로 남기고 싶을 때 `/write-post`로 문서화 (유저 스코프 스킬)
4. 최종 검토 후 커밋

## 포스트 작성

### 파일 구조
- **위치**: `_posts/`
- **파일명**: `YYYY-MM-DD-제목.md`
- **템플릿**: `template/New Post.md`

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
- **제목**: `[기술명] 제목` 형식 (예: `[Swift]`, `[ReactNative]`, `[UIKit]`, `[부스트캠프]`)
- **description**: 1-2문장 요약 (SEO 메타 태그로 사용)
- **섹션**: 이모지 + 제목 (예: `## 🤔 제목`)
- **레퍼런스**: 마지막에 `## 🔗 레퍼런스` 섹션 포함
- **Prompt 블록**: `{: .prompt-info }`, `{: .prompt-tip }` 사용 가능

### 태그 컨벤션
- **기술 태그** (영어): `Architecture`, `Concurrency`, `CI-CD`, `MVVM`, `TCA`, `SPM`, `Storyboard` 등
- **유형 태그** (영어): `tutorial`, `troubleshooting`, `review`
- **비기술 태그** (한글): `회고`, `후기`
- 프로젝트명은 태그에 넣지 않음
- 카테고리와 중복되는 태그(Swift, UIKit 등) 사용하지 않음

## 이미지 관리

**경로**: `assets/img/post/연도/`

## 카테고리 구조

| 상위 카테고리 | 하위 카테고리 |
|-------------|-------------|
| Mobile | iOS, Android, ReactNative |
| Web | React, ... |
| Backend | NestJS, SpringBoot, ... |
| Game | Unity, ... |
| Language | Swift, Kotlin, TypeScript, Go, ... |
| DevOps | CI/CD, Docker, ... |
| CS | ComputerScience, Algorithm, DataStructure, DesignPattern |
| Log | Boostcamp, 회고, 후기 |

## 지식 저장소

- **위치**: `_notes/` (Jekyll 빌드 제외, 옵시디언 전용)
- **용도**: 학습 로드맵, 주제별 노트, 지식 연결 (wikilink)
- **구조**: `_notes/{카테고리}/{주제}/`

## Git 워크플로우

**커밋 메시지**: `docs: 설명` 형식

예시:
- `docs: "제목" 문서 추가`
- `docs: "제목" 문서 수정`
- `docs: 문서 카테고리 리네이밍`
