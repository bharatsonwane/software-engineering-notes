# CSS

Notes for CSS — layout, responsive design, and modern styling.

Files are numbered in **learning order**: `01-css-topic.md`, `02-css-topic.md`, and so on.

**Prerequisite:** [HTML](../html/) basics (especially semantics and forms) before layout-heavy chapters.

## Contents

**01. [Syntax & Selectors](./01-css-syntax-selectors.md)**  
Tier — Fundamental

1. How CSS applies to HTML  
2. Selectors (element, class, id, attribute)  
3. Combinators and pseudo-classes  
4. Specificity and cascade  
5. Common questions

**02. [Box Model](./02-css-box-model.md)**  
Tier — Fundamental

1. Content, padding, border, margin  
2. `box-sizing`  
3. Block vs inline vs inline-block  
4. Margin collapse  
5. Common questions

**03. [Display & Position](./03-css-display-position.md)**  
Tier — Fundamental

1. `display` values  
2. `position` — static, relative, absolute, fixed, sticky  
3. `z-index` and stacking  
4. Overflow  
5. Common questions

**04. [Flexbox](./04-css-flexbox.md)**  
Tier — Fundamental

1. Flex container and items  
2. Main axis and cross axis  
3. Alignment and wrapping  
4. Common layout patterns  
5. Common questions

**05. [Grid](./05-css-grid.md)**  
Tier — Fundamental

1. Grid template rows and columns  
2. `fr`, gaps, and areas  
3. Placing items  
4. Flexbox vs Grid  
5. Common questions

**06. [Typography](./06-css-typography.md)**  
Tier — Fundamental

1. Font families and stacks  
2. Size, weight, line-height  
3. Web fonts (`@font-face`, Google Fonts)  
4. Text alignment and decoration  
5. Common questions

**07. [Colors & Backgrounds](./07-css-colors-backgrounds.md)**  
Tier — Fundamental

1. Color formats (hex, rgb, hsl)  
2. `background` shorthand  
3. Gradients  
4. Opacity and transparency  
5. Common questions

**08. [Units & Sizing](./08-css-units-sizing.md)**  
Tier — Intermediate

1. Absolute vs relative units  
2. `rem`, `em`, `%`, `vw`/`vh`  
3. `min-` / `max-` / `clamp()`  
4. Aspect ratio  
5. Common questions

**09. [Responsive Design](./09-css-responsive-design.md)**  
Tier — Intermediate

1. Mobile-first mindset  
2. Media queries  
3. Fluid layouts and breakpoints  
4. Container queries (intro)  
5. Common questions

**10. [Transitions & Animations](./10-css-transitions-animations.md)**  
Tier — Intermediate

1. `transition` property  
2. `@keyframes` and animation  
3. `transform`  
4. Performance (GPU-friendly properties)  
5. Common questions

**11. [Modern CSS](./11-css-modern-features.md)**  
Tier — Intermediate

1. Custom properties (`--var`)  
2. Nesting (native)  
3. `:is`, `:where`, `:has`  
4. Logical properties  
5. Common questions

**12. [Architecture & Best Practices](./12-css-architecture-best-practices.md)**  
Tier — Advanced

1. Naming (BEM overview)  
2. Organizing stylesheets  
3. Reset vs normalize  
4. Debugging in DevTools  
5. Common questions

## Learning path

```
01–07  Fundamental  →  selectors, box model, flex, grid, type, color
08–11  Intermediate →  responsive, motion, modern CSS
12     Advanced     →  maintainable CSS at scale
```

Each file uses **relative** section numbers (`1.`, `1.1.`, `1.1.1.` within that file).

## Topic map (by area)

| Area | Files |
|------|-------|
| Core mechanics | 01, 02, 03 |
| Layout | 04, 05, 08, 09 |
| Visual design | 06, 07, 10 |
| Production | 11, 12 |
