# 고급 기능 상세 참조

## 목차
1. 프레젠터 모드 (S 키)
2. 슬라이드 오버뷰 (O 키)
3. 자동 재생 (P 키)
4. View Transitions API
5. 인라인 노트 패널 (N 키)
6. 수동 Fragment Step (data-manual)
7. 코드 줄 하이라이트 (data-highlight-steps)
8. 다크/라이트 테마 (T 키)

## 1. 프레젠터 모드

### 1.1 개요
- `S` 키로 별도 창 열기 (`window.open` 동기 호출)
- BroadcastChannel API로 양방향 동기화
- 현재/다음 슬라이드 미리보기, 발표자 노트, 타이머

### 1.2 BroadcastChannel 프로토콜

채널명: `'presenter-sync'`

**메인 → 프레젠터 메시지:**
```javascript
{
  type: 'slide-change',
  index: currentSlide,       // 현재 슬라이드 인덱스
  total: totalSlides,         // 전체 슬라이드 수
  notes: 'HTML 문자열',       // speaker-notes innerHTML
  slideHTML: 'HTML 문자열',   // 현재 슬라이드 outerHTML
  nextSlideHTML: 'HTML 문자열' // 다음 슬라이드 outerHTML
}
```

**프레젠터 → 메인 메시지:**
```javascript
{
  type: 'navigate',
  index: 1 // 또는 -1 (상대 이동)
}
```

### 1.3 프레젠터 창 구조
- 상단: 슬라이드 카운터 + 경과 시간 타이머 (HH:MM:SS)
- 중앙 좌: 현재 슬라이드 (큰 미리보기)
- 중앙 우 상: 다음 슬라이드 (작은 미리보기)
- 중앙 우 하: 발표자 노트
- 하단: 이전/다음 버튼

### 1.4 팝업 차단 처리
```javascript
const presWin = window.open('', 'presenter', 'width=1000,height=700');
if (!presWin) {
  alert('팝업이 차단되었습니다. 팝업을 허용해 주세요.');
  return;
}
```

## 2. 슬라이드 오버뷰

### 2.1 CSS 그리드 뷰
```css
.overview .slides {
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 16px; padding: 24px;
  overflow-y: auto; max-height: 100vh;
  transform: none !important;
}
```

### 2.2 성능 최적화
```css
.overview .slide {
  content-visibility: auto;
  contain-intrinsic-size: 240px 135px;
}
```
- `content-visibility: auto` — 뷰포트 밖 슬라이드 렌더링 지연
- `contain-intrinsic-size` — 레이아웃 시프트 방지

### 2.3 상호작용
- 슬라이드 클릭 시 해당 슬라이드로 이동 + 오버뷰 닫기
- 현재 슬라이드: `--color-primary` 테두리
- 호버: `--color-accent` 테두리
- Escape 키로 오버뷰 탈출

## 3. 자동 재생

### 3.1 HTML 설정
```html
<main class="presentation" data-autoplay="5000">
```
- `data-autoplay` — 밀리초 단위 전환 간격 (0이면 비활성)
- `data-autoslide` — 개별 슬라이드 재생 시간 오버라이드

### 3.2 동작 흐름
1. 페이지 로드 → `data-autoplay` 값 확인 → 타이머 시작
2. 사용자 키보드/스와이프 입력 → 타이머 리셋 + 재시작
3. `P` 키 → 일시정지/재개 토글
4. 마지막 슬라이드 → 자동 정지

### 3.3 시각 피드백
```javascript
function showAutoplayIndicator(icon) {
  autoplayIndicator.textContent = icon; // '⏸' 또는 '▶'
  autoplayIndicator.className = 'autoplay-indicator show';
  setTimeout(() => {
    autoplayIndicator.className = 'autoplay-indicator fade-out';
  }, 100);
}
```

## 4. View Transitions API

### 4.1 지원 감지
```javascript
if (document.startViewTransition && !isOverview) {
  document.body.classList.add('vt-active');
  const transition = document.startViewTransition(() => performTransition());
  transition.finished.then(() => {
    document.body.classList.remove('vt-active');
  });
} else {
  performTransition();
}
```

### 4.2 이중 애니메이션 방지
```css
.vt-active .slide { transition: none !important; }
```
View Transitions와 CSS transitions가 동시에 발생하면 이중 애니메이션이 되므로, `.vt-active` 클래스가 있는 동안 CSS transitions를 비활성화한다.

### 4.3 폴백
View Transitions API 미지원 브라우저에서는 기존 CSS transitions가 자동으로 작동한다.
오버뷰 모드에서는 그리드↔스택 전환 충돌 방지를 위해 항상 CSS transitions를 사용한다.

## 5. 인라인 노트 패널

### 5.1 HTML 구조
```html
<aside class="speaker-notes" hidden>슬라이드별 노트</aside>
<div class="notes-panel" id="notesPanel"></div>
```

### 5.2 동작
- `N` 키로 하단 패널 토글 (슬라이드 위에 오버레이)
- 슬라이드 전환 시 자동으로 현재 슬라이드의 `speaker-notes` 내용으로 갱신
- 노트 없는 슬라이드: "노트 없음" 표시

## 6. 수동 Fragment Step

### 6.1 HTML 사용법
```html
<section class="slide" data-manual>
  <div class="slide-content">
    <h2>제목</h2>
    <ul>
      <li class="fragment" data-step="1">첫 번째</li>
      <li class="fragment" data-step="2">두 번째</li>
      <li class="fragment" data-step="3">세 번째</li>
    </ul>
  </div>
</section>
```

### 6.2 동작 차이
| 모드 | 방향키 동작 |
|------|-----------|
| 자동 (기본) | 슬라이드 전환, 모든 fragment 자동 등장 |
| 수동 (data-manual) | step 단위 진행, 마지막 step 후 다음 슬라이드 |

### 6.3 역방향 탐색
- `currentStep > 0`이면 step 감소
- `currentStep === 0`이면 이전 슬라이드로 이동

## 7. 코드 줄 하이라이트

### 7.1 HTML 사용법
```html
<div class="code-block" data-highlight-steps="1-3|5|7-9">
<pre>
<span class="code-line" data-line="1">...</span>
<span class="code-line" data-line="2">...</span>
</pre>
</div>
```

### 7.2 step 파싱
`data-highlight-steps="1-3|5|7-9"` → 3개 step:
- Step 1: 줄 1, 2, 3 강조
- Step 2: 줄 5 강조
- Step 3: 줄 7, 8, 9 강조

### 7.3 시각 효과
- 강조 줄: 노란색 배경 + 왼쪽 골드 보더
- 비강조 줄: `opacity: 0.35`로 어둡게

## 8. 다크/라이트 테마

### 8.1 초기화 우선순위
1. `localStorage('web-pres-theme')` 값 (사용자 선택 기억)
2. `prefers-color-scheme: light` 미디어 쿼리 (시스템 설정)
3. 기본값: 다크 테마

### 8.2 CSS 변수 오버라이드
```css
[data-theme="light"] {
  --color-bg: #f5f5f5;
  --color-surface: #ffffff;
  --color-primary: #1565c0;
  /* ... */
}
```

### 8.3 코드 블록 예외
라이트 테마에서도 코드 블록은 다크 배경을 유지한다:
```css
[data-theme="light"] .code-block {
  --code-bg: #0d1117;
  background: var(--code-bg);
}
```
