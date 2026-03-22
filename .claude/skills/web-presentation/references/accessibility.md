# 접근성 상세 참조

## 목차
1. WAI-ARIA Carousel 패턴
2. ARIA 속성 매핑
3. 포커스 관리
4. 오버뷰 모드 접근성
5. 자동재생 접근성 (WCAG 2.2.2)
6. 인쇄 접근성

## 1. WAI-ARIA Carousel 패턴

웹 프레젠테이션은 WAI-ARIA Carousel 패턴을 따른다.

### 컨테이너 구조
- `<main class="presentation" role="region" aria-roledescription="carousel" aria-label="제목">`
- `<div class="slides" aria-live="polite">` — 자동재생 시 `aria-live="off"`로 전환

### 개별 슬라이드
- `role="group"` — 슬라이드를 논리적 그룹으로 표시
- `aria-roledescription="slide"` — 스크린 리더에 "slide"로 안내
- `aria-label="슬라이드 N / Total"` — 현재 위치
- `tabindex="-1"` — 프로그래밍 포커스 가능

## 2. ARIA 속성 매핑

### 활성 슬라이드
```html
<section class="slide active" id="slide-1"
         role="group" aria-roledescription="slide"
         aria-label="슬라이드 1 / 10" tabindex="-1">
```

### 비활성 슬라이드
```html
<section class="slide" id="slide-2"
         role="group" aria-roledescription="slide"
         aria-label="슬라이드 2 / 10" tabindex="-1"
         aria-hidden="true" inert>
```

- `aria-hidden="true"` — 스크린 리더에서 숨김
- `inert` — 포커스 및 클릭 이벤트 차단

### JS 토글 패턴
```javascript
// 슬라이드 전환 시
slides[oldIndex].setAttribute('aria-hidden', 'true');
slides[oldIndex].setAttribute('inert', '');

slides[newIndex].removeAttribute('aria-hidden');
slides[newIndex].removeAttribute('inert');
slides[newIndex].setAttribute('aria-label', '슬라이드 ' + (newIndex + 1) + ' / ' + total);
slides[newIndex].focus({ preventScroll: true });
```

## 3. 포커스 관리

### focus-visible 스타일
```css
.slide:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: -2px;
}
.slide:focus:not(:focus-visible) { outline: none; }
```

- 키보드 탐색 시에만 포커스 링 표시
- 마우스 클릭 시 포커스 링 숨김

### 슬라이드 전환 시 포커스
- `slides[currentSlide].focus({ preventScroll: true })` — 페이지 스크롤 없이 포커스 이동
- `tabindex="-1"` — Tab 순서에 포함되지 않으나 프로그래밍 포커스 가능

## 4. 오버뷰 모드 접근성

### 진입 시
- 모든 슬라이드에서 `inert`, `aria-hidden` 제거 (클릭/탭 가능하게)
- 슬라이드에 `cursor: pointer` 추가

### 탈출 시
- 비활성 슬라이드에 `inert`, `aria-hidden` 복원
- `cursor` 초기화

```javascript
if (isOverview) {
  slides.forEach(slide => {
    slide.removeAttribute('inert');
    slide.removeAttribute('aria-hidden');
  });
} else {
  slides.forEach((slide, i) => {
    if (i !== currentSlide) {
      slide.setAttribute('inert', '');
      slide.setAttribute('aria-hidden', 'true');
    }
  });
}
```

## 5. 자동재생 접근성 (WCAG 2.2.2)

WCAG 2.2.2 "Pause, Stop, Hide" 준수:

- `P` 키로 자동재생 일시정지/재개 가능
- 자동재생 중: `aria-live="off"` — 매 전환마다 알림 방지
- 일시정지/수동 시: `aria-live="polite"` — 사용자 요청 전환만 알림
- 시각 피드백: ⏸/▶ 아이콘 2초 표시 후 페이드아웃

## 6. 인쇄 접근성

```css
@media print {
  .slide[aria-hidden], .slide[inert] {
    opacity: 1 !important; visibility: visible !important;
  }
  .speaker-notes, .speaker-notes[hidden] {
    display: block !important;
  }
}
```

- 인쇄 시 모든 슬라이드 표시 (ARIA 상태 무시)
- `speaker-notes[hidden]` 오버라이드로 발표자 노트 포함
- UI 요소(progress bar, counter, help) 숨김
