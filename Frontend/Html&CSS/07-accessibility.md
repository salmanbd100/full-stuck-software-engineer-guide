# Accessibility (a11y)

Web accessibility ensures your website is usable by everyone, including people with disabilities. This is both a legal requirement and ethical responsibility.

## 📚 Core Principles (WCAG)

### 1. Perceivable

**Text Alternatives**
```html
<!-- Images -->
<img src="logo.png" alt="Company Logo">
<img src="decorative.png" alt="" role="presentation"> <!-- Decorative -->

<!-- Icons with text -->
<button>
    <svg aria-hidden="true">...</svg>
    <span>Delete</span>
</button>

<!-- Icons without text -->
<button aria-label="Close dialog">
    <svg aria-hidden="true"><use href="#close-icon"></use></svg>
</button>
```

**Color Contrast**
```css
/* Minimum contrast ratios (WCAG AA) */
/* Normal text: 4.5:1 */
/* Large text (18pt+): 3:1 */
/* UI components: 3:1 */

.good-contrast {
    background: #ffffff;
    color: #333333; /* 12.6:1 - Excellent! */
}

.bad-contrast {
    background: #ffffff;
    color: #cccccc; /* 1.6:1 - Fail! */
}

/* Don't rely on color alone */
.error {
    color: red; /* Bad */
    border-left: 3px solid red; /* Better */
}

.error::before {
    content: "❌ "; /* Even better */
}
```

**Responsive Text**
```css
/* Allow text resizing up to 200% */
body {
    font-size: 16px; /* Use rem, not px */
}

h1 {
    font-size: 2rem; /* Scales with user settings */
}

/* Don't disable zoom */
/* Bad: <meta name="viewport" content="user-scalable=no"> */
/* Good: <meta name="viewport" content="width=device-width, initial-scale=1.0"> */
```

### 2. Operable

**Keyboard Navigation**
```html
<!-- All interactive elements must be keyboard accessible -->
<button>I'm accessible by default</button>

<!-- Don't do this -->
<div onclick="handleClick()">Not keyboard accessible</div>

<!-- Do this instead -->
<button onclick="handleClick()">Keyboard accessible</button>

<!-- Custom interactive elements need tabindex -->
<div role="button" tabindex="0" onclick="..." onkeypress="...">
    Custom button
</div>
```

**Focus Indicators**
```css
/* Never remove focus outline without replacement */
button:focus {
    outline: none; /* Bad! */
}

/* Good: Provide alternative */
button:focus {
    outline: 2px solid #3498db;
    outline-offset: 2px;
}

/* Modern approach */
button:focus-visible {
    outline: 2px solid #3498db;
}

/* Skip to main content */
.skip-link {
    position: absolute;
    top: -40px;
    left: 0;
    background: #000;
    color: #fff;
    padding: 8px;
    text-decoration: none;
}

.skip-link:focus {
    top: 0;
}
```

**Tab Order**
```html
<!-- Natural tab order (best) -->
<form>
    <input type="text"> <!-- Tab 1 -->
    <input type="email"> <!-- Tab 2 -->
    <button>Submit</button> <!-- Tab 3 -->
</form>

<!-- Override tab order (use sparingly) -->
<input tabindex="2">
<input tabindex="1"> <!-- Focused first -->
<input tabindex="3">

<!-- Remove from tab order -->
<div tabindex="-1">Not keyboard accessible</div>
```

**Timing and Limits**
```html
<!-- Provide way to extend time limits -->
<div role="alert">
    Session expires in 2 minutes.
    <button>Extend session</button>
</div>

<!-- Pause auto-playing content -->
<video controls>...</video>

<!-- Provide way to stop animations -->
<button onclick="document.body.classList.toggle('reduced-motion')">
    Toggle animations
</button>
```

```css
@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}
```

### 3. Understandable

**Language**
```html
<html lang="en">

<p lang="es">Hola mundo</p>

<abbr title="World Wide Web">WWW</abbr>
```

**Form Labels and Instructions**
```html
<!-- Always associate labels -->
<label for="email">Email:</label>
<input type="email" id="email" name="email" required>

<!-- Or wrap -->
<label>
    Email:
    <input type="email" name="email" required>
</label>

<!-- Provide instructions -->
<label for="password">Password:</label>
<input type="password" id="password" aria-describedby="password-help">
<span id="password-help">Must be at least 8 characters</span>

<!-- Error messages -->
<input
    type="email"
    id="email"
    aria-invalid="true"
    aria-describedby="email-error">
<span id="email-error" role="alert">
    Please enter a valid email address
</span>
```

**Consistent Navigation**
```html
<!-- Keep navigation consistent across pages -->
<nav aria-label="Main navigation">
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>
```

### 4. Robust

**Semantic HTML**
```html
<!-- Good -->
<header>
    <nav>
        <ul>
            <li><a href="/">Home</a></li>
        </ul>
    </nav>
</header>

<main>
    <article>
        <h1>Article Title</h1>
        <p>Content...</p>
    </article>
</main>

<footer>
    <p>&copy; 2024</p>
</footer>

<!-- Bad -->
<div class="header">
    <div class="nav">
        <div class="link">Home</div>
    </div>
</div>
```

## 🎯 ARIA (Accessible Rich Internet Applications)

### ARIA Roles

```html
<!-- Landmark roles -->
<header role="banner">
<nav role="navigation">
<main role="main">
<aside role="complementary">
<footer role="contentinfo">
<form role="search">

<!-- Widget roles -->
<div role="button" tabindex="0">Custom Button</div>
<div role="checkbox" aria-checked="false" tabindex="0">Option</div>
<div role="tab" aria-selected="true">Tab 1</div>
<div role="tabpanel">Tab content</div>
<div role="dialog" aria-labelledby="dialog-title">
    <h2 id="dialog-title">Dialog Title</h2>
</div>

<!-- Live region roles -->
<div role="alert">Important message</div>
<div role="status">Status update</div>
<div aria-live="polite">Updates</div>
<div aria-live="assertive">Urgent updates</div>
```

### ARIA Attributes

```html
<!-- Labels -->
<button aria-label="Close">×</button>
<div aria-labelledby="title">
    <h2 id="title">Section Title</h2>
</div>

<!-- Descriptions -->
<button aria-describedby="help">Submit</button>
<span id="help">Submits the form</span>

<!-- States -->
<button aria-pressed="false">Toggle</button>
<input aria-invalid="true">
<div aria-expanded="false">Collapsed</div>
<div aria-hidden="true">Hidden from screen readers</div>
<div aria-disabled="true">Disabled</div>

<!-- Properties -->
<input aria-required="true">
<div aria-haspopup="true">Menu</div>
<div role="tablist">
    <button role="tab" aria-controls="panel1" aria-selected="true">Tab 1</button>
    <button role="tab" aria-controls="panel2" aria-selected="false">Tab 2</button>
</div>
<div role="tabpanel" id="panel1" aria-labelledby="tab1">...</div>
```

### Common Patterns

**Modal Dialog**
```html
<div
    role="dialog"
    aria-modal="true"
    aria-labelledby="dialog-title"
    aria-describedby="dialog-desc">

    <h2 id="dialog-title">Confirm Action</h2>
    <p id="dialog-desc">Are you sure?</p>

    <button>Confirm</button>
    <button>Cancel</button>
</div>
```

**Accordion**
```html
<div class="accordion">
    <h3>
        <button
            aria-expanded="false"
            aria-controls="section1">
            Section 1
        </button>
    </h3>
    <div id="section1" hidden>
        Content...
    </div>
</div>
```

**Dropdown Menu**
```html
<button
    aria-haspopup="true"
    aria-expanded="false"
    aria-controls="menu">
    Menu
</button>

<ul id="menu" role="menu" hidden>
    <li role="menuitem"><a href="#">Item 1</a></li>
    <li role="menuitem"><a href="#">Item 2</a></li>
</ul>
```

## 💡 Practical Examples

### Example 1: Accessible Form

```html
<form>
    <fieldset>
        <legend>Personal Information</legend>

        <div class="form-group">
            <label for="name">
                Full Name
                <span aria-label="required">*</span>
            </label>
            <input
                type="text"
                id="name"
                name="name"
                required
                aria-required="true">
        </div>

        <div class="form-group">
            <label for="email">Email</label>
            <input
                type="email"
                id="email"
                name="email"
                aria-describedby="email-help">
            <span id="email-help">We'll never share your email</span>
        </div>

        <div class="form-group">
            <span id="password-label">Password</span>
            <input
                type="password"
                id="password"
                aria-labelledby="password-label"
                aria-describedby="password-requirements">
            <ul id="password-requirements">
                <li>At least 8 characters</li>
                <li>One uppercase letter</li>
                <li>One number</li>
            </ul>
        </div>
    </fieldset>

    <button type="submit">Submit Form</button>
</form>
```

### Example 2: Accessible Navigation

```html
<nav aria-label="Main navigation">
    <ul>
        <li><a href="/" aria-current="page">Home</a></li>
        <li><a href="/about">About</a></li>
        <li>
            <button
                aria-expanded="false"
                aria-controls="services-menu">
                Services
            </button>
            <ul id="services-menu" hidden>
                <li><a href="/web-design">Web Design</a></li>
                <li><a href="/development">Development</a></li>
            </ul>
        </li>
    </ul>
</nav>

<!-- Breadcrumb -->
<nav aria-label="Breadcrumb">
    <ol>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li aria-current="page">Laptops</li>
    </ol>
</nav>
```

### Example 3: Accessible Data Table

```html
<table>
    <caption>Monthly Sales Report</caption>
    <thead>
        <tr>
            <th scope="col">Month</th>
            <th scope="col">Revenue</th>
            <th scope="col">Expenses</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">January</th>
            <td>$10,000</td>
            <td>$5,000</td>
        </tr>
        <tr>
            <th scope="row">February</th>
            <td>$12,000</td>
            <td>$6,000</td>
        </tr>
    </tbody>
</table>
```

## 🎯 Common Interview Questions

### Q1: What is ARIA and when should you use it?

**Answer:**
ARIA (Accessible Rich Internet Applications) provides extra semantics for assistive technologies. Use it when HTML alone isn't sufficient.

**First rule of ARIA: Don't use ARIA**
- Use semantic HTML when possible
- Only add ARIA when HTML can't express the pattern

```html
<!-- Good: No ARIA needed -->
<button>Click me</button>

<!-- Bad: Unnecessary ARIA -->
<button role="button">Click me</button>

<!-- Good: ARIA adds missing semantics -->
<div role="button" tabindex="0">Custom button</div>
```

### Q2: How do you make a custom dropdown accessible?

**Answer:**
```html
<div class="dropdown">
    <button
        id="menu-button"
        aria-haspopup="true"
        aria-expanded="false"
        aria-controls="menu-list">
        Options
    </button>

    <ul id="menu-list" role="menu" aria-labelledby="menu-button" hidden>
        <li role="menuitem"><button>Option 1</button></li>
        <li role="menuitem"><button>Option 2</button></li>
    </ul>
</div>
```

Plus keyboard support:
- Enter/Space: Open menu
- Esc: Close menu
- Arrow keys: Navigate items

### Q3: What's the difference between `aria-label` and `aria-labelledby`?

**Answer:**
- `aria-label`: Provides string label directly
- `aria-labelledby`: References another element's text

```html
<!-- aria-label -->
<button aria-label="Close dialog">×</button>

<!-- aria-labelledby -->
<section aria-labelledby="section-title">
    <h2 id="section-title">Recent Posts</h2>
</section>
```

## 🚨 Common Pitfalls

1. **Removing focus outlines** without replacement
2. **Using `<div>` for everything** instead of semantic HTML
3. **Missing alt text** on images
4. **Poor color contrast**
5. **Not testing with keyboard**
6. **Not testing with screen readers**
7. **Overusing ARIA** when HTML would suffice

## 🎓 Best Practices

1. **Use semantic HTML first**, ARIA second
2. **Test with keyboard** (Tab, Enter, Arrow keys, Esc)
3. **Test with screen readers** (NVDA, JAWS, VoiceOver)
4. **Maintain focus order** (logical tab sequence)
5. **Provide text alternatives** for all non-text content
6. **Ensure sufficient color contrast**
7. **Support browser zoom** up to 200%
8. **Add skip links** for keyboard users
9. **Test with automated tools** (Lighthouse, axe)
10. **Include real users with disabilities** in testing

## 🔗 Related Topics

- [Semantic HTML](./01-semantic-html.md)
- [Forms](./02-css-fundamentals.md)

---

[← Back to HTML & CSS](./README.md)
