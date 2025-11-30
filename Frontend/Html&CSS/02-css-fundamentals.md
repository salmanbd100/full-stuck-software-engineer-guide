# CSS Fundamentals

Understanding CSS fundamentals is essential for styling web pages effectively. This covers selectors, specificity, the box model, and positioning.

## 📚 Core Concepts

### 1. CSS Selectors

**Basic Selectors**
```css
/* Element selector */
p {
    color: blue;
}

/* Class selector */
.button {
    background-color: green;
}

/* ID selector (use sparingly) */
#header {
    font-size: 24px;
}

/* Universal selector */
* {
    margin: 0;
    padding: 0;
}
```

**Combinators**
```css
/* Descendant selector (all children) */
.container p {
    color: gray;
}

/* Child selector (direct children only) */
.container > p {
    font-weight: bold;
}

/* Adjacent sibling */
h2 + p {
    margin-top: 0;
}

/* General sibling */
h2 ~ p {
    color: blue;
}
```

**Attribute Selectors**
```css
/* Has attribute */
input[required] {
    border-color: red;
}

/* Exact value */
input[type="email"] {
    background-color: lightblue;
}

/* Contains value */
a[href*="example"] {
    color: green;
}

/* Starts with */
a[href^="https"] {
    padding-left: 20px;
}

/* Ends with */
a[href$=".pdf"] {
    background: url('pdf-icon.png');
}

/* Contains word */
div[class~="highlight"] {
    background: yellow;
}
```

**Pseudo-classes**
```css
/* Link states */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: orange; }

/* Form states */
input:focus {
    border-color: blue;
    outline: 2px solid blue;
}

input:disabled {
    opacity: 0.5;
}

input:checked + label {
    font-weight: bold;
}

input:valid {
    border-color: green;
}

input:invalid {
    border-color: red;
}

/* Structural pseudo-classes */
li:first-child {
    font-weight: bold;
}

li:last-child {
    border-bottom: none;
}

li:nth-child(odd) {
    background-color: #f0f0f0;
}

li:nth-child(3n) {
    color: blue;
}

p:not(.special) {
    color: gray;
}

div:empty {
    display: none;
}
```

**Pseudo-elements**
```css
/* ::before and ::after */
.quote::before {
    content: '"';
    font-size: 2em;
}

.quote::after {
    content: '"';
    font-size: 2em;
}

/* ::first-letter and ::first-line */
p::first-letter {
    font-size: 2em;
    font-weight: bold;
}

p::first-line {
    font-variant: small-caps;
}

/* ::selection */
::selection {
    background-color: yellow;
    color: black;
}

/* ::placeholder */
input::placeholder {
    color: #999;
    font-style: italic;
}
```

### 2. Specificity

**Specificity Hierarchy**
```css
/* Inline styles: 1000 */
<div style="color: red;">

/* ID: 100 */
#header { color: blue; }

/* Class, pseudo-class, attribute: 10 */
.button { color: green; }
:hover { color: orange; }
[type="text"] { color: purple; }

/* Element, pseudo-element: 1 */
p { color: black; }
::before { color: gray; }

/* Specificity calculation */
/* (inline, id, class, element) */

/* 0,1,0,1 - Specificity: 101 */
#header p { color: red; }

/* 0,0,2,1 - Specificity: 21 */
.container .button p { color: blue; }

/* Winner: #header p (higher specificity) */
```

**Important Rule**
```css
/* Overrides everything (use sparingly!) */
p {
    color: red !important;
}

/* Even this won't override */
#special {
    color: blue; /* Lost! */
}
```

### 3. The Box Model

```css
.box {
    /* Content */
    width: 200px;
    height: 100px;

    /* Padding (inside border) */
    padding: 20px;
    /* Or: padding: top right bottom left; */
    padding: 10px 20px 10px 20px;
    /* Or: padding: vertical horizontal; */
    padding: 10px 20px;

    /* Border */
    border: 2px solid black;
    border-width: 2px;
    border-style: solid;
    border-color: black;
    border-radius: 8px;

    /* Margin (outside border) */
    margin: 20px;
    margin: 10px 20px 30px 40px;

    /* Total width = width + padding + border + margin */
    /* Total: 200 + 40 + 4 + 40 = 284px */
}

/* box-sizing (changes box model) */
.box-sizing-example {
    box-sizing: border-box; /* Includes padding and border in width */
    width: 200px;
    padding: 20px;
    border: 2px solid black;
    /* Total width = 200px (not 244px) */
}

/* Best practice: Apply to all elements */
* {
    box-sizing: border-box;
}
```

### 4. Display Property

```css
/* Block (takes full width) */
div {
    display: block;
}

/* Inline (flows with text) */
span {
    display: inline;
}

/* Inline-block (best of both) */
.button {
    display: inline-block;
    width: 100px;
    height: 40px;
}

/* None (removes from layout) */
.hidden {
    display: none;
}

/* Visibility (keeps space) */
.invisible {
    visibility: hidden;
}
```

### 5. Position Property

**Static (default)**
```css
.static {
    position: static; /* Default, not affected by top/left/right/bottom */
}
```

**Relative**
```css
.relative {
    position: relative;
    top: 10px;    /* Moves 10px down from original position */
    left: 20px;   /* Moves 20px right from original position */
    /* Original space is preserved */
}
```

**Absolute**
```css
.container {
    position: relative; /* Positioning context */
}

.absolute {
    position: absolute;
    top: 0;
    right: 0;
    /* Positioned relative to nearest positioned ancestor */
    /* Removed from normal document flow */
}
```

**Fixed**
```css
.fixed-header {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    /* Fixed to viewport, doesn't scroll */
}
```

**Sticky**
```css
.sticky-nav {
    position: sticky;
    top: 0;
    /* Behaves like relative until scrolled to threshold, then becomes fixed */
}
```

**Z-index**
```css
.modal {
    position: fixed;
    z-index: 1000; /* Higher values appear on top */
}

.overlay {
    position: fixed;
    z-index: 999;
}

.content {
    position: relative;
    z-index: 1;
}
```

### 6. Colors and Units

**Colors**
```css
.colors {
    /* Named colors */
    color: red;

    /* Hex */
    color: #ff0000;
    color: #f00; /* Shorthand */

    /* RGB */
    color: rgb(255, 0, 0);

    /* RGBA (with alpha/transparency) */
    color: rgba(255, 0, 0, 0.5); /* 50% transparent */

    /* HSL (Hue, Saturation, Lightness) */
    color: hsl(0, 100%, 50%);

    /* HSLA */
    color: hsla(0, 100%, 50%, 0.5);

    /* CSS variables */
    --primary-color: #3498db;
    color: var(--primary-color);
}
```

**Length Units**
```css
.units {
    /* Absolute units */
    width: 100px;    /* Pixels */
    width: 1in;      /* Inches */
    width: 2.54cm;   /* Centimeters */

    /* Relative units */
    width: 50%;      /* Percentage of parent */
    width: 10em;     /* Relative to element's font-size */
    width: 10rem;    /* Relative to root font-size */
    width: 50vw;     /* 50% of viewport width */
    width: 50vh;     /* 50% of viewport height */
    width: 5vmin;    /* 5% of smaller viewport dimension */
    width: 5vmax;    /* 5% of larger viewport dimension */

    /* Modern units */
    width: calc(100% - 20px);
    width: min(500px, 100%);
    width: max(300px, 50%);
    width: clamp(300px, 50%, 800px); /* min, preferred, max */
}
```

### 7. Typography

```css
.typography {
    /* Font family */
    font-family: 'Arial', sans-serif;
    font-family: 'Georgia', serif;
    font-family: 'Courier New', monospace;

    /* Font size */
    font-size: 16px;
    font-size: 1rem;
    font-size: 1.5em;

    /* Font weight */
    font-weight: normal;  /* 400 */
    font-weight: bold;    /* 700 */
    font-weight: 600;

    /* Font style */
    font-style: normal;
    font-style: italic;

    /* Line height */
    line-height: 1.5;     /* Unitless (recommended) */
    line-height: 24px;

    /* Letter spacing */
    letter-spacing: 0.5px;
    letter-spacing: -0.05em;

    /* Word spacing */
    word-spacing: 2px;

    /* Text align */
    text-align: left;
    text-align: center;
    text-align: right;
    text-align: justify;

    /* Text decoration */
    text-decoration: none;
    text-decoration: underline;
    text-decoration: line-through;

    /* Text transform */
    text-transform: uppercase;
    text-transform: lowercase;
    text-transform: capitalize;

    /* White space */
    white-space: normal;
    white-space: nowrap;   /* No wrapping */
    white-space: pre;      /* Preserve whitespace */
    white-space: pre-wrap; /* Preserve + wrap */

    /* Text overflow */
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;

    /* Word break */
    word-break: break-all;
    word-wrap: break-word;
}
```

### 8. Backgrounds

```css
.backgrounds {
    /* Color */
    background-color: #f0f0f0;

    /* Image */
    background-image: url('image.jpg');

    /* Repeat */
    background-repeat: no-repeat;
    background-repeat: repeat-x;
    background-repeat: repeat-y;

    /* Position */
    background-position: center;
    background-position: top right;
    background-position: 50% 50%;

    /* Size */
    background-size: cover;      /* Fill container */
    background-size: contain;    /* Fit in container */
    background-size: 100px 200px;

    /* Attachment */
    background-attachment: fixed;  /* Parallax effect */
    background-attachment: scroll;

    /* Shorthand */
    background: #f0f0f0 url('image.jpg') no-repeat center/cover;

    /* Multiple backgrounds */
    background-image: url('front.png'), url('back.png');
    background-position: center, top left;
    background-repeat: no-repeat, repeat;

    /* Gradients */
    background: linear-gradient(to right, red, blue);
    background: linear-gradient(45deg, red, yellow, green);
    background: radial-gradient(circle, red, blue);
}
```

### 9. CSS Variables (Custom Properties)

```css
:root {
    /* Define variables */
    --primary-color: #3498db;
    --secondary-color: #2ecc71;
    --spacing: 16px;
    --border-radius: 4px;
    --font-size-base: 16px;
}

.component {
    /* Use variables */
    background-color: var(--primary-color);
    padding: var(--spacing);
    border-radius: var(--border-radius);
    font-size: var(--font-size-base);

    /* Fallback value */
    color: var(--text-color, black);

    /* Calculations */
    margin: calc(var(--spacing) * 2);
}

/* Override in specific context */
.dark-theme {
    --primary-color: #2c3e50;
    --secondary-color: #27ae60;
}
```

## 🎯 Common Interview Questions

### Q1: What is the CSS Box Model?

**Answer:**
Every element is a box with:
- **Content**: The actual content (text, images)
- **Padding**: Space inside border
- **Border**: Surrounds padding
- **Margin**: Space outside border

`box-sizing: border-box` includes padding and border in the width.

### Q2: Explain CSS specificity

**Answer:**
Specificity determines which CSS rule applies:
1. Inline styles (1000)
2. IDs (100)
3. Classes, attributes, pseudo-classes (10)
4. Elements, pseudo-elements (1)

`!important` overrides everything (avoid if possible).

### Q3: What's the difference between `display: none` and `visibility: hidden`?

**Answer:**
- `display: none`: Removes element from layout (doesn't take up space)
- `visibility: hidden`: Hides element but keeps its space

## 💡 Practical Examples

### Example 1: Centering Content

```css
/* Horizontal centering */
.center-horizontal {
    margin: 0 auto;
    width: 800px;
}

/* Flexbox centering */
.flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

/* Grid centering */
.grid-center {
    display: grid;
    place-items: center;
    height: 100vh;
}

/* Absolute centering */
.absolute-center {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

### Example 2: Card Component

```css
.card {
    /* Box model */
    width: 300px;
    padding: 20px;
    margin: 20px;

    /* Border and shadow */
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

    /* Typography */
    font-family: Arial, sans-serif;
    line-height: 1.6;

    /* Transition */
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
}

.card-title {
    font-size: 1.5rem;
    font-weight: bold;
    margin-bottom: 10px;
    color: #333;
}

.card-description {
    color: #666;
    margin-bottom: 15px;
}

.card-button {
    display: inline-block;
    padding: 10px 20px;
    background-color: #3498db;
    color: white;
    text-decoration: none;
    border-radius: 4px;
    transition: background-color 0.3s ease;
}

.card-button:hover {
    background-color: #2980b9;
}
```

### Example 3: Responsive Typography

```css
:root {
    --font-size-base: 16px;
    --line-height-base: 1.6;
}

body {
    font-size: var(--font-size-base);
    line-height: var(--line-height-base);
}

h1 {
    font-size: clamp(2rem, 5vw, 3.5rem);
    line-height: 1.2;
}

h2 {
    font-size: clamp(1.5rem, 4vw, 2.5rem);
    line-height: 1.3;
}

p {
    font-size: clamp(1rem, 2vw, 1.125rem);
    max-width: 65ch; /* Optimal reading length */
}
```

## 🚨 Common Pitfalls

1. **Not using `box-sizing: border-box`**
2. **Overusing `!important`**
3. **Using inline styles** (hard to maintain)
4. **Not considering specificity conflicts**
5. **Using pixels for everything** (not responsive)
6. **Forgetting vendor prefixes** (when needed)

## 🎓 Best Practices

1. **Use `box-sizing: border-box`** on all elements
2. **Prefer classes over IDs** for styling
3. **Use CSS variables** for theming
4. **Use relative units** (rem, em, %) for responsiveness
5. **Follow BEM naming** or similar methodology
6. **Organize CSS** logically (variables, base, components)
7. **Minimize use of `!important`**
8. **Use shorthand properties** when appropriate

## 🔗 Related Topics

- [Flexbox](./03-flexbox.md)
- [CSS Grid](./04-grid.md)
- [Responsive Design](./05-responsive-design.md)

---

[← Back to HTML & CSS](./README.md) | [Next: Flexbox →](./03-flexbox.md)
