# Responsive Design

Responsive design ensures your website works well on all devices - from mobile phones to large desktop monitors.

## 📚 Core Concepts

### 1. Viewport Meta Tag

```html
<!-- Essential for responsive design -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### 2. Media Queries

**Basic Syntax**
```css
/* Mobile first (min-width) */
.container {
    width: 100%;
}

@media (min-width: 768px) {
    .container {
        width: 750px;
    }
}

@media (min-width: 1024px) {
    .container {
        width: 980px;
    }
}

/* Desktop first (max-width) */
.container {
    width: 1200px;
}

@media (max-width: 1024px) {
    .container {
        width: 100%;
    }
}
```

**Common Breakpoints**
```css
/* Extra small devices (phones, 600px and down) */
@media only screen and (max-width: 600px) {...}

/* Small devices (portrait tablets and large phones, 600px and up) */
@media only screen and (min-width: 600px) {...}

/* Medium devices (landscape tablets, 768px and up) */
@media only screen and (min-width: 768px) {...}

/* Large devices (laptops/desktops, 992px and up) */
@media only screen and (min-width: 992px) {...}

/* Extra large devices (large laptops and desktops, 1200px and up) */
@media only screen and (min-width: 1200px) {...}
```

**Media Features**
```css
/* Width and height */
@media (min-width: 768px) and (max-width: 1024px) {}

/* Orientation */
@media (orientation: portrait) {}
@media (orientation: landscape) {}

/* Resolution */
@media (min-resolution: 192dpi) {}

/* Hover capability */
@media (hover: hover) {
    .button:hover {
        background: blue;
    }
}

/* Pointer type */
@media (pointer: coarse) {
    /* Touch devices - make buttons larger */
    button {
        min-height: 44px;
    }
}

/* Prefers color scheme */
@media (prefers-color-scheme: dark) {
    body {
        background: #1a1a1a;
        color: #ffffff;
    }
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
    * {
        animation: none !important;
        transition: none !important;
    }
}
```

### 3. Fluid Layouts

**Relative Units**
```css
.container {
    width: 90%;
    max-width: 1200px;
    margin: 0 auto;
}

.column {
    width: 50%; /* Percentage */
    padding: 2rem; /* rem */
}

.text {
    font-size: clamp(1rem, 2.5vw, 2rem); /* Fluid typography */
}
```

**CSS Grid Responsive**
```css
.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
}
```

**Flexbox Responsive**
```css
.flex-container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
}

.flex-item {
    flex: 1 1 300px; /* grow shrink basis */
}
```

### 4. Responsive Images

**Basic Responsive Image**
```css
img {
    max-width: 100%;
    height: auto;
}
```

**Picture Element**
```html
<picture>
    <source media="(min-width: 1200px)" srcset="large.jpg">
    <source media="(min-width: 768px)" srcset="medium.jpg">
    <img src="small.jpg" alt="Responsive image">
</picture>
```

**Srcset**
```html
<!-- Responsive images based on screen density -->
<img
    src="image-1x.jpg"
    srcset="image-1x.jpg 1x, image-2x.jpg 2x, image-3x.jpg 3x"
    alt="High DPI image">

<!-- Responsive images based on viewport width -->
<img
    src="small.jpg"
    srcset="small.jpg 400w, medium.jpg 800w, large.jpg 1200w"
    sizes="(max-width: 600px) 100vw, (max-width: 1000px) 50vw, 33vw"
    alt="Responsive widths">
```

**Background Images**
```css
.hero {
    background-image: url('small.jpg');
}

@media (min-width: 768px) {
    .hero {
        background-image: url('medium.jpg');
    }
}

@media (min-width: 1200px) {
    .hero {
        background-image: url('large.jpg');
    }
}
```

### 5. Container Queries (Modern)

```css
.card-container {
    container-type: inline-size;
    container-name: card;
}

@container card (min-width: 400px) {
    .card {
        display: grid;
        grid-template-columns: 1fr 2fr;
    }
}
```

### 6. Mobile-First vs Desktop-First

**Mobile-First (Recommended)**
```css
/* Base styles for mobile */
.nav {
    flex-direction: column;
}

/* Enhance for larger screens */
@media (min-width: 768px) {
    .nav {
        flex-direction: row;
    }
}
```

**Desktop-First**
```css
/* Base styles for desktop */
.nav {
    flex-direction: row;
}

/* Override for smaller screens */
@media (max-width: 767px) {
    .nav {
        flex-direction: column;
    }
}
```

## 💡 Practical Examples

### Example 1: Responsive Navigation

```html
<nav class="navbar">
    <div class="logo">Logo</div>
    <button class="menu-toggle">☰</button>
    <ul class="nav-links">
        <li><a href="#home">Home</a></li>
        <li><a href="#about">About</a></li>
        <li><a href="#contact">Contact</a></li>
    </ul>
</nav>
```

```css
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
}

.menu-toggle {
    display: none;
}

.nav-links {
    display: flex;
    gap: 2rem;
    list-style: none;
}

/* Mobile */
@media (max-width: 768px) {
    .menu-toggle {
        display: block;
    }

    .nav-links {
        display: none;
        flex-direction: column;
        width: 100%;
        position: absolute;
        top: 60px;
        left: 0;
        background: white;
    }

    .nav-links.active {
        display: flex;
    }
}
```

### Example 2: Responsive Card Grid

```css
.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 2rem;
    padding: 2rem;
}

.card {
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.card img {
    width: 100%;
    height: 200px;
    object-fit: cover;
}

.card-content {
    padding: 1.5rem;
}

@media (max-width: 600px) {
    .card-grid {
        grid-template-columns: 1fr;
        padding: 1rem;
    }
}
```

### Example 3: Responsive Typography

```css
:root {
    --font-size-small: clamp(0.875rem, 1.5vw, 1rem);
    --font-size-base: clamp(1rem, 2vw, 1.125rem);
    --font-size-large: clamp(1.25rem, 3vw, 1.5rem);
    --font-size-xlarge: clamp(2rem, 5vw, 3rem);
}

body {
    font-size: var(--font-size-base);
    line-height: 1.6;
}

h1 {
    font-size: var(--font-size-xlarge);
    line-height: 1.2;
}

h2 {
    font-size: var(--font-size-large);
}

small {
    font-size: var(--font-size-small);
}
```

### Example 4: Responsive Dashboard

```css
.dashboard {
    display: grid;
    grid-template-columns: 250px 1fr;
    grid-template-rows: 60px 1fr;
    grid-template-areas:
        "sidebar header"
        "sidebar main";
    height: 100vh;
    gap: 20px;
}

.sidebar { grid-area: sidebar; }
.header { grid-area: header; }
.main { grid-area: main; }

@media (max-width: 1024px) {
    .dashboard {
        grid-template-columns: 1fr;
        grid-template-rows: 60px auto 1fr;
        grid-template-areas:
            "header"
            "sidebar"
            "main";
    }

    .sidebar {
        height: auto;
    }
}
```

## 🎯 Common Interview Questions

### Q1: What is mobile-first design and why use it?

**Answer:**
Mobile-first means designing for mobile devices first, then progressively enhancing for larger screens.

**Benefits:**
- Better performance on mobile (no unused CSS)
- Forces prioritization of content
- Easier to scale up than down
- Better for majority mobile users

### Q2: What are common responsive breakpoints?

**Answer:**
```css
/* Common breakpoints */
- 320px: Small phones
- 480px: Phones
- 768px: Tablets
- 1024px: Small laptops
- 1200px: Desktops
- 1440px+: Large screens
```

### Q3: How do you make images responsive?

**Answer:**
```css
/* Basic */
img {
    max-width: 100%;
    height: auto;
}

/* Advanced */
<picture>
    <source media="(min-width: 768px)" srcset="large.jpg">
    <img src="small.jpg" alt="Responsive">
</picture>
```

## 🚨 Common Pitfalls

1. **Forgetting viewport meta tag**
2. **Not testing on real devices**
3. **Fixed widths instead of percentages**
4. **Ignoring touch targets** (min 44x44px)
5. **Not considering landscape orientation**
6. **Too many breakpoints** (keep it simple)

## 🎓 Best Practices

1. **Use mobile-first approach**
2. **Test on real devices**, not just DevTools
3. **Use relative units** (rem, em, %)
4. **Touch targets min 44x44px**
5. **Optimize images** for different screens
6. **Use system fonts** for better performance
7. **Consider accessibility** (color contrast, font size)
8. **Test with slow network**

## 🔗 Related Topics

- [CSS Fundamentals](./02-css-fundamentals.md)
- [Flexbox](./03-flexbox.md)
- [Grid](./04-grid.md)

---

[← Back to HTML & CSS](./README.md)
