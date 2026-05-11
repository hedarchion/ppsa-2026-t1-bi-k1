


Here is the updated `skill.md`, fully refactored to prioritize the **IMAGO IMB86AIOS+ (Chrome on Android)** technical specifications. 

This update replaces the previous JS-scaling architecture with the hardware-specific, performance-optimized, and distance-scaled `vw` architecture required for this 86-inch 4K smartboard.

***

```markdown
---
name: imago-4k-slide-architecture
description: Architecture rules and constraints for generating 4K slide layouts optimized for the IMAGO IMB86AIOS+ interactive smartboard (Chrome on Android).
---

# IMAGO 4K Slide Architecture Skill

## 🎯 Core Objective
Generate performance-optimized, highly legible interactive slides designed specifically for an 86-inch 4K display viewed from 2–8 meters away. 

## 🏗️ Display & Layout Architecture
1. **Target Viewport**: Design for a fluid `100vw` width and `16:9` aspect ratio. 
2. **Physical Resolution Assumed**: `3840 × 2160` (4K UHD).
3. **Safe Zone Padding**: You **must** apply a `5%` (approx 192px) inner padding on all sides of the main slide container. This prevents UI collisions with IMAGO's floating sidebars and the Android system "chin."

## 🔠 Sizing & Typography (Distance-Optimized)
Do not use pixel (`px`) values for fonts or standard media queries. Because students are viewing from a distance, maintain absolute scale regardless of browser zoom using **Viewport Width (`vw`)** units:

* **H1 / Main Title**: `10vw` (Instant readability at 8m)
* **H2 / Subtitle**: `6vw` (Secondary emphasis)
* **Body Text**: `3vw` (Minimum for general reading)
* **Small Caps / Nav**: `2vw` (Absolute minimum for UI)

## ⚡ Critical CSS Overrides
Include these exact properties in your global styles to handle the Android Chrome environment and IR Touch input natively:

```css
body {
  /* Prevent accidental refresh when swiping down at the board */
  overscroll-behavior-y: contain;
  
  /* Remove 300ms tap delay for instant touch response */
  touch-action: manipulation;
  
  /* Hardware acceleration for 4K transitions */
  transform: translate3d(0, 0, 0);
  
  /* Classroom glare reduction */
  background-color: #F9F9F9; 
  color: #1A1A1A;
}

.interactive-element {
  /* Minimum physical hit area for fingers/stylus */
  min-width: 100px;
  min-height: 100px;
  
  /* Disable hover (non-existent on touch) and use active states */
  outline: none;
}

.interactive-element:active {
  background-color: #E2E8F0; /* High-contrast feedback */
}
```

## 🖱️ JavaScript & Hardware Specs
When writing Javascript components or event listeners, adhere to the following hardware-specific optimizations:

1. **Clicker / Pen Support**: Map slide navigation to standard keyboard events so wireless clickers and IMAGO pen buttons function automatically.
   ```javascript
   window.addEventListener('keydown', (e) => {
     if (e.key === 'ArrowRight' || e.key === 'PageDown' || e.key === ' ') nextSlide();
     if (e.key === 'ArrowLeft' || e.key === 'PageUp') prevSlide();
   });
   ```
2. **Touch Events**: The IMAGO IR frame registers touches as a "pointer". Bind custom interactions to `pointerup` instead of `click` for faster response times.
3. **GPU Offloading**: Apply `will-change: transform` strictly to elements that animate during slide transitions to prevent the Android SoC from stuttering at 4K.
4. **Image Boundaries**: Use `object-fit: contain` for images and diagrams to ensure visual boundaries align perfectly with the IR touch matrix.

## 🚫 Absolute Constraints (NEVER DO THESE)
As an LLM agent, strictly adhere to these limitations:
1. **NEVER use `:hover` states for critical UI.** IR touchframes do not support hovering; rely entirely on `:active` for feedback.
2. **NEVER create touch targets smaller than `100px` × `100px`.** Smaller targets fail when used with a thick stylus or finger on an 86-inch board.
3. **NEVER use `px` for font sizing.** Always strictly adhere to the `vw` scale provided above to maintain distance legibility.
4. **NEVER place navigation or interactive buttons in the outer 5% margin.** They will conflict with the Android OS gestures or the IMAGO sidebars.
```
