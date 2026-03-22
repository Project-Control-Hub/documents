# 웹 프레젠테이션 상세 구현 참조

SKILL.md의 요약을 보완하는 전체 CSS/JS 코드와 고급 패턴을 담고 있다.

## 목차

1. [전체 CSS 코드](#1-전체-css-코드)
2. [전체 JS 엔진 코드](#2-전체-js-엔진-코드)
3. [도움말 오버레이 HTML](#3-도움말-오버레이-html)
4. [고급 패턴](#4-고급-패턴)

---

## 1. 전체 CSS 코드

```css
/* ─── 리셋 ─── */
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

/* ─── 웹폰트 ─── */
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700;900&family=Fira+Code:wght@400;500&display=swap');

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

/* ─── 라이트 테마 ─── */
[data-theme="light"] {
  --color-bg: #f5f5f5;
  --color-surface: #ffffff;
  --color-primary: #1565c0;
  --color-secondary: #7b1fa2;
  --color-accent: #e65100;
  --color-text: #212121;
  --color-text-dim: #757575;
  --color-success: #2e7d32;
  --color-danger: #c62828;
}
[data-theme="light"] .code-block {
  --code-bg: #0d1117;
  background: var(--code-bg);
}

html, body {
  width: 100%; height: 100%;
  overflow: hidden;
  background: var(--color-bg);
  font-family: var(--font-sans);
  color: var(--color-text);
}

/* ═══ 슬라이드 레이아웃 ═══ */

.presentation {
  width: 100vw; height: 100vh;
  display: grid; place-items: center;
  overflow: hidden;
}

.slides {
  width: var(--slide-width); height: var(--slide-height);
  position: relative; display: grid;
  transform: scale(var(--slide-scale, 1));
  transform-origin: center center;
}

.slide {
  grid-area: 1 / 1;
  width: 100%; height: 100%;
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  padding: 40px 56px;
  opacity: 0; visibility: hidden;
  transition: opacity 0.5s ease, transform 0.5s ease, visibility 0.5s;
  background: var(--color-surface);
  border-radius: 8px; overflow: hidden;
  transform: scale(0.95);
}

.slide.active {
  opacity: 1; visibility: visible;
  z-index: 1; transform: scale(1);
}

.slide-content { width: 100%; max-height: 100%; }

/* ═══ 타이포그래피 ═══ */

.slide h1 { font-size: 42px; font-weight: 900; color: #fff; margin-bottom: 12px; line-height: 1.3; }
.slide h2 { font-size: 30px; font-weight: 700; color: var(--color-primary); margin-bottom: 18px; line-height: 1.3; }
.slide h3 { font-size: 20px; font-weight: 500; color: var(--color-accent); margin-bottom: 12px; }
.slide p  { font-size: 18px; line-height: 1.6; color: var(--color-text); margin-bottom: 10px; }
.slide .subtitle { font-size: 22px; color: var(--color-text-dim); font-weight: 300; }

.slide ul, .slide ol { font-size: 18px; line-height: 1.7; padding-left: 24px; margin-bottom: 10px; }
.slide li { margin-bottom: 4px; }

.slide strong { color: var(--color-primary); }
.slide em { color: var(--color-accent); font-style: normal; }

/* ─── 태그 ─── */
.tag { display: inline-block; padding: 3px 10px; border-radius: 12px; font-size: 13px; font-weight: 500; margin: 2px 4px; }
.tag-blue   { background: rgba(79,195,247,0.2); color: var(--color-primary); }
.tag-purple { background: rgba(224,64,251,0.2); color: var(--color-secondary); }
.tag-yellow { background: rgba(255,215,64,0.2); color: var(--color-accent); }
.tag-green  { background: rgba(102,187,106,0.2); color: var(--color-success); }
.tag-red    { background: rgba(239,83,80,0.2); color: var(--color-danger); }

/* ─── 타이틀 슬라이드 ─── */
.slide.title-slide {
  text-align: center;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
}
.slide.title-slide h1 {
  font-size: 48px;
  background: linear-gradient(135deg, var(--color-primary), var(--color-secondary));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}
.slide.title-slide .subtitle { font-size: 24px; margin-top: 8px; }

/* ─── 섹션 슬라이드 ─── */
.slide.section-slide {
  text-align: center;
  background: linear-gradient(135deg, #0f3460 0%, #1a1a2e 100%);
}
.slide.section-slide h2 { font-size: 36px; color: #fff; }

/* ═══ 테이블 ═══ */

.slide table { width: 100%; border-collapse: collapse; font-size: 15px; margin: 10px 0; }
.slide th { background: rgba(79,195,247,0.15); color: var(--color-primary); padding: 8px 12px; text-align: left; font-weight: 500; border-bottom: 2px solid rgba(79,195,247,0.3); }
.slide td { padding: 6px 12px; border-bottom: 1px solid rgba(255,255,255,0.08); }

/* ═══ 코드 블록 ═══ */

.code-block { background: var(--code-bg, #0d1117); border-radius: 8px; padding: 14px 18px; margin: 10px 0; overflow-x: auto; border: 1px solid rgba(255,255,255,0.08); }
.code-block pre { margin: 0; font-family: var(--font-mono); font-size: 14px; line-height: 1.55; color: #d4d4d4; }
.code-block .comment { color: #6a9955; }
.code-block .keyword { color: #c586c0; }
.code-block .string  { color: #ce9178; }
.code-block .func    { color: #dcdcaa; }
.code-block .type    { color: #4ec9b0; }
.code-block .number  { color: #b5cea8; }
.code-block .attr    { color: #9cdcfe; }

.slide code:not(pre code) { font-family: var(--font-mono); font-size: 0.85em; background: rgba(79,195,247,0.12); color: var(--color-primary); padding: 2px 8px; border-radius: 4px; }

/* ─── 코드 줄 하이라이트 ─── */
.code-line {
  display: block; padding: 0 8px;
  transition: background-color 0.3s ease, opacity 0.3s ease;
}
.code-line.highlighted {
  background-color: rgba(255, 255, 100, 0.15);
  border-left: 3px solid var(--color-accent);
  padding-left: 5px;
}
.code-block.has-highlight .code-line:not(.highlighted) { opacity: 0.35; }

/* ═══ 다이어그램 / 시각 요소 ═══ */

.diagram { display: flex; align-items: center; justify-content: center; gap: 16px; margin: 16px 0; flex-wrap: wrap; }
.diagram-box { background: rgba(79,195,247,0.08); border: 2px solid rgba(79,195,247,0.3); border-radius: 12px; padding: 12px 20px; text-align: center; min-width: 120px; transition: all 0.3s ease; }
.diagram-box.highlight { border-color: var(--color-primary); background: rgba(79,195,247,0.15); box-shadow: 0 0 20px rgba(79,195,247,0.2); }
.diagram-box h4 { font-size: 15px; color: var(--color-primary); margin-bottom: 4px; }
.diagram-box p  { font-size: 13px; color: var(--color-text-dim); margin: 0; }
.diagram-arrow { font-size: 22px; color: var(--color-text-dim); }

.two-columns { display: grid; grid-template-columns: 1fr 1fr; gap: 28px; align-items: start; }

.callout { background: rgba(79,195,247,0.08); border-left: 4px solid var(--color-primary); padding: 12px 18px; border-radius: 0 8px 8px 0; margin: 10px 0; font-size: 16px; }
.callout.warning { border-left-color: var(--color-accent); background: rgba(255,215,64,0.08); }
.callout.danger  { border-left-color: var(--color-danger); background: rgba(239,83,80,0.08); }

.compare-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; margin: 12px 0; }
.compare-card { background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.1); border-radius: 12px; padding: 16px; }
.compare-card h4 { font-size: 16px; margin-bottom: 8px; }
.compare-card.good { border-color: rgba(102,187,106,0.4); }
.compare-card.good h4 { color: var(--color-success); }
.compare-card.bad  { border-color: rgba(239,83,80,0.4); }
.compare-card.bad h4  { color: var(--color-danger); }

/* ═══ 프래그먼트 ═══ */

.fragment {
  opacity: 0; transform: translateY(16px);
  transition: opacity 0.4s ease, transform 0.4s ease;
  transition-delay: var(--step-delay, 0s);
}
.fragment.visible { opacity: 1; transform: translateY(0); }

/* ═══ 오버뷰 모드 ═══ */
.overview .slides {
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 16px; padding: 24px;
  overflow-y: auto; max-height: 100vh;
  transform: none !important;
}
.overview .slide {
  grid-area: auto;
  opacity: 1; visibility: visible;
  transform: none !important; cursor: pointer;
  border: 2px solid transparent;
  transition: border-color 0.2s ease;
  content-visibility: auto;
  contain-intrinsic-size: 240px 135px;
}
.overview .slide.active { border-color: var(--color-primary); }
.overview .slide:hover { border-color: var(--color-accent); }
.overview .progress-bar,
.overview .slide-counter { display: none; }

/* ═══ UI 오버레이 ═══ */

.progress-bar { position: fixed; bottom: 0; left: 0; width: 100%; height: 4px; background: rgba(255,255,255,0.1); z-index: 100; }
.progress-fill { height: 100%; width: 0%; background: linear-gradient(90deg, var(--color-primary), var(--color-secondary)); transition: width 0.3s ease-out; }
.slide-counter { position: fixed; bottom: 14px; right: 24px; font-size: 14px; font-family: var(--font-mono); color: rgba(255,255,255,0.4); z-index: 100; user-select: none; }

.help-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.85); z-index: 200; display: none; place-items: center; }
.help-overlay.show { display: grid; }
.help-content { background: var(--color-surface); border-radius: 16px; padding: 32px 40px; max-width: 460px; width: 90%; }
.help-content h3 { color: var(--color-primary); font-size: 22px; margin-bottom: 16px; }
.help-content table { width: 100%; font-size: 15px; }
.help-content td { padding: 5px 0; border: none; }
.help-content kbd { display: inline-block; background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2); border-radius: 4px; padding: 2px 8px; font-family: var(--font-mono); font-size: 13px; min-width: 28px; text-align: center; }

.hide-cursor { cursor: none; }

/* ═══ 접근성 ═══ */
.slide:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: -2px;
}
.slide:focus:not(:focus-visible) { outline: none; }

/* ─── 인라인 노트 패널 ─── */
.notes-panel {
  position: fixed; bottom: 0; left: 0; right: 0;
  max-height: 30vh; overflow-y: auto;
  background: rgba(0, 0, 0, 0.9); color: var(--color-text);
  padding: 16px 24px; font-size: 15px; line-height: 1.6;
  transform: translateY(100%); transition: transform 0.3s ease;
  z-index: 150; border-top: 1px solid rgba(255,255,255,0.1);
}
.notes-panel.show { transform: translateY(0); }
.notes-panel .notes-empty { color: var(--color-text-dim); font-style: italic; }

/* ═══ View Transitions ═══ */
.vt-active .slide { transition: none !important; }
::view-transition-old(root) { animation: vt-fade-out 0.3s ease; }
::view-transition-new(root) { animation: vt-fade-in 0.3s ease; }
@keyframes vt-fade-out { from { opacity: 1; } to { opacity: 0; } }
@keyframes vt-fade-in  { from { opacity: 0; } to { opacity: 1; } }

/* ─── 자동재생 표시 ─── */
.autoplay-indicator {
  position: fixed; bottom: 14px; left: 24px;
  font-size: 20px; color: rgba(255,255,255,0.6);
  z-index: 100; opacity: 0;
  transition: opacity 0.3s ease;
}
.autoplay-indicator.show { opacity: 1; }
.autoplay-indicator.fade-out { animation: fadeOutIndicator 2s ease forwards; }
@keyframes fadeOutIndicator {
  0% { opacity: 1; } 70% { opacity: 1; } 100% { opacity: 0; }
}

/* ═══ 인쇄 지원 ═══ */
@media print {
  @page { size: 960px 540px landscape; margin: 0; }
  .presentation, .slides {
    display: block; overflow: visible; height: auto;
    transform: none !important;
  }
  .slide {
    break-before: page;
    opacity: 1 !important; visibility: visible !important;
    transform: none !important; position: relative;
    display: flex; width: 960px; height: 540px;
  }
  .slide[aria-hidden], .slide[inert] {
    opacity: 1 !important; visibility: visible !important;
  }
  .fragment { opacity: 1 !important; transform: none !important; }
  .progress-bar, .slide-counter, .help-overlay,
  .notes-panel, .autoplay-indicator { display: none !important; }
  .speaker-notes, .speaker-notes[hidden] {
    display: block !important; border-top: 1px solid #ccc;
    font-style: italic; padding: 8px 0;
  }
}
```

---

## 2. 전체 JS 엔진 코드

```javascript
(function () {
  'use strict';

  // ═══ 전역 상태 & DOM 참조 ═══

  const slides = document.querySelectorAll('.slide');
  const totalSlides = slides.length;
  let currentSlide = 0;
  let isAnimating = false;

  const progressFill = document.getElementById('progressFill');
  const counterCurrent = document.getElementById('counterCurrent');
  const counterTotal = document.getElementById('counterTotal');
  const helpOverlay = document.getElementById('helpOverlay');
  const notesPanel = document.getElementById('notesPanel');
  const autoplayIndicator = document.getElementById('autoplayIndicator');
  const presentation = document.querySelector('.presentation');
  const slidesContainer = document.querySelector('.slides');
  const presenterChannel = new BroadcastChannel('presenter-sync');

  counterTotal.textContent = totalSlides;

  const STEP_DELAY = 0.2; // 자동 등장 딜레이 (초)

  // ── 모듈 간 공유 상태 변수 ──
  let isManualMode = false;
  let currentStep = 0;
  let maxStep = 0;
  let isOverview = false;
  let isNotesVisible = false;
  const autoplayInterval = parseInt(presentation?.dataset.autoplay) || 0;
  let autoplayTimer = null;
  let isAutoplayPaused = false;

  // ═══ 1. 뷰포트 스케일링 ═══

  function fitSlides() {
    const container = document.querySelector('.presentation');
    const slidesEl = document.querySelector('.slides');
    const W = 960, H = 540, MARGIN = 0.04;

    const aw = container.clientWidth  * (1 - MARGIN * 2);
    const ah = container.clientHeight * (1 - MARGIN * 2);
    const scale = Math.min(aw / W, ah / H);

    slidesEl.style.setProperty('--slide-scale', scale);
  }

  window.addEventListener('resize', fitSlides);
  fitSlides();

  // ═══ 2. 프래그먼트 — 자동 순차 등장 ═══

  function autoRevealFragments(slide) {
    const fragments = slide.querySelectorAll('.fragment');
    fragments.forEach(f => {
      const step = parseInt(f.dataset.step) || 1;
      f.style.setProperty('--step-delay', (step - 1) * STEP_DELAY + 's');
      f.classList.add('visible');
    });
  }

  function resetFragments(slide) {
    const fragments = slide.querySelectorAll('.fragment');
    fragments.forEach(f => {
      f.style.setProperty('--step-delay', '0s');
      f.classList.remove('visible');
    });
  }

  // 초기 상태: 첫 슬라이드만 자동 등장
  slides.forEach((slide, i) => {
    if (i === 0) autoRevealFragments(slide);
    else resetFragments(slide);
  });

  // ═══ 3. 슬라이드 전환 ═══

  function revealStep(step) {
    const slide = slides[currentSlide];
    const fragments = slide.querySelectorAll('.fragment');
    fragments.forEach(f => {
      const s = parseInt(f.dataset.step) || 1;
      if (s <= step) {
        f.style.setProperty('--step-delay', '0s');
        f.classList.add('visible');
      } else {
        f.classList.remove('visible');
      }
    });
  }

  function initCodeHighlights(slide) {
    slide.querySelectorAll('.code-block[data-highlight-steps]').forEach(block => {
      const steps = parseHighlightSteps(block.dataset.highlightSteps);
      if (steps.length > 0) applyHighlight(block, steps[0]);
    });
  }

  function showSlide(index) {
    if (index < 0 || index >= totalSlides) return;
    if (isAnimating && !isOverview) return;

    const oldIndex = currentSlide;

    function performTransition() {
      isAnimating = true;

      // ARIA: 이전 슬라이드 숨김
      slides[oldIndex].setAttribute('aria-hidden', 'true');
      slides[oldIndex].setAttribute('inert', '');
      resetFragments(slides[oldIndex]);
      slides[oldIndex].classList.remove('active');

      currentSlide = index;

      // ARIA: 새 슬라이드 노출
      slides[currentSlide].removeAttribute('aria-hidden');
      slides[currentSlide].removeAttribute('inert');
      slides[currentSlide].setAttribute('aria-label',
        '슬라이드 ' + (currentSlide + 1) + ' / ' + totalSlides);
      slides[currentSlide].classList.add('active');
      slides[currentSlide].focus({ preventScroll: true });

      // 수동 모드 또는 자동 등장
      if (slides[currentSlide].hasAttribute('data-manual')) {
        isManualMode = true;
        currentStep = 0;
        const fragments = slides[currentSlide].querySelectorAll('.fragment');
        maxStep = 0;
        fragments.forEach(f => {
          const s = parseInt(f.dataset.step) || 1;
          if (s > maxStep) maxStep = s;
        });
        revealStep(0);
      } else {
        isManualMode = false;
        currentStep = 0;
        maxStep = 0;
        setTimeout(() => {
          autoRevealFragments(slides[currentSlide]);
        }, 150);
      }

      // 코드 하이라이트 초기화
      initCodeHighlights(slides[currentSlide]);

      history.replaceState(null, '', '#slide-' + (currentSlide + 1));
      updateProgress();
      updateNotesContent();
      syncPresenter();

      slides[currentSlide].addEventListener('transitionend', function handler(e) {
        if (e.propertyName === 'opacity') {
          isAnimating = false;
          slides[currentSlide].removeEventListener('transitionend', handler);
        }
      });
      setTimeout(() => { isAnimating = false; }, 600);
    }

    // View Transitions API 통합
    if (!isOverview && document.startViewTransition) {
      document.body.classList.add('vt-active');
      const transition = document.startViewTransition(() => performTransition());
      transition.finished.then(() => {
        document.body.classList.remove('vt-active');
      });
    } else {
      performTransition();
    }
  }

  // ═══ 4. 키보드 네비게이션 ═══

  document.addEventListener('keydown', (e) => {
    if (e.target.matches('input, textarea, select') || e.target.isContentEditable) return;
    if (e.repeat) return;

    if (helpOverlay.classList.contains('show')) {
      if (e.key === 'Escape' || e.key === '?') {
        helpOverlay.classList.remove('show');
        e.preventDefault();
      }
      return;
    }

    switch (e.key) {
      case 'ArrowRight': case 'ArrowDown': case ' ': case 'PageDown':
        e.preventDefault();
        if (isManualMode && currentStep < maxStep) {
          currentStep++;
          revealStep(currentStep);
        } else {
          onUserInput();
          showSlide(currentSlide + 1);
        }
        break;
      case 'ArrowLeft': case 'ArrowUp': case 'PageUp':
        e.preventDefault();
        if (isManualMode && currentStep > 0) {
          currentStep--;
          revealStep(currentStep);
        } else {
          onUserInput();
          showSlide(currentSlide - 1);
        }
        break;
      case 'Home':
        e.preventDefault(); onUserInput(); showSlide(0); break;
      case 'End':
        e.preventDefault(); onUserInput(); showSlide(totalSlides - 1); break;
      case 'f': case 'F':
        e.preventDefault(); toggleFullscreen(); break;
      case 't': case 'T':
        e.preventDefault(); toggleTheme(); break;
      case 'o': case 'O':
        e.preventDefault(); toggleOverview(); break;
      case 's': case 'S':
        e.preventDefault(); openPresenterView(); break;
      case 'p': case 'P':
        e.preventDefault(); toggleAutoplay(); break;
      case 'n': case 'N':
        e.preventDefault(); toggleNotes(); break;
      case '?':
        e.preventDefault(); helpOverlay.classList.toggle('show'); break;
      case 'Escape':
        if (helpOverlay.classList.contains('show')) {
          helpOverlay.classList.remove('show');
        } else if (isOverview) {
          toggleOverview();
        } else if (document.fullscreenElement) {
          document.exitFullscreen();
        }
        break;
    }
  });

  // ═══ 5. 터치 / 스와이프 ═══

  let pointerStartX = 0, pointerStartY = 0;
  const SWIPE_THRESHOLD = 50, SWIPE_RESTRAINT = 100;

  document.addEventListener('pointerdown', (e) => {
    pointerStartX = e.clientX;
    pointerStartY = e.clientY;
  });

  document.addEventListener('pointerup', (e) => {
    const diffX = e.clientX - pointerStartX;
    const diffY = e.clientY - pointerStartY;
    if (Math.abs(diffX) > SWIPE_THRESHOLD && Math.abs(diffY) < SWIPE_RESTRAINT) {
      onUserInput();
      diffX < 0 ? showSlide(currentSlide + 1) : showSlide(currentSlide - 1);
    }
  });

  // ═══ 6. URL Hash 복원 ═══

  function restoreFromHash() {
    const match = location.hash.match(/^#slide-(\d+)$/);
    if (match) {
      const index = parseInt(match[1]) - 1;
      if (index >= 0 && index < totalSlides) showSlide(index);
    }
  }

  window.addEventListener('hashchange', restoreFromHash);
  restoreFromHash();

  // ═══ 7. 프로그레스 바 ═══

  function updateProgress() {
    const pct = totalSlides > 1 ? (currentSlide / (totalSlides - 1)) * 100 : 100;
    progressFill.style.width = pct + '%';
    counterCurrent.textContent = currentSlide + 1;
  }
  updateProgress();

  // ═══ 8. 풀스크린 ═══

  function toggleFullscreen() {
    if (!document.fullscreenElement) {
      document.documentElement.requestFullscreen().catch(() => {});
    } else {
      document.exitFullscreen();
    }
  }

  let cursorTimer = null;
  document.addEventListener('fullscreenchange', () => {
    if (document.fullscreenElement) {
      cursorTimer = setTimeout(() => document.body.classList.add('hide-cursor'), 2000);
    } else {
      clearTimeout(cursorTimer);
      document.body.classList.remove('hide-cursor');
    }
  });

  document.addEventListener('mousemove', () => {
    if (document.fullscreenElement) {
      document.body.classList.remove('hide-cursor');
      clearTimeout(cursorTimer);
      cursorTimer = setTimeout(() => document.body.classList.add('hide-cursor'), 2000);
    }
  });

  // ═══ 9. ARIA 접근성 ═══

  slides.forEach((slide, i) => {
    slide.setAttribute('role', 'group');
    slide.setAttribute('aria-roledescription', 'slide');
    slide.setAttribute('aria-label', '슬라이드 ' + (i + 1) + ' / ' + totalSlides);
    slide.setAttribute('tabindex', '-1');
    if (i !== 0) {
      slide.setAttribute('aria-hidden', 'true');
      slide.setAttribute('inert', '');
    }
  });

  // ═══ 10. 코드 줄 하이라이트 ═══

  function parseHighlightSteps(attr) {
    if (!attr) return [];
    return attr.split('|').map(group =>
      group.split(',').flatMap(range => {
        const [start, end] = range.trim().split('-').map(Number);
        if (end) return Array.from({ length: end - start + 1 }, (_, i) => start + i);
        return [start];
      })
    );
  }

  function applyHighlight(codeBlock, lines) {
    codeBlock.classList.toggle('has-highlight', lines.length > 0);
    codeBlock.querySelectorAll('.code-line').forEach(el => {
      const lineNum = parseInt(el.dataset.line);
      el.classList.toggle('highlighted', lines.includes(lineNum));
    });
  }

  // ═══ 11. 다크/라이트 테마 ═══

  function initTheme() {
    const saved = localStorage.getItem('web-pres-theme');
    if (saved) {
      document.documentElement.dataset.theme = saved;
    } else if (window.matchMedia('(prefers-color-scheme: light)').matches) {
      document.documentElement.dataset.theme = 'light';
    }
  }

  function toggleTheme() {
    const current = document.documentElement.dataset.theme;
    const next = current === 'light' ? 'dark' : 'light';
    document.documentElement.dataset.theme = next;
    localStorage.setItem('web-pres-theme', next);
  }

  initTheme();

  // ═══ 12. 슬라이드 오버뷰 ═══

  function toggleOverview() {
    isOverview = !isOverview;
    document.body.classList.toggle('overview', isOverview);
    if (isOverview) {
      slides.forEach(slide => {
        slide.removeAttribute('inert');
        slide.removeAttribute('aria-hidden');
        slide.style.cursor = 'pointer';
      });
    } else {
      slides.forEach((slide, i) => {
        if (i !== currentSlide) {
          slide.setAttribute('inert', '');
          slide.setAttribute('aria-hidden', 'true');
        }
        slide.style.cursor = '';
      });
    }
  }

  document.querySelector('.slides').addEventListener('click', (e) => {
    if (!isOverview) return;
    const clickedSlide = e.target.closest('.slide');
    if (!clickedSlide) return;
    const idx = Array.from(slides).indexOf(clickedSlide);
    if (idx !== -1) {
      toggleOverview();
      showSlide(idx);
    }
  });

  // ═══ 13. 인라인 노트 패널 ═══

  function toggleNotes() {
    isNotesVisible = !isNotesVisible;
    notesPanel.classList.toggle('show', isNotesVisible);
    if (isNotesVisible) updateNotesContent();
  }

  function updateNotesContent() {
    if (!notesPanel || !isNotesVisible) return;
    const notes = slides[currentSlide].querySelector('.speaker-notes');
    notesPanel.innerHTML = notes
      ? notes.innerHTML
      : '<span class="notes-empty">노트 없음</span>';
  }

  // ═══ 14. 자동 재생 ═══

  function startAutoplay() {
    if (!autoplayInterval || isAutoplayPaused) return;
    const slideInterval = parseInt(slides[currentSlide]?.dataset.autoslide) || autoplayInterval;
    autoplayTimer = setTimeout(() => {
      if (currentSlide < totalSlides - 1) {
        showSlide(currentSlide + 1);
        startAutoplay();
      }
    }, slideInterval);
    slidesContainer.setAttribute('aria-live', 'off');
  }

  function stopAutoplay() {
    clearTimeout(autoplayTimer);
    autoplayTimer = null;
    slidesContainer.setAttribute('aria-live', 'polite');
  }

  function toggleAutoplay() {
    isAutoplayPaused = !isAutoplayPaused;
    if (isAutoplayPaused) {
      stopAutoplay();
      showAutoplayIndicator('\u23F8');
    } else {
      startAutoplay();
      showAutoplayIndicator('\u25B6');
    }
  }

  function showAutoplayIndicator(icon) {
    if (!autoplayIndicator) return;
    autoplayIndicator.textContent = icon;
    autoplayIndicator.className = 'autoplay-indicator show';
    setTimeout(() => { autoplayIndicator.className = 'autoplay-indicator fade-out'; }, 100);
  }

  function onUserInput() {
    if (autoplayInterval && !isAutoplayPaused) {
      stopAutoplay();
      setTimeout(startAutoplay, 600);
    }
  }

  if (autoplayInterval) startAutoplay();

  // ═══ 15. 프레젠터 모드 ═══

  function buildPresenterHTML(styleContent) {
    return '<!DOCTYPE html><html lang="ko"><head><meta charset="UTF-8">' +
      '<title>Presenter View</title>' +
      '<style>' + styleContent + '\n' +
      'body { display: grid; grid-template-columns: 2fr 1fr; grid-template-rows: auto 1fr auto; gap: 12px; padding: 16px; height: 100vh; background: #1a1a2e; color: #e0e0e0; font-family: sans-serif; }' +
      '.pres-header { grid-column: 1 / -1; display: flex; justify-content: space-between; align-items: center; padding: 8px 0; border-bottom: 1px solid rgba(255,255,255,0.1); }' +
      '.pres-timer { font-size: 28px; font-family: monospace; color: #4fc3f7; }' +
      '.pres-counter { font-size: 18px; color: rgba(255,255,255,0.6); }' +
      '.pres-current { border: 2px solid #4fc3f7; border-radius: 8px; overflow: hidden; display: grid; place-items: center; }' +
      '.pres-current .slide { position: relative; opacity: 1; visibility: visible; transform: none; }' +
      '.pres-sidebar { display: flex; flex-direction: column; gap: 12px; }' +
      '.pres-next { border: 1px solid rgba(255,255,255,0.2); border-radius: 8px; overflow: hidden; flex: 0 0 auto; max-height: 40%; display: grid; place-items: center; }' +
      '.pres-next .slide { position: relative; opacity: 1; visibility: visible; transform: none; font-size: 10px; }' +
      '.pres-next-label { font-size: 12px; color: rgba(255,255,255,0.4); margin-bottom: 4px; }' +
      '.pres-notes { flex: 1; overflow-y: auto; background: rgba(0,0,0,0.3); border-radius: 8px; padding: 16px; font-size: 15px; line-height: 1.6; }' +
      '.pres-notes-label { font-size: 12px; color: rgba(255,255,255,0.4); margin-bottom: 4px; }' +
      '.pres-controls { grid-column: 1 / -1; display: flex; gap: 8px; justify-content: center; padding: 8px 0; border-top: 1px solid rgba(255,255,255,0.1); }' +
      '.pres-controls button { padding: 6px 16px; border: 1px solid rgba(255,255,255,0.2); background: rgba(255,255,255,0.05); color: #e0e0e0; border-radius: 4px; cursor: pointer; font-size: 14px; }' +
      '.pres-controls button:hover { background: rgba(255,255,255,0.1); }' +
      '</style></head><body>' +
      '<div class="pres-header"><span class="pres-counter" id="presCounter">1 / 1</span><span class="pres-timer" id="presTimer">00:00:00</span></div>' +
      '<div class="pres-current" id="presCurrent"></div>' +
      '<div class="pres-sidebar">' +
      '<div><div class="pres-next-label">다음 슬라이드</div><div class="pres-next" id="presNext"></div></div>' +
      '<div style="flex:1;display:flex;flex-direction:column;"><div class="pres-notes-label">발표자 노트</div><div class="pres-notes" id="presNotes"></div></div>' +
      '</div>' +
      '<div class="pres-controls">' +
      '<button onclick="navigate(-1)">\u25C0 이전</button>' +
      '<button onclick="navigate(1)">다음 \u25B6</button>' +
      '</div>' +
      '<script>' +
      'const ch = new BroadcastChannel("presenter-sync");' +
      'const startTime = Date.now();' +
      'setInterval(() => {' +
      '  const elapsed = Math.floor((Date.now() - startTime) / 1000);' +
      '  const h = String(Math.floor(elapsed / 3600)).padStart(2, "0");' +
      '  const m = String(Math.floor((elapsed % 3600) / 60)).padStart(2, "0");' +
      '  const s = String(elapsed % 60).padStart(2, "0");' +
      '  document.getElementById("presTimer").textContent = h + ":" + m + ":" + s;' +
      '}, 1000);' +
      'ch.onmessage = (e) => {' +
      '  if (e.data.type === "slide-change") {' +
      '    document.getElementById("presCounter").textContent = (e.data.index + 1) + " / " + e.data.total;' +
      '    document.getElementById("presCurrent").innerHTML = e.data.slideHTML;' +
      '    document.getElementById("presNext").innerHTML = e.data.nextSlideHTML || "<div style=\\"padding:20px;color:rgba(255,255,255,0.3)\\">마지막 슬라이드</div>";' +
      '    document.getElementById("presNotes").innerHTML = e.data.notes || "<span style=\\"color:rgba(255,255,255,0.3);font-style:italic\\">노트 없음</span>";' +
      '  }' +
      '};' +
      'function navigate(dir) { ch.postMessage({ type: "navigate", index: dir }); }' +
      '<\/script></body></html>';
  }

  function openPresenterView() {
    const presWin = window.open('', 'presenter', 'width=1000,height=700');
    if (!presWin) { alert('팝업이 차단되었습니다. 팝업을 허용해 주세요.'); return; }
    const styleContent = document.querySelector('style').textContent;
    presWin.document.write(buildPresenterHTML(styleContent));
    presWin.document.close();
    syncPresenter();
  }

  function syncPresenter() {
    const notes = slides[currentSlide].querySelector('.speaker-notes');
    presenterChannel.postMessage({
      type: 'slide-change',
      index: currentSlide,
      total: totalSlides,
      notes: notes ? notes.innerHTML : '',
      slideHTML: slides[currentSlide].outerHTML,
      nextSlideHTML: currentSlide < totalSlides - 1 ? slides[currentSlide + 1].outerHTML : ''
    });
  }

  presenterChannel.onmessage = (e) => {
    if (e.data.type === 'navigate') {
      const target = currentSlide + e.data.index;
      if (target >= 0 && target < totalSlides) showSlide(target);
    }
  };

})();
```

---

## 3. 도움말 오버레이 HTML

```html
<div class="help-overlay" id="helpOverlay">
  <div class="help-content">
    <h3>키보드 단축키</h3>
    <table>
      <tr><td><kbd>&rarr;</kbd> <kbd>Space</kbd></td><td>다음 슬라이드</td></tr>
      <tr><td><kbd>&larr;</kbd></td><td>이전 슬라이드</td></tr>
      <tr><td><kbd>Home</kbd></td><td>첫 슬라이드</td></tr>
      <tr><td><kbd>End</kbd></td><td>마지막 슬라이드</td></tr>
      <tr><td><kbd>F</kbd></td><td>풀스크린 토글</td></tr>
      <tr><td><kbd>T</kbd></td><td>테마 전환</td></tr>
      <tr><td><kbd>O</kbd></td><td>슬라이드 오버뷰</td></tr>
      <tr><td><kbd>S</kbd></td><td>발표자 뷰</td></tr>
      <tr><td><kbd>P</kbd></td><td>자동 재생 일시정지/재개</td></tr>
      <tr><td><kbd>N</kbd></td><td>노트 패널</td></tr>
      <tr><td><kbd>?</kbd></td><td>이 도움말 토글</td></tr>
      <tr><td><kbd>Esc</kbd></td><td>도움말/오버뷰/풀스크린 닫기</td></tr>
    </table>
    <p style="margin-top: 14px; font-size: 13px; color: var(--color-text-dim);">
      터치 기기: 좌/우 스와이프로 네비게이션 | <code>data-manual</code> 슬라이드에서는 방향키가 step 단위
    </p>
  </div>
</div>
```

---

## 4. 고급 패턴

### 4.1 다양한 Fragment 효과

```css
/* slide-up */
.fragment.slide-up { transform: translateY(40px); }
.fragment.slide-up.visible { transform: translateY(0); }

/* grow */
.fragment.grow { transform: scale(0.5); }
.fragment.grow.visible { transform: scale(1); }

/* blur-in */
.fragment.blur-in { filter: blur(8px); }
.fragment.blur-in.visible { filter: blur(0); }
```

### 4.2 슬라이드 전환 효과 변형

기본은 fade + scale이며, 필요시 다른 전환 효과를 적용할 수 있다.

**Slide (좌우 이동)**:
```css
.slide { transform: translateX(100%); transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1); }
.slide.active { transform: translateX(0); z-index: 1; }
.slide.prev { transform: translateX(-100%); }
```

**Zoom**:
```css
.slide { opacity: 0; transform: scale(0.8); transition: opacity 0.5s ease, transform 0.5s ease; }
.slide.active { opacity: 1; transform: scale(1); }
```
