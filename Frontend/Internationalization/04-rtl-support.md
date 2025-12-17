# RTL Support

## Overview

Right-to-Left (RTL) language support is one of the most overlooked aspects of internationalization, yet it affects over 1 billion people worldwide who use RTL languages. Beyond simply flipping the text direction, proper RTL implementation requires careful attention to CSS, layout, component structure, and thorough testing. This guide covers everything needed to build truly RTL-compatible applications.

## Table of Contents
- [RTL Languages Overview](#rtl-languages-overview)
- [HTML Foundation](#html-foundation)
- [CSS Logical Properties](#css-logical-properties)
- [Flexbox and Grid in RTL](#flexbox-and-grid-in-rtl)
- [Icons and Images](#icons-and-images)
- [Component-Level RTL](#component-level-rtl)
- [Testing RTL Layouts](#testing-rtl-layouts)
- [Common Mistakes](#common-mistakes)
- [Performance Considerations](#performance-considerations)
- [Interview Questions](#interview-questions)

---

## RTL Languages Overview

### Which Languages Use RTL?

```javascript
// Complete list of RTL languages
const rtlLanguages = {
  // Major RTL languages
  arabic: {
    code: 'ar',
    speakers: '374 million native speakers',
    variants: ['ar-SA', 'ar-AE', 'ar-EG', 'ar-US', 'ar-GB'],
    scripts: 'Arabic script'
  },

  hebrew: {
    code: 'he',
    speakers: '9 million',
    country: 'Israel',
    scripts: 'Hebrew script'
  },

  persian: {
    code: 'fa',
    speakers: '70+ million',
    countries: ['Iran', 'Afghanistan'],
    altName: 'Farsi'
  },

  urdu: {
    code: 'ur',
    speakers: '70+ million',
    countries: ['Pakistan', 'India'],
    scripts: 'Urdu script (similar to Arabic)'
  },

  // Lesser-known RTL languages
  yiddish: {
    code: 'yi',
    speakers: '1-3 million',
    scripts: 'Hebrew script'
  },

  dhivehi: {
    code: 'dv',
    country: 'Maldives',
    speakers: '0.3 million'
  },

  syriac: {
    code: 'syr',
    scripts: 'Syriac script'
  },

  nko: {
    code: 'nqo',
    speakers: 'West African languages'
  }
};

// Statistics
const statistics = {
  rtl_speakers: '1.7+ billion people',
  percentage_of_world: '~20% of world population',
  top_rtl_country: 'Arabic-speaking countries',
  internet_growth: 'Arabic internet users growing fastest'
};
```

### RTL vs LTR Characteristics

```javascript
// Key differences in RTL languages
const rtlCharacteristics = {
  text_direction: 'Right to Left',
  reading_direction: 'Lines read right to left',
  page_flow: 'Sidebar on left (like LTR on right)',
  number_direction: 'Numbers still read left to right (bidirectional)',
  punctuation: 'Same as LTR, but position may differ',

  // Example: "Hello" in Arabic
  arabic_example: {
    text: 'E1-('',
    direction: 'RTL',
    pronunciation: 'Marhaba',
    meaning: 'Hello'
  },

  // Bidirectional text (mixed LTR and RTL)
  mixed_example: {
    text: ''D391: $100',
    meaning: 'Price: $100',
    challenge: 'Contains both Arabic (RTL) and English (LTR) with numbers'
  }
};
```

---

## HTML Foundation

### The dir Attribute

```html
<!-- CRITICAL: Set dir attribute at document root -->

<!-- LTR (default) -->
<html dir="ltr">
  <!-- Page content is left-to-right -->
</html>

<!-- RTL -->
<html dir="rtl">
  <!-- Page content is right-to-left -->
</html>

<!-- Auto (browser detects) -->
<html dir="auto">
  <!-- Browser determines direction based on content -->
</html>

<!-- When to use each -->
<!--
  ltr: English, French, German, etc.
  rtl: Arabic, Hebrew, Persian, Urdu, etc.
  auto: Mixed content or when not known in advance
-->
```

### Proper HTML Structure for RTL

```html
<!-- GOOD: Complete RTL markup -->
<!DOCTYPE html>
<html dir="rtl" lang="ar">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="...">
    <title>Arabic Page</title>
  </head>
  <body>
    <!-- Content -->
  </body>
</html>

<!-- IMPORTANT: Match dir and lang attributes -->
<!--
  dir="rtl" with lang="ar-SA" (Arabic)
  dir="rtl" with lang="he" (Hebrew)
  dir="ltr" with lang="en-US" (English)

  Mismatches cause accessibility issues!
-->
```

### Lang Attribute Importance

```html
<!-- WRONG: RTL dir without lang -->
<html dir="rtl">
  <!-- Screen readers don't know language -->
  <!-- CSS :lang() pseudo-class won't work -->
</html>

<!-- RIGHT: Both dir and lang specified -->
<html dir="rtl" lang="ar-SA">
  <!-- Correct language and direction -->
  <!-- Screen readers know to use Arabic pronunciation -->
  <!-- CSS can use :lang(ar) for Arabic-specific styles -->
</html>

<!-- Setting direction per component -->
<div dir="rtl" lang="ar">
  E-*HI ('D91(J)
</div>

<div dir="ltr" lang="en">
  English content
</div>

<!-- Nested directions: Important for embedded content! -->
<html dir="rtl" lang="ar">
  <body>
    <p>F5 91(J E9 English word</p>
    <!-- Browser handles bidirectional text -->
  </body>
</html>
```

---

## CSS Logical Properties

### Understanding Logical Properties

Logical properties are CSS properties that adapt to text direction automatically.

```javascript
// Two approaches to RTL CSS:
const approaches = {
  physical: {
    description: 'Use left/right, top/bottom (breaks in RTL)',
    example: 'margin-left, padding-right, border-top',
    problem: 'Must write two versions of every property'
  },

  logical: {
    description: 'Use start/end, block/inline (adapts automatically)',
    example: 'margin-inline-start, padding-inline-end, border-block-start',
    benefit: 'Same CSS works for LTR and RTL'
  }
};

// The key mapping:
const logicalMapping = {
  // Start/End (horizontal direction)
  'margin-inline-start': 'margin-left (LTR) or margin-right (RTL)',
  'margin-inline-end': 'margin-right (LTR) or margin-left (RTL)',
  'padding-inline-start': 'padding-left (LTR) or padding-right (RTL)',
  'padding-inline-end': 'padding-right (LTR) or padding-left (RTL)',
  'border-inline-start': 'border-left (LTR) or border-right (RTL)',
  'border-inline-end': 'border-right (LTR) or border-left (RTL)',
  'text-align': 'start or end (adapts to direction)',

  // Block (vertical direction - doesn't change)
  'margin-block-start': 'margin-top (always)',
  'margin-block-end': 'margin-bottom (always)',
  'padding-block-start': 'padding-top (always)',
  'padding-block-end': 'padding-bottom (always)',
  'border-block-start': 'border-top (always)',
  'border-block-end': 'border-bottom (always)'
};
```

### Logical Properties Examples

```css
/* WRONG: Physical properties (breaks in RTL) */
.sidebar {
  margin-left: 2rem;      /* Wrong: appears on right in RTL */
  padding-right: 1rem;    /* Wrong: appears on left in RTL */
  border-left: 3px solid #333;  /* Wrong border position */
}

/* RIGHT: Logical properties (works in both LTR and RTL) */
.sidebar {
  margin-inline-start: 2rem;      /* Left in LTR, right in RTL */
  padding-inline-end: 1rem;       /* Right in LTR, left in RTL */
  border-inline-start: 3px solid #333;  /* Left in LTR, right in RTL */
}

/* Real-world example: Card layout */
.card {
  padding: 1rem;
  /* Use inline-start for first item */
  margin-inline-start: 1.5rem;
  /* Use inline-end for trailing content */
  margin-inline-end: auto;
}

.card-header {
  border-bottom: 1px solid #ddd;  /* Use block for vertical */
  padding-block-end: 1rem;        /* Vertical always same */
}

/* Text alignment with logical properties */
.button {
  text-align: start;      /* Left in LTR, right in RTL */
}

.right-aligned {
  text-align: end;        /* Right in LTR, left in RTL */
}

/* Inline layout */
.horizontal-list {
  display: flex;
  gap: 1rem;              /* Gap works in both directions */
  margin-inline: 1rem;    /* Shorthand for start and end */
}

/* Shorthand logical properties */
.container {
  /* All four directions */
  margin: 1rem;

  /* Inline (horizontal) */
  margin-inline: 1rem;    /* Equals: margin-inline-start + margin-inline-end */

  /* Block (vertical) */
  margin-block: 2rem;     /* Equals: margin-block-start + margin-block-end */

  /* Padding shorthand */
  padding-inline: 1.5rem 2rem;
}
```

### Browser Support for Logical Properties

```javascript
// Check which properties are supported
const logicalPropertySupport = {
  margin_inline: 'Chrome 69+, Firefox 41+, Safari 12.1+',
  padding_inline: 'Chrome 69+, Firefox 41+, Safari 12.1+',
  border_inline: 'Chrome 69+, Firefox 41+, Safari 12.1+',
  inset: 'Modern browsers',

  // Fallback for older browsers
  fallback_needed: 'IE11 and older browsers'
};

// Polyfill approach for older browsers
function setLogicalMargin(element, start, end) {
  const dir = document.documentElement.dir;
  if (dir === 'rtl') {
    element.style.marginRight = start;
    element.style.marginLeft = end;
  } else {
    element.style.marginLeft = start;
    element.style.marginRight = end;
  }
}
```

---

## Flexbox and Grid in RTL

### Flexbox in RTL

```css
/* Flexbox automatically adjusts in RTL */
.flex-container {
  display: flex;
  gap: 1rem;
}

/* LTR: items flow left to right */
/* RTL: items flow right to left (automatic!) */

/* Example with direction awareness */
.navbar {
  display: flex;
  justify-content: space-between;
  /* In LTR: logo on left, menu on right */
  /* In RTL: logo on right, menu on left (flipped!) */
}

/* Explicit flex direction */
.flex-row {
  display: flex;
  flex-direction: row;  /* Default, respects direction */
}

.flex-row-reverse {
  display: flex;
  flex-direction: row-reverse;  /* Reverses order */
}

/* Important: flex-start and flex-end are logical! */
.flex-alignment {
  display: flex;
  justify-content: flex-start;  /* Start in LTR, start in RTL */

  /* NOT flex-left/flex-right! */
  /* flex-start adapts to direction */
}

/* Alignment with align-items */
.flex-center {
  display: flex;
  align-items: center;    /* Works the same in LTR/RTL */
  justify-content: center;
}

/* Gap: works universally */
.flex-with-gap {
  display: flex;
  gap: 1rem;  /* Consistent spacing in both directions */
}
```

### CSS Grid in RTL

```css
/* Grid respects direction automatically */
.grid-container {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  gap: 1rem;
}

/* LTR: First column on left, last on right */
/* RTL: First column on right, last on left (automatic!) */

/* Named grid areas: direction-aware! */
.grid-with-areas {
  display: grid;
  grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";
}

/* In LTR: sidebar on left, main on right */
/* In RTL: sidebar on right, main on left (automatic!) */

/* Grid column numbers: WATCH OUT! */
.grid-item {
  grid-column: 1 / 3;  /* Starts at column 1 in both LTR and RTL */
  /* But visual position changes! */
}

/* Better: Use named areas instead of numbers */
.main-content {
  grid-area: main;  /* Works correctly in both directions */
}

/* Logical start/end values */
.grid-placement {
  grid-column-start: 1;
  /* Using numbers can be confusing in RTL */

  /* Better to use named areas or logical properties */
}
```

### Real-World Layout Examples

```jsx
// React component with responsive RTL layout
import React from 'react';
import styles from './layout.module.css';

export function ResponsiveLayout() {
  return (
    <div className={styles.container}>
      <aside className={styles.sidebar}>
        {/* Sidebar appears on start (left in LTR, right in RTL) */}
      </aside>
      <main className={styles.main}>
        {/* Main content in center */}
      </main>
    </div>
  );
}

/* layout.module.css */
.container {
  display: grid;
  grid-template-areas:
    "sidebar main";
  grid-template-columns: 250px 1fr;
  gap: 2rem;
  padding-inline: 1rem;  /* Logical padding */
}

.sidebar {
  grid-area: sidebar;
  padding-inline: 1rem;      /* Logical padding */
  border-inline-end: 1px solid #ddd;  /* Border on end */
}

.main {
  grid-area: main;
  padding-inline: 2rem;
}

/* Mobile: Stack vertically (works in both RTL/LTR) */
@media (max-width: 768px) {
  .container {
    grid-template-areas:
      "sidebar"
      "main";
    grid-template-columns: 1fr;
  }

  .sidebar {
    border-inline-end: none;
    border-block-end: 1px solid #ddd;
  }
}
```

---

## Icons and Images

### Icon Mirroring Strategy

```javascript
// Some icons need mirroring in RTL, others don't

const iconMirroringRules = {
  // Icons that SHOULD be mirrored in RTL
  mirror: [
    'arrow-left', 'arrow-right',     // Direction indicators
    'chevron-left', 'chevron-right',
    'bullet-list-indent',
    'quote-left', 'quote-right',
    'redo', 'undo',                 // Directional actions
    'reply', 'share',               // Depend on direction
    'back-arrow', 'forward-arrow'
  ],

  // Icons that should NOT be mirrored
  noMirror: [
    'home', 'settings', 'user',     // Universal symbols
    'search', 'close', 'menu',
    'star', 'heart', 'check',
    'calendar', 'clock', 'email',
    'save', 'delete', 'edit',
    'languages', 'translate'        // The translation icon itself!
  ],

  // Context-dependent
  contextual: [
    'arrow-up', 'arrow-down',       // Never mirror (vertical)
    'arrows-expand', 'arrows-collapse',
    'alignment-left', 'alignment-right',  // Mirroring depends on intent
    'image-icons'                   // May contain directional elements
  ]
};
```

### CSS Transform for Icon Mirroring

```css
/* Approach 1: CSS transform (most efficient) */
[dir="rtl"] .icon-arrow-left {
  transform: scaleX(-1);  /* Flip horizontally */
}

[dir="rtl"] .icon-chevron-right {
  transform: scaleX(-1);
}

/* Approach 2: Using logical direction */
.icon-next {
  /* Icon points to next, which is right in LTR, left in RTL */
}

[dir="rtl"] .icon-next {
  transform: scaleX(-1);
}

/* Approach 3: Separate icon files for RTL */
[dir="rtl"] .icon {
  background-image: url('/icons/rtl/icon-name.svg');
}

[dir="ltr"] .icon {
  background-image: url('/icons/ltr/icon-name.svg');
}

/* Best practice: Utility class */
.icon--mirror {
  [dir="rtl"] & {
    transform: scaleX(-1);
  }
}

.icon--no-mirror {
  /* No transform */
}
```

### React Component for Icons

```jsx
import React from 'react';

// Icon component with RTL awareness
function Icon({ name, mirror = false, className = '' }) {
  const dir = document.documentElement.dir;
  const shouldMirror = mirror && dir === 'rtl';

  return (
    <svg
      className={`icon icon--${name} ${shouldMirror ? 'icon--mirrored' : ''} ${className}`}
      style={shouldMirror ? { transform: 'scaleX(-1)' } : {}}
    >
      <use href={`#icon-${name}`} />
    </svg>
  );
}

// Usage
<Icon name="arrow-left" mirror={true} />
<Icon name="star" mirror={false} />
<Icon name="home" />  {/* No mirroring */}

// In a button
<button>
  <Icon name="arrow-right" mirror={true} />
  Next
</button>
// In RTL: arrow points left, button content right-aligned

// Icon configuration map
const iconConfig = {
  'arrow-left': { mirror: true },
  'arrow-right': { mirror: true },
  'home': { mirror: false },
  'star': { mirror: false },
  'menu': { mirror: false },
  'settings': { mirror: false }
};

// Utility: Get mirroring config
function shouldMirrorIcon(iconName) {
  return iconConfig[iconName]?.mirror ?? false;
}
```

### Image Considerations

```html
<!-- Text in images should be mirrored in RTL -->
<!-- Use CSS transforms or separate images -->

<!-- WRONG: Same image for LTR and RTL -->
<img src="/banner.jpg" alt="Welcome" />
<!-- If banner has LTR text, looks wrong in RTL -->

<!-- RIGHT: Direction-aware image -->
<picture>
  <source media="(dir: rtl)" srcset="/banner-rtl.jpg" />
  <source media="(dir: ltr)" srcset="/banner-ltr.jpg" />
  <img src="/banner-ltr.jpg" alt="Welcome" />
</picture>

<!-- Alternative: CSS background-image with transform -->
<div
  className="banner"
  style={
    dir === 'rtl' ? { backgroundImage: 'url(...)', transform: 'scaleX(-1)' } : {}
  }
/>

<!-- Text overlays: Especially important -->
<!-- Move text position based on direction -->
<div className="card-with-image">
  <img src="image.jpg" alt="..." />
  <p className="caption">Image caption</p>
  {/* CSS positions this with logical properties */}
</div>
```

---

## Component-Level RTL

### React Component Pattern

```jsx
import React, { useContext } from 'react';

// RTL context
const RTLContext = React.createContext({ isRTL: false, dir: 'ltr' });

export function RTLProvider({ children }) {
  const dir = document.documentElement.dir;
  const isRTL = dir === 'rtl';

  return (
    <RTLContext.Provider value={{ isRTL, dir }}>
      {children}
    </RTLContext.Provider>
  );
}

export function useRTL() {
  return useContext(RTLContext);
}

// Component using RTL context
function Sidebar({ children }) {
  const { isRTL } = useRTL();

  return (
    <aside
      className="sidebar"
      style={{
        marginInlineStart: isRTL ? 'auto' : 0,
        marginInlineEnd: isRTL ? 0 : 'auto'
      }}
    >
      {children}
    </aside>
  );
}

// Component with mirror-aware icons
function Navigation() {
  const { isRTL } = useRTL();

  return (
    <nav className="nav">
      <button className="nav-item">
        <Icon
          name={isRTL ? 'arrow-right' : 'arrow-left'}
          mirror={false}
        />
        Back
      </button>
    </nav>
  );
}

// Conditional rendering approach
function Menu({ items }) {
  const { isRTL } = useRTL();

  return (
    <ul className={`menu ${isRTL ? 'menu--rtl' : 'menu--ltr'}`}>
      {items.map(item => (
        <li key={item.id} className="menu-item">
          {isRTL && item.icon && <Icon name={item.icon} mirror={true} />}
          <span>{item.label}</span>
          {!isRTL && item.icon && <Icon name={item.icon} mirror={false} />}
        </li>
      ))}
    </ul>
  );
}
```

### CSS Module Approach

```css
/* Create separate classes for LTR and RTL */
/* styles.module.css */

.container {
  /* Base styles that work in both directions */
  padding: 1rem;
  display: flex;
}

/* LTR-specific styles */
.container[dir="ltr"] {
  flex-direction: row;
}

.sidebar--ltr {
  margin-left: 2rem;
  border-left: 3px solid #333;
}

/* RTL-specific styles */
.container[dir="rtl"] {
  flex-direction: row-reverse;
}

.sidebar--rtl {
  margin-right: 2rem;
  border-right: 3px solid #333;
}

/* Better: Use logical properties (works in both) */
.container-logical {
  display: flex;
  padding: 1rem;
  gap: 1rem;
}

.sidebar-logical {
  margin-inline-start: 2rem;
  border-inline-start: 3px solid #333;
}
```

---

## Testing RTL Layouts

### Manual Testing Strategy

```javascript
// Testing checklist for RTL support
const rtlTestingChecklist = {
  structure: [
    '¡ HTML has dir="rtl" and lang="ar-SA" (or appropriate)',
    '¡ All text content displays right-to-left',
    '¡ Numbers display correctly (still left-to-right within RTL)',
    '¡ Mixed LTR/RTL content displays correctly'
  ],

  layout: [
    '¡ Sidebars appear on correct side',
    '¡ Navigation items flow in correct direction',
    '¡ Form inputs align correctly',
    '¡ Modal dialogs display correctly',
    '¡ Dropdowns position correctly'
  ],

  icons: [
    '¡ Directional icons are mirrored',
    '¡ Non-directional icons are NOT mirrored',
    '¡ Icon positions are correct'
  ],

  images: [
    '¡ Images with text are either mirrored or replaced',
    '¡ Image captions display in correct direction'
  ],

  spacing: [
    '¡ No margin-left/right used (use logical properties)',
    '¡ Padding is consistent',
    '¡ Gaps in flexbox work correctly'
  ],

  accessibility: [
    '¡ Screen reader announces correct language',
    '¡ Tab order is correct for RTL',
    '¡ Focus visible states display correctly'
  ]
};
```

### Automated Testing with Cypress

```javascript
// cypress/e2e/rtl.cy.js
describe('RTL Language Support', () => {
  beforeEach(() => {
    cy.visit('/?lang=ar');  // Visit with Arabic language
    cy.get('html').should('have.attr', 'dir', 'rtl');
  });

  it('displays content in RTL', () => {
    cy.get('body').should('have.css', 'direction', 'rtl');
  });

  it('positions sidebar on correct side', () => {
    cy.get('.sidebar')
      .should('have.css', 'position', 'fixed')
      .should(($el) => {
        const rect = $el[0].getBoundingClientRect();
        // In RTL, sidebar should be on the right (high left value)
        expect(rect.right).to.be.greaterThan(window.innerWidth - 300);
      });
  });

  it('mirrors directional icons', () => {
    cy.get('.icon--arrow-right').should(($el) => {
      const transform = window.getComputedStyle($el[0]).transform;
      expect(transform).to.include('scaleX(-1)');
    });
  });

  it('does not mirror non-directional icons', () => {
    cy.get('.icon--home').should(($el) => {
      const transform = window.getComputedStyle($el[0]).transform;
      expect(transform).not.to.include('scaleX(-1)');
    });
  });

  it('handles form inputs correctly', () => {
    cy.get('input[type="text"]')
      .should('have.css', 'direction', 'rtl')
      .should('have.css', 'text-align', 'right');
  });

  it('uses logical CSS properties', () => {
    cy.get('.container').should(($el) => {
      const styles = window.getComputedStyle($el[0]);
      const marginInlineStart = styles.marginInlineStart;
      expect(marginInlineStart).to.exist;
    });
  });

  it('maintains readability with numbers', () => {
    cy.get('[data-test="price"]').should('contain', '1,234.56');
    // Number should still be readable despite RTL text
  });
});
```

### Visual Regression Testing

```javascript
// Detect layout differences between LTR and RTL
import { test, expect } from '@playwright/test';

test.describe('RTL Visual Regression', () => {
  test('LTR layout screenshot', async ({ page }) => {
    await page.goto('/?lang=en');
    await expect(page).toHaveScreenshot('layout-ltr.png');
  });

  test('RTL layout screenshot', async ({ page }) => {
    await page.goto('/?lang=ar');
    await expect(page).toHaveScreenshot('layout-rtl.png');
  });

  // Compare critical components
  test('header layout RTL', async ({ page }) => {
    await page.goto('/?lang=ar');
    await expect(page.locator('header')).toHaveScreenshot('header-rtl.png');
  });

  test('sidebar positioning RTL', async ({ page }) => {
    await page.goto('/?lang=ar');
    const sidebar = page.locator('.sidebar');
    const sidebarBox = await sidebar.boundingBox();

    // Sidebar should be on the right (left > window.width/2)
    expect(sidebarBox.x).toBeGreaterThan(page.viewportSize().width / 2);
  });
});
```

---

## Common Mistakes

### 1. Forgetting dir Attribute

```jsx
// WRONG: No dir attribute
function App() {
  return (
    <html>  {/* Missing dir! */}
      <body>Content</body>
    </html>
  );
}

// RIGHT: Add dir attribute
function App() {
  const dir = i18n.language.startsWith('ar') ? 'rtl' : 'ltr';
  return (
    <html dir={dir} lang={i18n.language}>
      <body>Content</body>
    </html>
  );
}
```

### 2. Using Physical Properties Instead of Logical

```css
/* WRONG: Physical properties break in RTL */
.sidebar {
  margin-left: 20px;    /* Appears on wrong side! */
  border-right: 1px solid black;  /* Wrong position! */
  padding-left: 10px;
}

/* RIGHT: Logical properties work in both */
.sidebar {
  margin-inline-start: 20px;
  border-inline-end: 1px solid black;
  padding-inline-start: 10px;
}
```

### 3. Not Testing with Real RTL Content

```jsx
// WRONG: Using English text as placeholder
function Component() {
  return <div dir="rtl">English text</div>;
}

// RIGHT: Test with actual RTL content
function Component() {
  return <div dir="rtl">E-*HI ('D91(J)</div>;
  // Arabic text is longer and wraps differently!
}
```

### 4. Mirroring Everything

```css
/* WRONG: Mirroring non-directional icons */
[dir="rtl"] .icon-home {
  transform: scaleX(-1);  /* Home icon looks weird flipped! */
}

[dir="rtl"] .icon-heart {
  transform: scaleX(-1);  /* Heart looks the same flipped */
}

/* RIGHT: Only mirror directional icons */
[dir="rtl"] .icon-arrow-right {
  transform: scaleX(-1);  /* Makes sense flipped */
}

[dir="rtl"] .icon-home {
  /* Don't mirror */
}
```

### 5. Not Handling Bidirectional Text

```jsx
// WRONG: Assuming simple RTL
function UserProfile({ name, message }) {
  return (
    <div dir="rtl">
      <h1>{name}</h1>  {/* Might be Arabic name */}
      <p>{message}</p> {/* Might contain English mixed with Arabic */}
    </div>
  );
}

// RIGHT: Let browser handle bidirectional text
function UserProfile({ name, message }) {
  return (
    <div dir="rtl">
      <h1>{name}</h1>
      <p>{message}</p>
      {/* Browser's bidi algorithm handles mixed text */}
    </div>
  );
  // Example: "E1-(' Hello E1-('" renders correctly
}
```

---

## Performance Considerations

### 1. Avoid Repeated Direction Checks

```jsx
// WRONG: Checking direction on every render
function Button() {
  const isRTL = document.documentElement.dir === 'rtl';
  return <button style={{ marginLeft: isRTL ? 'auto' : 0 }}>Click</button>;
}

// RIGHT: Use context and memoization
function Button() {
  const { isRTL } = useRTL();
  return <button style={{ marginLeft: isRTL ? 'auto' : 0 }}>Click</button>;
}

// Even better: Use CSS
const styles = css`
  button {
    margin-inline-start: auto;  /* No JavaScript needed! */
  }
`;
```

### 2. Optimize Icon Loading

```javascript
// WRONG: Load separate icons for each direction
const leftIcon = require('./icons/left.svg');
const rightIcon = require('./icons/right.svg');

// RIGHT: Use transforms instead
const styles = {
  icon: {
    backgroundImage: 'url(./icons/arrow.svg)'
  },
  iconRTL: {
    transform: 'scaleX(-1)'
  }
};
```

### 3. Bundle Size Considerations

```javascript
// WRONG: Include separate RTL CSS files
// index.html
<link rel="stylesheet" href="style-ltr.css" />
<link rel="stylesheet" href="style-rtl.css" />
// Duplicates CSS!

// RIGHT: Use single CSS file with direction awareness
// style.css
.container {
  margin-inline-start: 1rem;  /* Works for both */
}

// Or use CSS variables
[dir="ltr"] {
  --start: left;
  --end: right;
}

[dir="rtl"] {
  --start: right;
  --end: left;
}

.container {
  margin-left: var(--start);
}
```

---

## Interview Questions

### 1. How do you implement RTL support in a React app?

**Answer:**
1. Set `dir="rtl"` on the HTML element and match `lang` attribute
2. Use CSS logical properties instead of physical (margin-inline-start vs margin-left)
3. Mirror directional icons with CSS transforms
4. Test with actual RTL content
5. Use React context for RTL state across components

### 2. What's the difference between the dir attribute and CSS direction property?

**Answer:**
- **dir attribute**: Sets text directionality at HTML level, affects DOM layout order
- **CSS direction property**: Visual property, only affects how text flows
- Both are needed for proper RTL support

### 3. Which icons should be mirrored in RTL?

**Answer:**
Mirror directional icons:
- Arrow left/right
- Chevron left/right
- Navigation arrows

Don't mirror universal icons:
- Home, settings, search
- Star, heart, check
- These look the same or better without mirroring

### 4. What are logical CSS properties and why should you use them?

**Answer:**
Logical properties like `margin-inline-start` automatically adapt to text direction. `margin-inline-start` means "left in LTR, right in RTL". This eliminates the need to write separate styles for each direction.

### 5. How do you handle mixed LTR/RTL content (like Arabic text with English words)?

**Answer:**
Let the browser handle it automatically. The Unicode Bidirectional Algorithm (bidi algorithm) built into browsers correctly handles mixed-direction text. Just ensure your HTML has the correct `dir` attribute and `lang` attribute.

### 6. How would you test RTL layouts comprehensively?

**Answer:**
1. Manual testing: Check layout with actual RTL content
2. Verify sidebar position, icon mirroring, form alignment
3. Use Cypress/Playwright for automated testing
4. Visual regression testing to catch layout changes
5. Test with screen readers to ensure accessibility

### 7. What's the performance impact of RTL support?

**Answer:**
Minimal if done correctly:
- Use CSS logical properties (same CSS file for both)
- Avoid duplicate CSS files
- Use CSS transforms for icons (not separate images)
- RTL context through React Context (not prop drilling)
- One set of tests covers both directions

### 8. How do you detect the user's language preference for RTL?

**Answer:**
1. Check localStorage (user preference)
2. Check URL parameter or path
3. Check browser language (navigator.language)
4. Fall back to default

Then set `dir` on HTML element based on whether language is RTL.

### 9. What are the CSS logical properties you use most?

**Answer:**
- `margin-inline-start/end` (left/right)
- `margin-block-start/end` (top/bottom)
- `padding-inline-start/end`
- `border-inline-start/end`
- `text-align: start/end`
- `inset-inline-start/end`

### 10. How would you handle an image with text that needs to work in both LTR and RTL?

**Answer:**
Options:
1. Create separate images for each direction
2. Use CSS transform: `scaleX(-1)` in RTL (if appropriate)
3. Extract text from image and translate it
4. Use SVG with CSS direction-aware transforms

Best approach depends on content type - text content should be translated, directional graphics can be transformed.

---

## Navigation

**Complete i18n Series:**
- [01 - i18n Fundamentals](./01-i18n-fundamentals.md) - Core i18n concepts
- [02 - Pluralization](./02-pluralization.md) - Handle plural forms
- [03 - Date & Number Formatting](./03-date-number-formatting.md) - Intl APIs
- [04 - RTL Support](./04-rtl-support.md) - Current document

**Related Topics:**
- Frontend/React - Component patterns
- Frontend/CSSArchitecture - CSS best practices
- Frontend/BrowserAPIs - Browser APIs
- Frontend/Testing - Testing strategies

---

**Last Updated:** December 2025
**Difficulty:** Intermediate to Advanced
**Estimated Time:** 3-4 hours

---

## Summary: RTL Support Checklist

- [ ] Set `dir="rtl"` and matching `lang` attribute on HTML
- [ ] Use CSS logical properties throughout (no margin-left/right)
- [ ] Test with actual RTL content (not English as placeholder)
- [ ] Mirror only directional icons (not universal icons)
- [ ] Handle bidirectional text (let browser's bidi algorithm work)
- [ ] Test flexbox and grid layouts in RTL
- [ ] Create React components with RTL awareness
- [ ] Implement proper testing strategy
- [ ] Verify with actual RTL users if possible
- [ ] Optimize for performance (single CSS file, no duplicates)
