# CSS — Layout, Design, and Best Practices

*From CSS Projects with Jen Kramer (Frontend Masters)*

## Style Guide First

**Establish a style guide before building** — define your color palette, type scale, and spacing units upfront.

Prevents inconsistency from accumulating as the project grows.

---

## Sizing and Units

### Line-Height

**Without units is a multiplier** of the element's font size. Unitless is preferred — stays proportional if font size changes.

```css
/* GOOD — proportional, scales with font changes */
line-height: 1.5;

/* Less ideal — fixed to 20px, won't scale */
line-height: 20px;
```

### Image Sizing

**max-width: 100%** ensures an image never exceeds its natural size.

**width: 100%** stretches it to fill its container regardless of natural size.

```css
/* GOOD — never stretches beyond original */
img {
  max-width: 100%;
  height: auto;
}

/* Stretches image to container width */
img {
  width: 100%;
}
```

### The Critical Question

**Always ask: "Relative to what?"**

Most CSS sizing is relative to something:
- Font size (em, rem)
- Containing block (%, width)
- Viewport (vw, vh)
- Parent element (flex, grid)

Knowing the reference is key to getting expected behavior.

### Auto Sizing

**auto** lets the browser calculate based on content or context.

```css
width: auto;    /* Width based on content + padding */
height: auto;   /* Height based on content */
margin: 0 auto; /* Center block element horizontally */
```

---

## The Box Model

Every element is a box:
```
Content → Padding → Border → Margin (inside out)
```

**Box-model properties do NOT inherit:**
- margin
- padding
- border
- width
- height

**Set box-sizing: border-box on everything** so width/height apply to the full box, not just content:

```css
*, *::before, *::after { 
  box-sizing: border-box; 
}
```

With `border-box`:
- `width: 100px` = 100px total (content + padding + border)
- `padding: 20px` still fits within that 100px

Without `border-box`:
- `width: 100px` = content only
- Adding `padding: 20px` makes total 140px (100 + 20 + 20)

---

## CSS Inheritance

**Text-related properties inherit:**
- font-size
- color
- font-family
- line-height
- text-align

**Box-model properties do NOT inherit:**
- margin
- padding
- border
- width
- height

```css
body {
  font-size: 16px;  /* Child paragraphs inherit */
  margin: 20px;     /* Child elements do NOT inherit margin */
}
```

---

## Specificity

The browser resolves conflicting styles by specificity score.

**Hierarchy (highest to lowest):**
1. **Inline styles** — `<div style="color: red">`
2. **IDs** — `#header`
3. **Classes, attributes, pseudo-classes** — `.button`, `[type="text"]`, `:hover`
4. **Elements, pseudo-elements** — `p`, `div`, `::before`

When specificity is **equal**, the **later rule wins** (cascade).

```css
/* Specificity: 001 (one class) */
.button { color: blue; }

/* Specificity: 002 (one ID, one class) — wins */
#nav .button { color: red; }

/* Specificity: 001 (one class) — happens later, wins */
.button.active { color: green; }
```

**Tip:** Avoid IDs in stylesheets for styling (high specificity makes overrides hard). Use them for JavaScript targeting.

---

## Flexbox vs Grid

### Flexbox — One-Dimensional

- Works with **one direction** (row or column)
- Best for **variable numbers of items** that flow and wrap naturally
- Common uses:
  - Navigation bars
  - Card rows
  - Toolbars
  - Flexible layouts

```css
.container {
  display: flex;
  flex-direction: row;  /* or column */
  justify-content: space-between;
  gap: 1rem;
}

.item {
  flex: 1;  /* Equal growth */
}
```

### Grid — Two-Dimensional

- Works with **rows AND columns** simultaneously
- Best for **defined layout structures** with precise placement
- Common uses:
  - Page layouts
  - Dashboards
  - Photo galleries
  - Complex interfaces

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}

.header {
  grid-column: 1 / -1;  /* Span all columns */
}
```

### Complementary Use

**Grid for overall page structure, Flexbox for items within a grid cell.**

```css
.page {
  display: grid;
  grid-template-columns: sidebar 1fr;
  grid-template-rows: header 1fr footer;
}

.content {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}
```

---

## Semantic HTML

### Heading Hierarchy

**Don't skip heading levels** — if you need a different visual size, use a class.

Heading hierarchy matters for:
- Screen readers
- SEO
- Document outline

```html
<!-- GOOD -->
<h1>Main Title</h1>
<h2>Section</h2>
<h3>Subsection</h3>

<!-- BAD — skips h2 -->
<h1>Main Title</h1>
<h3>Subsection</h3>
```

### Article vs Section

- **`<article>`** — Self-contained, syndicate-able content
  - Blog post
  - News story
  - Product review
  - Can be republished independently

- **`<section>`** — Thematically related content within a document
  - Chapter of a longer article
  - Tab panel within a tabbed interface
  - NOT independent on its own

### Figure and Figcaption

```html
<figure>
  <img src="diagram.jpg" alt="Process flow diagram">
  <figcaption>The three-step process showing inputs and outputs</figcaption>
</figure>
```

The `alt` attribute describes what you see — for screen readers and when the image fails to load.

---

## Pseudo-Classes vs Pseudo-Elements

### Pseudo-Classes (Single Colon)

Select an element in a particular **state:**

```css
:hover      /* Mouse over element */
:focus      /* Element has focus */
:nth-child(2)  /* Second child */
:first-of-type /* First of its type */
:visited    /* Visited link */
:disabled   /* Disabled form element */
```

### Pseudo-Elements (Double Colon)

Select a **part of an element** or create a virtual element:

```css
::before    /* Insert content before element */
::after     /* Insert content after element */
::first-line /* First line of text */
::first-letter /* First letter of text */
::placeholder /* Placeholder text in inputs */
::selection /* User-selected text */
```

```html
<!-- Example: decorative element without adding HTML -->
<style>
  .quote::before {
    content: """;  /* Add quotation mark */
    font-size: 2em;
  }
</style>

<p class="quote">This is a quote</p>
```

---

## Color Contrast

Always verify sufficient contrast between foreground and background.

**WCAG AA requires:** Minimum contrast ratio of **4.5:1** for normal text.

**Tools:**
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- Browser DevTools accessibility checks

```css
/* GOOD — High contrast */
color: #000;           /* Black text */
background-color: #fff; /* White background */
/* Ratio: 21:1 */

/* BAD — Low contrast */
color: #999;           /* Gray text */
background-color: #eee; /* Light gray background */
/* Ratio: 3.5:1 — fails WCAG AA */
```

---

## Typographic Scale

Rather than arbitrary font sizes, use a **consistent ratio between sizes** (e.g., 1.25 or 1.333).

Defines visual harmony.

```css
/* Base size: 16px, Ratio: 1.25 */
--base: 16px;

--text-xs: calc(var(--base) / (1.25 * 1.25));     /* 10.24px */
--text-sm: calc(var(--base) / 1.25);              /* 12.8px */
--text-base: var(--base);                         /* 16px */
--text-lg: calc(var(--base) * 1.25);              /* 20px */
--text-xl: calc(var(--base) * 1.25 * 1.25);       /* 25px */
--text-2xl: calc(var(--base) * 1.25 * 1.25 * 1.25); /* 31.25px */

body { font-size: var(--text-base); }
h1 { font-size: var(--text-2xl); }
h2 { font-size: var(--text-xl); }
small { font-size: var(--text-sm); }
```

**Reference:** [Typographic Scale](https://pairfonts.eu/articles/typographic-scale.php)

---

## Next Steps

- [Flexbox + Grid Cheat Sheet](https://frontendmasters.com/assets/resources/jenkramer/flexbox-grid-cheat-sheet.pdf)
- [Frontend Masters CSS Courses](https://frontendmasters.com/)
- Practice building layouts with both Flexbox and Grid

---

*Course: Frontend Masters | Instructor: Jen Kramer*
