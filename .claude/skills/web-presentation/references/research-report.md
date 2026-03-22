# 웹 프레젠테이션 스킬 개선 리서치 종합 보고서

> 리서치 일자: 2026-03-13
> 목적: 순수 HTML, CSS, JavaScript만으로 웹 기반 슬라이드 프레젠테이션을 구현하는 최적 패턴 도출

---

## 목차

1. [슬라이드 레이아웃 & DOM 구조](#1-슬라이드-레이아웃--dom-구조)
2. [키보드 & 마우스 네비게이션](#2-키보드--마우스-네비게이션)
3. [슬라이드 전환 애니메이션](#3-슬라이드-전환-애니메이션)
4. [콘텐츠 애니메이션 (빌드 효과)](#4-콘텐츠-애니메이션-빌드-효과)
5. [기존 스킬 개선 영역](#5-기존-스킬-개선-영역)
6. [현재 스킬 대비 개선 권장사항](#6-현재-스킬-대비-개선-권장사항)

---

## 1. 슬라이드 레이아웃 & DOM 구조

### 1.1 CSS 레이아웃 방식 비교

| 방식 | 장점 | 단점 | 적합 레벨 |
|------|------|------|-----------|
| **CSS Grid** (`grid-area: 1/1`) | absolute 없이 슬라이드 겹침, 2차원 레이아웃 | grid-area 스택 패턴이 직관성 떨어짐 | 슬라이드 컨테이너 (스택) |
| **Flexbox** | 중앙 정렬 한 줄로 해결, 1차원 레이아웃 최적 | 슬라이드 겹침에는 부적합 | 슬라이드 내부 콘텐츠 |
| **Absolute Positioning** | 전환 애니메이션 직관적, 호환성 최상 | normal flow 이탈, 명시적 크기 필수 | 레거시 대안 |

**권장**: Grid (컨테이너 스택) + Flexbox (내부 콘텐츠) 하이브리드 -- **현재 스킬과 동일**

### 1.2 16:9 비율 고정 방법 비교

| 방법 | 코드 간결성 | 비율 보장 | JS 필요 | 브라우저 호환 |
|------|:-----------:|:---------:|:-------:|:------------:|
| **transform scale** (960x540 고정) | ★★ | ★★★ | O | 100% |
| `aspect-ratio: 16/9` | ★★★ | ★★ | X | 96%+ |
| padding-top hack (56.25%) | ★ | ★★ | X | 100% |

**권장**: transform scale -- px 단위 디자인 가능, 비율 절대 보장 -- **현재 스킬과 동일**

### 1.3 HTML 시맨틱 구조

```html
<main class="presentation" role="region" aria-roledescription="carousel" aria-label="제목">
  <div class="slides">
    <section class="slide" role="group" aria-roledescription="slide" aria-label="1 of N">
      <div class="slide-content">...</div>
    </section>
  </div>
</main>
```

**핵심**: `<section>` 태그 + ARIA 속성(`aria-roledescription="slide"`, `aria-hidden` 토글)

### 1.4 최신 CSS 기능 브라우저 지원 (2026년 3월)

| CSS 기능 | 글로벌 지원율 | 비고 |
|----------|:------------:|------|
| `aspect-ratio` | ~96% | IE 미지원 (무관) |
| Container Queries (size) | ~95% | 슬라이드 내부 반응형에 활용 가능 |
| `dvh/svh/lvh` | ~92% | 모바일 주소창 대응 |
| CSS Nesting | ~90% | 코드 간결화 |
| `grid-area` 스택 | ~98% | 안정적 |

### 1.5 반응형 슬라이드

- **모바일 뷰포트 단위**: `svh`(최소 보장 높이)가 고정 레이아웃에 안정적
- **Container Queries**: 슬라이드 내부 콘텐츠를 슬라이드 크기에 반응하게 할 수 있음 (95% 지원)

---

## 2. 키보드 & 마우스 네비게이션

### 2.1 키보드 이벤트

- **`keydown`** 사용 (즉각 반응, keyup은 느림)
- **`event.key`** 사용 (의미 기반, 레이아웃 무관)
- **`event.repeat`** 차단 (auto-repeat 방지)
- **입력 요소 필터링**: `event.target.matches('input, textarea, select')` + `isContentEditable`

| 키 | event.key | 동작 |
|----|-----------|------|
| ←↑ / PageUp | `ArrowLeft`/`ArrowUp`/`PageUp` | 이전 슬라이드 |
| →↓ / Space / PageDown | `ArrowRight`/`ArrowDown`/`" "`/`PageDown` | 다음 슬라이드 |
| Shift+Space | `" "` + `shiftKey` | 이전 슬라이드 |
| Home / End | `Home`/`End` | 첫/마지막 슬라이드 |
| F | `f` | 풀스크린 토글 |
| ? | `?` | 도움말 토글 |
| Escape | `Escape` | 풀스크린 종료 / 오버뷰 탈출 |

### 2.2 URL Hash 동기화

**권장 패턴: Hash 기반 + replaceState 혼합**

- 일반 네비게이션(화살표): `history.replaceState()` -- 히스토리 오염 방지
- 명시적 이동(URL 입력, 링크): `pushState()` -- 뒤로가기 지원
- 초기 로드: `location.hash` 파싱 → 해당 슬라이드 이동
- `popstate` 이벤트 리스닝으로 뒤로가기 처리

### 2.3 터치/스와이프

**Pointer Events API 기본 (97%+ 지원) + Touch Events 폴백**

스와이프 감지 파라미터:
- **최소 거리 threshold**: 50px
- **수직 restraint**: 100px (수평 스와이프 시 수직 이동 제한)
- **시간 제한**: 300~500ms

터치 디바이스 감지:
- `navigator.maxTouchPoints > 0` + CSS `(hover: none) and (pointer: coarse)` 조합
- 양쪽 입력 모두 지원하는 것이 바람직 (2-in-1 기기 보편화)

### 2.4 진행 표시바 & 슬라이드 번호

- **Progress Bar**: `width: var(--progress)` + `transition: width 0.3s ease`
- **슬라이드 번호**: `3 / 15` 형식, 우하단 고정
- CSS 변수로 JS 로직과 스타일 분리

### 2.5 아키텍처 패턴

**단일 상태 관리**: 모든 입력(키보드, 터치, URL)이 `goToSlide(index)` 하나의 함수를 호출

| 모듈 | 역할 |
|------|------|
| SlideManager | currentSlide 상태, goToSlide/next/prev |
| KeyboardNav | keydown 리스너, 입력 필터링 |
| TouchNav | Pointer Events 스와이프 감지 |
| HashSync | replaceState/pushState, popstate |
| ProgressUI | progress bar, 슬라이드 번호 |

---

## 3. 슬라이드 전환 애니메이션

### 3.1 CSS Transition vs CSS Animation vs WAAPI

| 항목 | CSS Transition | CSS Animation | WAAPI |
|------|:--------------:|:-------------:|:-----:|
| 상태 수 | 2개 | 다수 (keyframes) | 다수 |
| 트리거 | class 변경 | 자동 or class | JS 호출 |
| 런타임 제어 | 거의 없음 | play-state 정도 | 완전 제어 |
| 완료 감지 | transitionend | animationend | finished Promise |
| 루프 | 불가 | 가능 | 가능 |

**권장**: CSS class toggle + transition 기본, WAAPI 보조 (고급 제어 필요 시)

### 3.2 전환 효과별 구현 원리

| 효과 | 핵심 CSS 속성 | GPU 가속 |
|------|--------------|:--------:|
| **Fade** | `opacity: 0→1` | O |
| **Slide** | `transform: translateX(±100%)→0` | O |
| **Zoom** | `transform: scale(0.8)→1` + opacity | O |
| **Flip** | `transform: rotateY(180deg)` + perspective + backface-visibility | O |

### 3.3 동시 표시 패턴 (이전/다음 슬라이드)

**패턴 A: z-index 스택킹** -- fade에 적합
- 들어오는 슬라이드 `z-index: 1`, 나가는 슬라이드 `z-index: 0`

**패턴 B: 동시 transform 이동** -- slide에 적합
- 들어옴: `translateX(100%)→0`, 나감: `translateX(0)→-100%` 동시 실행

**패턴 C: class 기반 상태 (reveal.js)** -- 범용
- `.past` / `.present` / `.future` 클래스로 상태 관리

### 3.4 GPU 가속

- **Compositor-only 속성**: `transform`과 `opacity`만 리플로우/리페인트 없이 처리
- **`will-change`**: 전환 직전 적용, 완료 후 제거 (메모리 효율)
- **사용 금지**: `width`, `height`, `top`, `left`, `margin`, `background-color` 애니메이션
- **모바일**: 동시 합성 레이어 2~3개로 제한 (GPU 메모리)

### 3.5 입력 잠금

**권장 조합**:
1. JS 플래그 (`isAnimating`) -- 키보드+마우스 모두 차단
2. `pointer-events: none` -- 마우스/터치 차단
3. `transitionend` + `setTimeout` 폴백 -- 이중 해제

### 3.6 View Transitions API (최신)

- Same-document 전환: 모든 주요 브라우저 지원 (Chrome 111+, Firefox 133+, Safari 18+)
- 브라우저가 스냅샷/애니메이션/입력차단 자동 처리
- `::view-transition-old` / `::view-transition-new` 의사 요소로 커스텀 가능
- 미지원 시 graceful degradation: `if (!document.startViewTransition) { updateFn(); return; }`

---

## 4. 콘텐츠 애니메이션 (빌드 효과)

### 4.1 Fragment Step 관리

**data-step 속성 패턴** (reveal.js fragment 시스템 단순화):

- `data-step="N"` 속성으로 등장 순서 지정
- 같은 step 값 = 동시 등장 (그룹핑)
- 슬라이드별 `maxStep` 미리 계산
- Space/클릭: `currentStep < maxStep` → step 증가, 아니면 다음 슬라이드
- Backspace/←: `currentStep > 0` → step 감소, 아니면 이전 슬라이드 마지막 step

**두 가지 모드**:
- **자동 순차 등장**: 슬라이드 진입 시 `transition-delay`로 시차 자동 등장 (현재 스킬 방식)
- **수동 step 진행**: Space/클릭으로 하나씩 노출 (reveal.js 기본 방식)
- 두 모드를 `data-auto-reveal` 속성으로 슬라이드별 선택 가능하게 하면 유연성 확보

### 4.2 등장 효과 종류

| 효과 | 초기 상태 | .visible 상태 |
|------|----------|--------------|
| fadeIn | `opacity: 0` | `opacity: 1` |
| slideUp | `opacity: 0; translateY(30px)` | `opacity: 1; translateY(0)` |
| slideLeft | `opacity: 0; translateX(-30px)` | `opacity: 1; translateX(0)` |
| scaleIn | `opacity: 0; scale(0.8)` | `opacity: 1; scale(1)` |
| blurIn | `opacity: 0; filter: blur(4px)` | `opacity: 1; filter: blur(0)` |

- `data-effect="fadeIn"` 속성으로 요소별 효과 지정
- 시차 등장: CSS `--i` 커스텀 프로퍼티 → `transition-delay: calc(var(--i) * 0.1s)`

### 4.3 @starting-style (최신 CSS)

- `display: none` → `display: block` 전환 시 애니메이션 가능
- `transition-behavior: allow-discrete` 필수 조합
- 브라우저 지원: Chrome 117+, Safari 17.5+, Firefox 129+ (~86%)
- fragment 요소를 `display: none`으로 완전히 숨겨두다가 등장 가능 (레이아웃 공간 미차지)

### 4.4 코드 블록 하이라이트

**방법 A: `<span class="line">` 래핑** (권장)
- 줄 단위로 span 래핑 → `.highlighted` 클래스 토글
- 비강조 줄 `opacity: 0.4` dimming
- `data-highlight-steps="1-3|4-6|7-9"` 파싱으로 step별 강조 이동

**방법 B: CSS linear-gradient 오버레이**
- `lh` 단위로 줄 위치에 반투명 밴드 생성
- 마크업 변경 불필요하지만 유연성 떨어짐

**방법 C: CSS Custom Highlight API**
- Range 객체 기반 텍스트 범위 스타일링
- 브라우저 지원 제한적 (아직 비권장)

### 4.5 화자 노트

**HTML**: `<aside class="speaker-notes">` (시맨틱) 또는 `data-notes="..."` 속성

**별도 윈도우 동기화 (우선순위)**:
1. **BroadcastChannel API** (권장) -- 양방향, 네이티브
2. **localStorage + storage 이벤트** -- 레거시 호환
3. **window.opener 직접 참조** -- 단순하지만 참조 깨질 수 있음

**인쇄 처리**:
```css
.speaker-notes { display: none; }
@media print { .speaker-notes { display: block; font-style: italic; } }
```

---

## 5. 기존 스킬 개선 영역

### 5.1 인쇄 지원 (난이도: 낮)

```css
@media print {
  @page { size: 960px 540px landscape; margin: 0; }
  .slide { break-before: page; opacity: 1 !important; visibility: visible !important; transform: none !important; }
  .fragment { opacity: 1 !important; transform: none !important; }
  .progress-bar, .slide-counter, .help-overlay { display: none; }
  .presentation, .slides { overflow: visible; height: auto; }
}
```

### 5.2 접근성 (난이도: 중)

**WAI-ARIA Carousel 패턴 적용**:
- `.slides` 컨테이너: `role="region"`, `aria-roledescription="carousel"`
- `.slide`: `role="group"`, `aria-roledescription="slide"`, `aria-label="N of M"`
- 비활성 슬라이드: `aria-hidden="true"`, `inert` 속성
- 슬라이드 전환 시 `focus()` 호출 (`tabindex="-1"`)
- `aria-live="polite"` (수동 조작 시) / `aria-live="off"` (자동 재생 시)
- `:focus-visible` 스타일 추가 (WCAG 2.4.13)

### 5.3 다크/라이트 테마 (난이도: 낮)

- CSS 변수 기반 이미 갖춰져 있음 → `[data-theme="light"]`에서 변수 재정의만 추가
- `prefers-color-scheme` 초기 감지 + `localStorage` 사용자 설정 우선
- `T` 키 단축키 토글
- `light-dark()` CSS 함수 활용 가능 (87% 지원)

### 5.4 슬라이드 오버뷰 (난이도: 중)

- `O` 키로 `.overview` 클래스 토글
- `.slides`를 `grid-area: 1/1` 스택에서 일반 그리드로 전환
- 클릭 시 해당 슬라이드로 이동 + 오버뷰 탈출
- `content-visibility: auto`로 대량 슬라이드 성능 최적화

### 5.5 프레젠터 모드 (난이도: 높)

- `S` 키 → `window.open` 별도 창
- 현재 슬라이드 미리보기 + 다음 슬라이드 + 노트 + 타이머
- BroadcastChannel + window.opener 병행 동기화

### 5.6 자동 재생 (난이도: 중)

- `setTimeout` 체이닝 (전환 완료 후 다음 타이머)
- `data-autoplay-interval` 슬라이드별 커스텀
- 사용자 조작 시 자동 일시정지 (WCAG 2.2.2)
- hover/focus 시 일시정지 (WAI-ARIA Carousel 패턴)

---

## 6. 현재 스킬 대비 개선 권장사항

### 현재 스킬의 강점 (유지)

| 항목 | 현재 구현 | 평가 |
|------|----------|------|
| 레이아웃 | Grid `grid-area: 1/1` + Flexbox | 최적 |
| 비율 고정 | 960x540 + `transform: scale()` | 최적 |
| 디자인 토큰 | CSS 변수 `:root` | 테마 확장 기반 확보 |
| Fragment | `data-step` + 자동 순차 등장 | 양호 |
| 네비게이션 | 키보드 + Pointer Events + URL hash | 양호 |
| 입력 잠금 | `isAnimating` + transitionend + setTimeout | 양호 |

### 개선이 필요한 영역

| 우선순위 | 영역 | 현재 상태 | 개선 방향 |
|:--------:|------|----------|----------|
| 1 | **접근성** | ARIA 속성 전무 | WAI-ARIA Carousel 패턴 적용 |
| 2 | **인쇄** | 미지원 | `@media print` 블록 추가 |
| 3 | **테마** | 다크 단일 | 라이트 테마 변수 + 토글 |
| 4 | **Fragment 모드** | 자동만 지원 | 수동 step 모드 추가 (선택적) |
| 5 | **코드 하이라이트** | 정적 구문 강조만 | step별 줄 하이라이트 이동 |
| 6 | **오버뷰** | 미지원 | Grid 토글 방식 |
| 7 | **View Transitions** | 미사용 | 폴백과 함께 progressive enhancement |
| 8 | **프레젠터 모드** | 기본 BroadcastChannel만 | 타이머 + 미리보기 보강 |

### 참고 소스 목록

- reveal.js: Layout, Markup, Keyboard, Touch, Transitions, Fragments, Speaker View, Overview
- impress.js: DOCUMENTATION.md
- MDN: CSS aspect-ratio, Container Queries, View Transitions API, Pointer Events, WAAPI, @starting-style
- W3C WAI-ARIA APG: Carousel Pattern
- Can I Use: aspect-ratio, container queries, viewport units, view-transitions, light-dark()
- web.dev: CSS animations guide, viewport units, entry animations
- Smashing Magazine: Accessible Carousels, Color Scheme Preferences
