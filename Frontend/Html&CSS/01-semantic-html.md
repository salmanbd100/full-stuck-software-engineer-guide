# Semantic HTML

Semantic HTML uses meaningful elements that describe their content, improving accessibility, SEO, and code readability.

## 📚 Core Concepts

### 1. What is Semantic HTML?

Semantic HTML uses elements that clearly describe their meaning to both the browser and developers.

```html
<!-- Non-semantic -->
<div class="header">
    <div class="nav">...</div>
</div>
<div class="main">
    <div class="article">...</div>
</div>
<div class="footer">...</div>

<!-- Semantic -->
<header>
    <nav>...</nav>
</header>
<main>
    <article>...</article>
</main>
<footer>...</footer>
```

### 2. Document Structure Elements

**`<header>` - Introductory Content**
```html
<!-- Page header -->
<header>
    <h1>Company Name</h1>
    <nav>
        <a href="#home">Home</a>
        <a href="#about">About</a>
    </nav>
</header>

<!-- Article header -->
<article>
    <header>
        <h2>Article Title</h2>
        <p>By John Doe | Published: Jan 1, 2024</p>
    </header>
    <p>Article content...</p>
</article>
```

**`<nav>` - Navigation Links**
```html
<nav aria-label="Main navigation">
    <ul>
        <li><a href="#home">Home</a></li>
        <li><a href="#services">Services</a></li>
        <li><a href="#contact">Contact</a></li>
    </ul>
</nav>

<!-- Breadcrumb navigation -->
<nav aria-label="Breadcrumb">
    <ol>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li aria-current="page">Laptops</li>
    </ol>
</nav>
```

**`<main>` - Main Content**
```html
<!-- Only one <main> per page -->
<main>
    <h1>Page Title</h1>
    <article>
        <h2>Article 1</h2>
        <p>Content...</p>
    </article>
    <article>
        <h2>Article 2</h2>
        <p>Content...</p>
    </article>
</main>
```

**`<article>` - Self-Contained Content**
```html
<!-- Blog post -->
<article>
    <header>
        <h2>Understanding JavaScript Closures</h2>
        <p>
            <time datetime="2024-01-15">January 15, 2024</time>
            by <span>Alice Johnson</span>
        </p>
    </header>

    <p>Introduction to closures...</p>

    <section>
        <h3>What are Closures?</h3>
        <p>Closures are...</p>
    </section>

    <footer>
        <p>Tags: JavaScript, Programming</p>
    </footer>
</article>

<!-- Product card -->
<article class="product-card">
    <h3>Product Name</h3>
    <img src="product.jpg" alt="Product description">
    <p>$99.99</p>
    <button>Add to Cart</button>
</article>
```

**`<section>` - Thematic Grouping**
```html
<main>
    <section>
        <h2>About Us</h2>
        <p>Company information...</p>
    </section>

    <section>
        <h2>Our Services</h2>
        <ul>
            <li>Service 1</li>
            <li>Service 2</li>
        </ul>
    </section>

    <section>
        <h2>Contact</h2>
        <form>...</form>
    </section>
</main>
```

**`<aside>` - Tangential Content**
```html
<main>
    <article>
        <h1>Main Article</h1>
        <p>Article content...</p>

        <!-- Related content -->
        <aside>
            <h3>Related Articles</h3>
            <ul>
                <li><a href="#">Related 1</a></li>
                <li><a href="#">Related 2</a></li>
            </ul>
        </aside>
    </article>
</main>

<!-- Sidebar -->
<aside class="sidebar">
    <section>
        <h2>Newsletter</h2>
        <form>...</form>
    </section>
    <section>
        <h2>Popular Posts</h2>
        <ul>...</ul>
    </section>
</aside>
```

**`<footer>` - Footer Content**
```html
<!-- Page footer -->
<footer>
    <nav aria-label="Footer navigation">
        <a href="#privacy">Privacy</a>
        <a href="#terms">Terms</a>
    </nav>
    <p>&copy; 2024 Company Name. All rights reserved.</p>
</footer>

<!-- Article footer -->
<article>
    <h2>Article Title</h2>
    <p>Content...</p>

    <footer>
        <p>Author: John Doe</p>
        <p>Tags: HTML, Semantic</p>
    </footer>
</article>
```

### 3. Text Content Elements

**Headings (`<h1>` - `<h6>`)**
```html
<!-- Proper heading hierarchy -->
<h1>Main Page Title</h1>
<section>
    <h2>Section 1</h2>
    <h3>Subsection 1.1</h3>
    <h3>Subsection 1.2</h3>
</section>
<section>
    <h2>Section 2</h2>
    <h3>Subsection 2.1</h3>
</section>

<!-- Bad: Skipping levels -->
<!-- <h1>Title</h1>
<h3>Should be h2</h3> -->
```

**`<p>` - Paragraphs**
```html
<p>This is a paragraph of text.</p>
<p>This is another paragraph.</p>

<!-- Not for layout! -->
<!-- Bad: <p><div>...</div></p> -->
```

**`<strong>` vs `<b>`**
```html
<!-- <strong> - Important (semantic) -->
<p>Warning: <strong>Do not</strong> delete system files.</p>

<!-- <b> - Visual only (non-semantic) -->
<p>The <b>first word</b> is bold.</p>
```

**`<em>` vs `<i>`**
```html
<!-- <em> - Emphasis (semantic) -->
<p>I <em>really</em> mean it.</p>

<!-- <i> - Visual/technical terms (non-semantic) -->
<p>The term <i>semantic HTML</i> refers to...</p>
```

**`<mark>` - Highlighted Text**
```html
<p>Search results for "JavaScript":</p>
<p>Learn <mark>JavaScript</mark> in 30 days.</p>
```

**`<small>` - Fine Print**
```html
<p>Price: $99.99 <small>per month</small></p>
<footer>
    <p>Copyright 2024</p>
    <small>Terms and conditions apply</small>
</footer>
```

**`<abbr>` - Abbreviations**
```html
<p>The <abbr title="World Wide Web">WWW</abbr> was invented in 1989.</p>
<p><abbr title="HyperText Markup Language">HTML</abbr> is not a programming language.</p>
```

**`<time>` - Dates and Times**
```html
<!-- Human and machine readable -->
<p>Published: <time datetime="2024-01-15">January 15, 2024</time></p>
<p>Event starts at <time datetime="2024-06-01T19:00">7:00 PM, June 1</time></p>
<p>Duration: <time datetime="PT2H30M">2 hours 30 minutes</time></p>
```

### 4. Lists

**Unordered List (`<ul>`)**
```html
<ul>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
</ul>
```

**Ordered List (`<ol>`)**
```html
<ol>
    <li>Step 1: Prepare ingredients</li>
    <li>Step 2: Mix ingredients</li>
    <li>Step 3: Bake for 30 minutes</li>
</ol>

<!-- Custom numbering -->
<ol start="5">
    <li>Item 5</li>
    <li>Item 6</li>
</ol>

<ol type="A">
    <li>Item A</li>
    <li>Item B</li>
</ol>
```

**Description List (`<dl>`)**
```html
<dl>
    <dt>HTML</dt>
    <dd>HyperText Markup Language</dd>

    <dt>CSS</dt>
    <dd>Cascading Style Sheets</dd>

    <dt>JavaScript</dt>
    <dd>Programming language for web</dd>
    <dd>Can run in browser and server</dd>
</dl>
```

### 5. Forms and Input

**Form with Semantic Structure**
```html
<form action="/submit" method="post">
    <fieldset>
        <legend>Personal Information</legend>

        <label for="name">Full Name:</label>
        <input type="text" id="name" name="name" required>

        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
    </fieldset>

    <fieldset>
        <legend>Preferences</legend>

        <label>
            <input type="checkbox" name="newsletter" value="yes">
            Subscribe to newsletter
        </label>
    </fieldset>

    <button type="submit">Submit</button>
</form>
```

**Input Types**
```html
<!-- Email -->
<input type="email" placeholder="user@example.com">

<!-- Tel -->
<input type="tel" pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}">

<!-- URL -->
<input type="url" placeholder="https://example.com">

<!-- Date -->
<input type="date" min="2024-01-01" max="2024-12-31">

<!-- Number -->
<input type="number" min="1" max="100" step="1">

<!-- Range -->
<input type="range" min="0" max="100" value="50">

<!-- Color -->
<input type="color" value="#ff0000">

<!-- Search -->
<input type="search" placeholder="Search...">
```

### 6. Media Elements

**`<figure>` and `<figcaption>`**
```html
<figure>
    <img src="chart.png" alt="Sales data for 2024">
    <figcaption>Figure 1: Annual sales growth</figcaption>
</figure>

<figure>
    <pre><code>
function greet(name) {
    return `Hello, ${name}!`;
}
    </code></pre>
    <figcaption>Example: JavaScript greeting function</figcaption>
</figure>
```

**`<picture>` - Responsive Images**
```html
<picture>
    <source media="(min-width: 1200px)" srcset="large.jpg">
    <source media="(min-width: 768px)" srcset="medium.jpg">
    <img src="small.jpg" alt="Responsive image">
</picture>
```

**`<audio>` and `<video>`**
```html
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    Your browser doesn't support audio.
</audio>

<video controls width="640" height="360">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    <track kind="subtitles" src="subtitles-en.vtt" srclang="en" label="English">
    Your browser doesn't support video.
</video>
```

### 7. Code and Quotations

**`<code>` - Inline Code**
```html
<p>Use the <code>console.log()</code> method to debug.</p>
```

**`<pre>` - Preformatted Text**
```html
<pre><code>
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
</code></pre>
```

**`<blockquote>` - Block Quote**
```html
<blockquote cite="https://example.com/quote">
    <p>The only way to do great work is to love what you do.</p>
    <footer>— Steve Jobs</footer>
</blockquote>
```

**`<q>` - Inline Quote**
```html
<p>As Einstein said, <q>Imagination is more important than knowledge.</q></p>
```

## 🎯 Common Interview Questions

### Q1: What's the difference between `<article>` and `<section>`?

**Answer:**
- **`<article>`**: Self-contained, reusable content (blog posts, products, comments)
- **`<section>`**: Thematic grouping of content (chapters, tabs)

```html
<!-- Good use of article -->
<article class="blog-post">
    <h2>Post Title</h2>
    <p>Content...</p>
</article>

<!-- Good use of section -->
<section>
    <h2>About Us</h2>
    <p>Company info...</p>
</section>
```

### Q2: When should you use `<div>` vs semantic elements?

**Answer:**
- Use semantic elements when meaning is important
- Use `<div>` only for styling/layout without semantic meaning

```html
<!-- Bad -->
<div class="navigation">...</div>

<!-- Good -->
<nav>...</nav>

<!-- OK (styling wrapper) -->
<div class="container">
    <header>...</header>
</div>
```

### Q3: Why is semantic HTML important?

**Answer:**
1. **Accessibility**: Screen readers understand content structure
2. **SEO**: Search engines better understand content
3. **Maintainability**: Code is easier to read and understand
4. **Future-proof**: Works better with new technologies

## 💡 Practical Examples

### Example 1: Blog Layout

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Blog</title>
</head>
<body>
    <header>
        <h1>My Tech Blog</h1>
        <nav aria-label="Main navigation">
            <ul>
                <li><a href="#home">Home</a></li>
                <li><a href="#about">About</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <article>
            <header>
                <h2>Understanding Semantic HTML</h2>
                <p>
                    By <span>Alice Johnson</span> on
                    <time datetime="2024-01-15">January 15, 2024</time>
                </p>
            </header>

            <section>
                <h3>Introduction</h3>
                <p>Semantic HTML is important because...</p>
            </section>

            <section>
                <h3>Benefits</h3>
                <ul>
                    <li>Better accessibility</li>
                    <li>Improved SEO</li>
                </ul>
            </section>

            <footer>
                <p>Tags: HTML, Web Development</p>
            </footer>
        </article>

        <aside>
            <h3>Related Posts</h3>
            <ul>
                <li><a href="#">CSS Best Practices</a></li>
                <li><a href="#">JavaScript Tips</a></li>
            </ul>
        </aside>
    </main>

    <footer>
        <p>&copy; 2024 My Blog. All rights reserved.</p>
    </footer>
</body>
</html>
```

### Example 2: E-commerce Product Page

```html
<main>
    <article class="product">
        <header>
            <h1>Wireless Headphones</h1>
        </header>

        <figure>
            <img src="headphones.jpg" alt="Black wireless headphones">
            <figcaption>Premium wireless headphones with noise cancellation</figcaption>
        </figure>

        <section>
            <h2>Product Details</h2>
            <dl>
                <dt>Brand</dt>
                <dd>TechBrand</dd>

                <dt>Model</dt>
                <dd>WH-1000XM4</dd>

                <dt>Price</dt>
                <dd>$349.99</dd>
            </dl>
        </section>

        <section>
            <h2>Description</h2>
            <p>Premium headphones with...</p>
        </section>

        <footer>
            <button type="button">Add to Cart</button>
        </footer>
    </article>
</main>
```

## 🚨 Common Pitfalls

1. **Using `<div>` for everything** - Use semantic elements when possible
2. **Multiple `<main>` elements** - Only one per page
3. **Skipping heading levels** - Maintain proper hierarchy (h1 → h2 → h3)
4. **Using `<br>` for spacing** - Use CSS margins instead
5. **Not using labels for inputs** - Always associate labels with inputs

## 🎓 Best Practices

1. **Use semantic elements** over generic divs
2. **Maintain heading hierarchy** (don't skip levels)
3. **Always include alt text** for images
4. **Use ARIA labels** when needed for clarity
5. **Associate labels with inputs** (for/id or wrap)
6. **One `<h1>` per page** (usually)
7. **Use `<button>` for actions**, `<a>` for navigation

## 🔗 Related Topics

- [Accessibility](./07-accessibility.md)
- [CSS Fundamentals](./02-css-fundamentals.md)

---

[← Back to HTML & CSS](./README.md) | [Next: CSS Fundamentals →](./02-css-fundamentals.md)
