---
name: web-presentation
description: >
  순수 HTML, CSS, JavaScript만으로 웹 기반 슬라이드 프레젠테이션을 구현하는 스킬.
  외부 라이브러리 없이 단일 HTML 파일로 완성되며, 15개 JS 모듈(키보드 네비게이션,
  WAI-ARIA 접근성, 프레젠터 모드, 코드 하이라이트, 다크/라이트 테마, 자동재생,
  오버뷰 그리드, View Transitions 등)을 포함하는 고품질 프레젠테이션을 생성한다.
  이 스킬의 아키텍처 참조 없이는 이 수준의 프레젠테이션을 만들 수 없으므로 반드시
  사용해야 한다. 사용자가 프레젠테이션, 슬라이드, 발표 자료, PPT 대체, 웹 슬라이드,
  HTML 슬라이드, 강의 자료, 세미나 발표, 학습 자료 프레젠테이션 등을 언급하면
  이 스킬을 사용한다. reveal.js 같은 외부 라이브러리 대신 바닐라로 직접 구현하고
  싶다는 맥락에서도 트리거된다. 사용자가 명시적으로 .pptx 파일을 요청하지 않는 한,
  슬라이드나 프레젠테이션 요청에는 이 스킬을 우선 사용한다.
---

# 바닐라 웹 프레젠테이션 스킬

순수 HTML, CSS, JavaScript만으로 웹 기반 슬라이드 프레젠테이션을 구현한다.
단일 `.html` 파일로 완성되며, 외부 의존성이 없다.

## 핵심 원칙

1. **단일 파일**: 모든 HTML, CSS, JS가 하나의 `.html` 파일에 포함
2. **외부 의존성 제로**: reveal.js, impress.js 등 외부 라이브러리 사용 금지 (웹폰트만 허용)
3. **960×540 고정 캔버스**: `transform: scale()`로 뷰포트에 맞춰 자동 스케일링
4. **자동 순차 등장**: Fragment는 슬라이드 진입 시 자동 등장 (`data-manual` 시 수동 step)
5. **다크 테마 기본**: 어두운 배경에 밝은 텍스트, `T` 키로 라이트 테마 전환
6. **접근성**: WAI-ARIA Carousel 패턴, `inert`, `:focus-visible` 적용
7. **인쇄 지원**: `@media print`로 슬라이드별 페이지 분리
8. **Progressive Enhancement**: View Transitions API 지원 시 활용, 미지원 시 CSS transition 폴백

## 구현 순서

1. 사용자의 프레젠테이션 주제와 슬라이드 구성을 파악
2. `references/architecture.md`를 읽고 상세 구현 패턴 확인
3. 아래 구조 템플릿에 따라 단일 HTML 파일 작성
4. 콘텐츠 오버플로 검증 (슬라이드당 콘텐츠 영역 약 460px)

## HTML 구조

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>프레젠테이션 제목</title>
  <style>/* CSS 전체 */</style>
</head>
<body>
  <main class="presentation" role="region" aria-roledescription="carousel" aria-label="제목">
    <div class="slides" aria-live="polite">
      <section class="slide title-slide active" id="slide-1"
               role="group" aria-roledescription="slide" aria-label="1 of N" tabindex="-1">
        <div class="slide-content">...</div>
        <aside class="speaker-notes" hidden>발표자 노트</aside>
      </section>
      <section class="slide" id="slide-2"
               role="group" aria-roledescription="slide" aria-label="2 of N"
               tabindex="-1" aria-hidden="true" inert>
        <div class="slide-content">...</div>
      </section>
      <!-- 추가 슬라이드 -->
    </div>
  </main>

  <!-- UI 오버레이 -->
  <div class="progress-bar"><div class="progress-fill" id="progressFill"></div></div>
  <div class="slide-counter"><span id="counterCurrent">1</span> / <span id="counterTotal">N</span></div>
  <div class="help-overlay" id="helpOverlay">...</div>
  <div class="notes-panel" id="notesPanel"></div>
  <div class="autoplay-indicator" id="autoplayIndicator"></div>

  <script>/* JS 엔진 전체 */</script>
</body>
</html>
```

## CSS 아키텍처

### 디자인 토큰 (CSS 변수)

```css
:root {
  --color-bg: #0f0f23;
  --color-surface: #1a1a2e;
  --color-primary: #4fc3f7;
  --color-secondary: #e040fb;
  --color-accent: #ffd740;
  --color-text: #e0e0e0;
  --color-text-dim: #888;
  --color-success: #66bb6a;
  --color-danger: #ef5350;
  --font-sans: 'Noto Sans KR', sans-serif;
  --font-mono: 'Fira Code', 'Consolas', monospace;
  --slide-width: 960px;
  --slide-height: 540px;
}
```

색상 팔레트는 주제에 맞게 조정할 수 있지만, 다크 배경 + 고대비 텍스트 원칙은 유지한다.
`[data-theme="light"]`에서 밝은 배경 변수를 오버라이드하여 라이트 테마를 제공한다.

### 레이아웃 핵심

- **뷰포트 컨테이너**: `.presentation` — `100vw × 100vh`, `display: grid`, `place-items: center`
- **슬라이드 캔버스**: `.slides` — `960×540px`, `display: grid`, `transform: scale(var(--slide-scale))`
- **슬라이드 스택**: `.slide` — `grid-area: 1/1`로 모든 슬라이드가 같은 셀에 겹침
- **활성 슬라이드**: `.slide.active` — `opacity: 1`, `visibility: visible`, `z-index: 1`

`grid-area: 1/1` 패턴으로 `position: absolute` 없이 자연스러운 슬라이드 스택을 구현한다.

## 슬라이드 유형

### 타이틀 슬라이드

```html
<section class="slide title-slide active" id="slide-1">
  <div class="slide-content">
    <h1>프레젠테이션 제목</h1>
    <p class="subtitle fragment" data-step="1">부제목</p>
    <p class="fragment" data-step="2">
      <span class="tag tag-blue">태그1</span>
      <span class="tag tag-purple">태그2</span>
    </p>
  </div>
</section>
```

- `background: linear-gradient(135deg, ...)` 그라데이션 배경
- h1에 `background-clip: text` 그라데이션 텍스트 효과

### 섹션 구분 슬라이드

```html
<section class="slide section-slide" id="slide-N">
  <div class="slide-content" style="text-align: center;">
    <h2>섹션 제목</h2>
  </div>
</section>
```

### 콘텐츠 슬라이드

일반 콘텐츠, 코드 블록, 테이블, 다이어그램 등을 포함하는 기본 슬라이드.

## 콘텐츠 컴포넌트

### Fragment (자동 순차 등장)

```html
<li class="fragment" data-step="1">첫 번째 항목</li>
<li class="fragment" data-step="2">두 번째 항목</li>
```

- `data-step` 값이 같은 요소는 동시에 등장
- 슬라이드 진입 시 `(step - 1) * 0.2초` 딜레이로 자동 순차 등장
- 방향키는 Fragment 개별 진행이 아닌 **페이지 단위 이동**
- `data-manual` 속성이 있는 슬라이드에서는 방향키로 step 단위 수동 진행

### 코드 블록

```html
<div class="code-block">
<pre><span class="keyword">const</span> <span class="attr">x</span> = <span class="number">42</span>;</pre>
</div>
```

구문 강조 클래스: `.comment`, `.keyword`, `.string`, `.func`, `.type`, `.number`, `.attr`

### 코드 줄 하이라이트

```html
<div class="code-block" data-highlight-steps="1-3|5|7-9">
<pre>
<span class="code-line" data-line="1"><span class="keyword">const</span> <span class="attr">a</span> = <span class="number">1</span>;</span>
</pre>
</div>
```

- `data-highlight-steps`를 `|`로 구분하여 step별 강조 줄 정의
- 자동 모드: `transition-delay`로 순차 강조 이동
- 수동 모드 (`data-manual`): fragment step 완료 후 하이라이트 step 진행

### 2단 레이아웃

```html
<div class="two-columns">
  <div class="column">왼쪽 내용</div>
  <div class="column">오른쪽 내용</div>
</div>
```

### 다이어그램 (플로우)

```html
<div class="diagram">
  <div class="diagram-box"><h4>A</h4><p>설명</p></div>
  <span class="diagram-arrow">→</span>
  <div class="diagram-box highlight"><h4>B</h4><p>강조</p></div>
</div>
```

### 콜아웃

```html
<div class="callout">기본 정보</div>
<div class="callout warning">주의사항</div>
<div class="callout danger">위험/금지 사항</div>
```

### 비교 카드

```html
<div class="compare-grid">
  <div class="compare-card good"><h4>좋은 예</h4><p>설명</p></div>
  <div class="compare-card bad"><h4>나쁜 예</h4><p>설명</p></div>
</div>
```

### 태그

```html
<span class="tag tag-blue">파란색</span>
<span class="tag tag-purple">보라색</span>
<span class="tag tag-yellow">노란색</span>
<span class="tag tag-green">초록색</span>
<span class="tag tag-red">빨간색</span>
```

### 테이블

표준 `<table>` 사용. 헤더는 `--color-primary` 배경, 행 구분은 얇은 border.

## JavaScript 엔진

필수 기능 15가지를 IIFE 패턴으로 구현한다:

1. **뷰포트 스케일링** — `fitSlides()`: 960×540 캔버스를 뷰포트에 맞춰 `--slide-scale` 조정
2. **Fragment 자동 등장** — `autoRevealFragments(slide)`: `data-step` 기반 `transition-delay` 설정 후 `.visible` 추가
3. **Fragment 리셋** — `resetFragments(slide)`: 슬라이드 이탈 시 `.visible` 제거
4. **슬라이드 전환** — `showSlide(index)`: View Transitions API 래핑, 애니메이션 잠금
5. **키보드 네비게이션** — `←→`, `Home/End`, `F`, `?`, `Space`, `PageUp/Down`, `T`, `O`, `S`, `P`, `N`
6. **터치/스와이프** — Pointer Events로 좌우 스와이프 감지 (threshold 50px)
7. **URL Hash 동기화** — `#slide-N` 형식, `replaceState`로 히스토리 오염 방지
8. **프로그레스 바** — 하단 그라데이션 바 + 슬라이드 카운터
9. **코드 줄 하이라이트** — `data-highlight-steps` 파싱, step별 `.highlighted` 토글
10. **다크/라이트 테마** — `T` 키로 `data-theme` 속성 전환, CSS 변수 오버라이드
11. **슬라이드 오버뷰** — `O` 키로 축소 그리드 뷰, 클릭으로 슬라이드 점프
12. **인라인 노트 패널** — `N` 키로 현재 슬라이드의 `speaker-notes` 표시/숨김
13. **자동 재생** — `P` 키로 일정 간격 자동 슬라이드 전환, 일시정지 시각 피드백
14. **프레젠터 모드** — `S` 키로 동기적 `window.open`, 현재/다음 슬라이드 + 노트 표시
15. **수동 Fragment Step** — `data-manual` 슬라이드에서 방향키로 step 단위 진행

### 입력 잠금 패턴

```javascript
let isAnimating = false;

function showSlide(index) {
  if (isAnimating) return;
  isAnimating = true;
  // ... 전환 로직 ...
  // transitionend + setTimeout 이중 해제
  slide.addEventListener('transitionend', handler, { once: true });
  setTimeout(() => { isAnimating = false; }, 600);
}
```

`transitionend`가 간혹 미발생하므로 `setTimeout` 안전장치를 반드시 포함한다.

## 키보드 단축키

| 키 | 동작 |
|----|------|
| →↓ Space PageDown | 다음 슬라이드 (수동 모드: 다음 step) |
| ←↑ PageUp | 이전 슬라이드 (수동 모드: 이전 step) |
| Home / End | 첫/마지막 슬라이드 |
| F | 풀스크린 토글 |
| T | 다크/라이트 테마 전환 |
| O | 슬라이드 오버뷰 토글 |
| S | 프레젠터 뷰 열기 |
| P | 자동 재생 일시정지/재개 |
| N | 노트 패널 토글 |
| ? | 도움말 토글 |
| Escape | (1) 도움말 닫기 → (2) 오버뷰 탈출 → (3) 풀스크린 종료 |

## 콘텐츠 오버플로 방지

슬라이드 패딩(상하 40px)을 제외한 실제 콘텐츠 영역은 약 **460px**이다.

| 제약 | 권장 |
|------|------|
| 코드 블록 | 15줄 이내 |
| 리스트 항목 | 6~8개 이내 |
| 테이블 + 코드 공존 | 슬라이드 분할 고려 |

내용이 많으면 슬라이드를 나누거나 `two-columns`로 수직 공간을 절약한다.

## 체크리스트

- [ ] 단일 HTML 파일로 완성
- [ ] 외부 라이브러리 미사용 (웹폰트만 허용)
- [ ] CSS 변수로 디자인 토큰 관리
- [ ] `grid-area: 1/1` 슬라이드 스택
- [ ] `transform: scale()` 뷰포트 스케일링
- [ ] Fragment 자동 순차 등장 (`data-step` + `transition-delay`)
- [ ] 방향키 = 페이지 단위 이동 (`data-manual` 시 step 단위)
- [ ] 키보드, 터치, URL hash 네비게이션
- [ ] 프로그레스 바 + 슬라이드 카운터
- [ ] 도움말 오버레이 (`?` 키)
- [ ] 풀스크린 지원 (`F` 키)
- [ ] 입력 잠금 (isAnimating + setTimeout 안전장치)
- [ ] 콘텐츠 오버플로 없음
- [ ] `counterTotal` 값이 실제 슬라이드 수와 일치
- [ ] ARIA 속성 (`aria-roledescription`, `aria-hidden`, `inert`)
- [ ] `@media print` 인쇄 지원 (`speaker-notes[hidden]` 오버라이드 포함)
- [ ] `:focus-visible` 포커스 스타일
- [ ] `[data-theme="light"]` 라이트 테마 변수
- [ ] `data-manual` 수동 Fragment step
- [ ] `data-highlight-steps` 코드 줄 하이라이트
- [ ] 오버뷰 모드 (`O` 키, `inert` 해제/복원)
- [ ] View Transitions API + 폴백 (`.vt-active` 이중 애니메이션 방지)
- [ ] 프레젠터 뷰 (`S` 키, 동기적 `window.open`)
- [ ] 자동재생 (`P` 키, 일시정지 시각 피드백)
- [ ] 인라인 노트 패널 (`N` 키)

## 상세 구현 참조

- `references/architecture.md` — CSS 전체 코드, JS 엔진 전체 코드, 도움말 HTML
- `references/accessibility.md` — WAI-ARIA 패턴, 포커스 관리, 접근성 상세
- `references/advanced-features.md` — 프레젠터 모드, 오버뷰, 자동재생 상세 구현
