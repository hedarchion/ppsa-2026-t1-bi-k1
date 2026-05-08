---
name: interactive-slide-deck
description: Build interactive HTML slide decks from Markdown exam/quiz content
---

# Interactive Slide Deck Builder

Build a self-contained interactive HTML/JS slide deck from Markdown source files (e.g. exam papers, quizzes). Single `index.html` — no build tools, no frameworks, no external deps beyond a Google Font.

## Architecture

### File structure
```
project/
  index.html          ← single-file deliverable
  source_questions.md ← question paper (verbatim source)
  source_answers.md   ← answer key / transcription (verbatim source)
```

### Slide container sizing
Strict 16:9 aspect ratio box centered in viewport:
```css
.slide-container {
  aspect-ratio: 16/9;
  width: min(100vw, calc(100vh * 16/9));
  height: min(100vh, calc(100vw * 9/16));
}
```
On mobile (≤768px), the aspect-ratio constraint is removed — slides fill full viewport height and scroll internally.

### Font sizing system
All body text is homogenized within each slide using `clamp()`:
- **Rule**: every interactive/readable text element in a slide shares the SAME font-size declaration
- **Pattern**: `font-size: clamp(.9rem, 1.5vw, 1.5rem)`
- The `rem` min bound ensures readability on small viewports
- The `vw` preferred value scales with screen width
- The `rem` max bound prevents overflow on very large screens (4K)
- Font resize buttons (A⁻/A⁺) toggle root font-size classes (15–19px) — the clamp bounds still prevent overflow

### Answer key font sizing
Answer keys are sized per-part at the maximum possible:
- Letter-only keys (single chars): `clamp(2.4rem, 5vw, 5rem)`
- Word/phrase keys: `clamp(1.2rem, 2.4vw, 2.4rem)`
- Grid columns must be wide enough for the longest word to fit on one line (no mid-word breaks)

## Layout Patterns

### Two-column grid
```css
.two-col { display: grid; grid-template-columns: 1fr 1fr; }
```
Stacks to single column on mobile via `@media(max-width:768px)`.

### Question card (.qa-card)
Used for Part 1 MCQ. Each card has:
- `.q-num` — question number badge
- `.q-passage` — reading passage text
- `.question-text` — the question
- `.options` — MCQ options (mutually exclusive)

### Reading passage (.passage-text)
Used for Part 2 error correction. Single-column scrollable passage with inline interactive elements.

### Table fill (.table-fill)
Table with `.blank-cell` elements that students click to reveal answers. Cells are styled as dashed yellow boxes that turn green when filled.

### T/F items (.tf-group)
Row of text + True/False buttons. Buttons get color feedback (green for correct, red for wrong).

### Gap fill (.gap-section)
Grid of passage area (1fr) + sentence cards (auto). `.gap-slot` inline elements in the passage. `.sentence-card` elements in the cards pool. Click card → fills next empty gap sequentially.

## Interactivity Patterns

| Pattern | HTML pattern | JS behavior |
|---------|-------------|-------------|
| MCQ | `.option` inside `.options[data-correct]` | Select one → click Check → correct/wrong classes |
| Error word | `.error-word[data-correct]` with `.correction` child | Toggle `.show` → strikethrough wrong + inline green correction |
| Table fill | `.blank-cell` in `<td>` elements | Click → fills with pre-set answer text, `.filled` class |
| T/F | `.tf-btn[data-value]` inside `.tf-item[data-correct]` | Click T/F → compares to data-correct, adds color class |
| Gap fill | `.sentence-card` + `.gap-slot` sequential | Click card → fills first empty `.gap-slot` |
| Answer key | `.answer-item .q-answer.hidden[data-answer]` | Click item or "Reveal All" → removes `.hidden` class |

### Error word behavior
Click the bolded error word → the wrong text gets `text-decoration:line-through` in gray, and the `.correction` span appears inline right after it in green. No tooltip/popover.

## Mobile Responsiveness

Breakpoint at **768px**:
- Aspect ratio removed from slide container
- `.two-col` stacks to single column
- `.signage` flex row stacks to column
- Nav bar becomes `position:fixed;bottom:0` with safe-area padding
- All interactive elements get larger touch targets
- Touch swipe (left/right with 40px threshold) added via JS

## Answer Key System

Each part's answer key follows this structure:
```html
<div class="answer-grid" style="grid-template-columns:repeat(auto-fill,minmax(clamp(Wpx,Xvw,Ypx),1fr))" data-answers='{...}'>
  <div class="answer-item">
    <span class="q-label">Question N</span>
    <span class="q-answer hidden" data-answer="...">...</span>
  </div>
</div>
```

- `data-answers` JSON attribute on grid (used by "Reveal All" to track state)
- `.q-answer.hidden` — text transparent with gray background pill
- Click removes `.hidden` → text appears green, pill gone
- Selected `minmax` column width must fit the longest word without wrapping

## How to Add New Slides

1. Add a new `<div class="slide" data-slide="N">` inside `.slide-container`
2. Include `.slide-header` with `.part-label` and `.slide-num`
3. Include `.slide-body` with content
4. Use one of the 6 interactivity patterns above
5. Add answer key slide with matching `.answer-grid`
6. Update `totalSlides` in the JS init

## Key Constraints
- No overflow, no scroll (except mobile where slides scroll internally)
- Font homogenized within each slide at max safe size
- 16:9 box maintained on desktop
- Light theme only
- Verbatim source text, no editing
- Inter font from Google Fonts
