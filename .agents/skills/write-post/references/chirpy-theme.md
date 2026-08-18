# Chirpy 테마 특이사항

Jekyll Chirpy 테마를 사용하면서 알아둬야 할 동작들이다.

## Mermaid 다이어그램

- 활성화 조건: 포스트 Front Matter에 `mermaid: true`를 추가해야 `mermaid.js`가 로드된다.
- 누락하면 Mermaid 코드 블록이 다이어그램이 아니라 일반 텍스트로 표시된다. [`failure-cases.md`](failure-cases.md)의 첫 번째 사례를 참고한다.
- 확인 경로: `vendor/bundle/ruby/.../jekyll-theme-chirpy-*/_includes/js-selector.html`의 `{% if page.mermaid %}` 조건
- 권장 테마: `%%{init: {'theme':'neutral'}}%%` 헤더를 사용한다.
  - 라이트 모드와 다크 모드에서 모두 무난하다.
  - 기본 색이나 진한 채우기 색은 다크 모드에서 지나치게 튈 수 있다.
- 강조가 필요하면 채우기 색보다 선의 굵기나 색을 사용한다. 예: `stroke:#555,stroke-width:2px`

## 수학 수식

- Front Matter에 `math: true`가 있어야 MathJax가 로드된다.

## 이미지

- 경로 컨벤션: `assets/img/post/{연도}/`

## Prompt 박스

- `{: .prompt-info }`
- `{: .prompt-tip }`
- `{: .prompt-warning }`
- `{: .prompt-danger }`

## 목차

- 모든 포스트에 사이드바 TOC가 자동 생성된다.
- 본문 서론에 별도의 목차나 흐름표를 반복해서 넣지 않는다.
