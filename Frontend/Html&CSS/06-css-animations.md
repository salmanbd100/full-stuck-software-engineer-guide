# CSS Animations

CSS animations bring life to web pages, creating engaging user experiences without JavaScript.

## 📚 Core Concepts

### 1. Transitions

**Basic Transition**
```css
.button {
    background: blue;
    transition: background 0.3s ease;
}

.button:hover {
    background: darkblue;
}
```

**Transition Properties**
```css
.element {
    /* Property duration timing-function delay */
    transition: all 0.3s ease 0s;

    /* Multiple properties */
    transition: background 0.3s ease, transform 0.5s ease-in-out;

    /* Individual properties */
    transition-property: background, transform;
    transition-duration: 0.3s, 0.5s;
    transition-timing-function: ease, ease-in-out;
    transition-delay: 0s, 0.1s;
}
```

**Timing Functions**
```css
.element {
    transition: all 0.3s linear;        /* Constant speed */
    transition: all 0.3s ease;          /* Slow-fast-slow (default) */
    transition: all 0.3s ease-in;       /* Slow start */
    transition: all 0.3s ease-out;      /* Slow end */
    transition: all 0.3s ease-in-out;   /* Slow start and end */

    /* Custom cubic-bezier */
    transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);

    /* Steps */
    transition: all 0.3s steps(4, end);
}
```

### 2. Transforms

**2D Transforms**
```css
.element {
    /* Translate (move) */
    transform: translate(50px, 100px);
    transform: translateX(50px);
    transform: translateY(100px);

    /* Scale */
    transform: scale(1.5);          /* 150% of original */
    transform: scale(2, 0.5);       /* width 2x, height 0.5x */
    transform: scaleX(2);
    transform: scaleY(0.5);

    /* Rotate */
    transform: rotate(45deg);
    transform: rotate(-90deg);

    /* Skew */
    transform: skew(20deg, 10deg);
    transform: skewX(20deg);
    transform: skewY(10deg);

    /* Combine multiple transforms */
    transform: translate(50px, 50px) rotate(45deg) scale(1.2);
}
```

**3D Transforms**
```css
.element {
    /* Enable 3D */
    transform-style: preserve-3d;
    perspective: 1000px;

    /* 3D transforms */
    transform: rotateX(45deg);
    transform: rotateY(45deg);
    transform: rotateZ(45deg);
    transform: rotate3d(1, 1, 1, 45deg);

    transform: translateZ(50px);
    transform: translate3d(50px, 100px, 50px);

    transform: scaleZ(2);
    transform: scale3d(1, 1, 2);
}

/* Perspective on parent */
.container {
    perspective: 1000px;
}

.card {
    transform: rotateY(180deg);
    transform-origin: center center;
}
```

### 3. Keyframe Animations

**Basic Syntax**
```css
@keyframes slide-in {
    from {
        transform: translateX(-100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

.element {
    animation: slide-in 0.5s ease-out;
}
```

**Multiple Keyframes**
```css
@keyframes bounce {
    0% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(-50px);
    }
    100% {
        transform: translateY(0);
    }
}

.ball {
    animation: bounce 1s ease-in-out infinite;
}
```

**Animation Properties**
```css
.element {
    /* Shorthand: name duration timing-function delay iteration-count direction fill-mode */
    animation: slide-in 1s ease-out 0.5s 2 alternate forwards;

    /* Individual properties */
    animation-name: slide-in;
    animation-duration: 1s;
    animation-timing-function: ease-out;
    animation-delay: 0.5s;
    animation-iteration-count: 2; /* or infinite */
    animation-direction: alternate; /* normal, reverse, alternate, alternate-reverse */
    animation-fill-mode: forwards; /* none, forwards, backwards, both */
    animation-play-state: running; /* running, paused */
}
```

### 4. Common Animation Patterns

**Fade In**
```css
@keyframes fade-in {
    from {
        opacity: 0;
    }
    to {
        opacity: 1;
    }
}

.fade {
    animation: fade-in 0.5s ease-in;
}
```

**Slide In**
```css
@keyframes slide-in-left {
    from {
        transform: translateX(-100%);
    }
    to {
        transform: translateX(0);
    }
}
```

**Bounce**
```css
@keyframes bounce {
    0%, 20%, 50%, 80%, 100% {
        transform: translateY(0);
    }
    40% {
        transform: translateY(-30px);
    }
    60% {
        transform: translateY(-15px);
    }
}
```

**Pulse**
```css
@keyframes pulse {
    0% {
        transform: scale(1);
    }
    50% {
        transform: scale(1.1);
    }
    100% {
        transform: scale(1);
    }
}
```

**Rotate**
```css
@keyframes rotate {
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
}

.spinner {
    animation: rotate 1s linear infinite;
}
```

**Shake**
```css
@keyframes shake {
    0%, 100% {
        transform: translateX(0);
    }
    10%, 30%, 50%, 70%, 90% {
        transform: translateX(-10px);
    }
    20%, 40%, 60%, 80% {
        transform: translateX(10px);
    }
}
```

## 💡 Practical Examples

### Example 1: Animated Button

```css
.button {
    background: #3498db;
    color: white;
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s ease;
    position: relative;
    overflow: hidden;
}

.button:hover {
    background: #2980b9;
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.button:active {
    transform: translateY(0);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

/* Ripple effect */
.button::after {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    width: 0;
    height: 0;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.5);
    transform: translate(-50%, -50%);
    transition: width 0.6s, height 0.6s;
}

.button:active::after {
    width: 300px;
    height: 300px;
}
```

### Example 2: Loading Spinner

```css
.spinner {
    width: 50px;
    height: 50px;
    border: 4px solid #f3f3f3;
    border-top: 4px solid #3498db;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}

/* Dots spinner */
.dots-spinner {
    display: flex;
    gap: 10px;
}

.dot {
    width: 15px;
    height: 15px;
    background: #3498db;
    border-radius: 50%;
    animation: bounce-dots 1.4s infinite ease-in-out;
}

.dot:nth-child(1) {
    animation-delay: -0.32s;
}

.dot:nth-child(2) {
    animation-delay: -0.16s;
}

@keyframes bounce-dots {
    0%, 80%, 100% {
        transform: scale(0);
    }
    40% {
        transform: scale(1);
    }
}
```

### Example 3: Card Flip

```css
.flip-card {
    width: 300px;
    height: 200px;
    perspective: 1000px;
}

.flip-card-inner {
    position: relative;
    width: 100%;
    height: 100%;
    transition: transform 0.6s;
    transform-style: preserve-3d;
}

.flip-card:hover .flip-card-inner {
    transform: rotateY(180deg);
}

.flip-card-front,
.flip-card-back {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    border-radius: 8px;
}

.flip-card-front {
    background: #3498db;
}

.flip-card-back {
    background: #2ecc71;
    transform: rotateY(180deg);
}
```

### Example 4: Slide-in Menu

```css
.menu {
    position: fixed;
    top: 0;
    left: -300px;
    width: 300px;
    height: 100vh;
    background: #2c3e50;
    transition: left 0.3s ease;
}

.menu.open {
    left: 0;
}

/* Slide-in items */
.menu-item {
    opacity: 0;
    transform: translateX(-20px);
}

.menu.open .menu-item {
    animation: slide-in-item 0.3s ease forwards;
}

.menu.open .menu-item:nth-child(1) {
    animation-delay: 0.1s;
}

.menu.open .menu-item:nth-child(2) {
    animation-delay: 0.2s;
}

.menu.open .menu-item:nth-child(3) {
    animation-delay: 0.3s;
}

@keyframes slide-in-item {
    to {
        opacity: 1;
        transform: translateX(0);
    }
}
```

### Example 5: Progress Bar

```css
.progress-bar {
    width: 100%;
    height: 30px;
    background: #f0f0f0;
    border-radius: 15px;
    overflow: hidden;
}

.progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #3498db, #2ecc71);
    border-radius: 15px;
    animation: fill-bar 2s ease-in-out forwards;
}

@keyframes fill-bar {
    from {
        width: 0%;
    }
    to {
        width: 75%;
    }
}
```

## 🎯 Common Interview Questions

### Q1: What's the difference between transition and animation?

**Answer:**
- **Transition**: Simple A-to-B animations, requires trigger (hover, focus)
- **Animation**: Complex multi-step animations, can auto-play

```css
/* Transition */
.button {
    transition: background 0.3s;
}
.button:hover {
    background: blue;
}

/* Animation */
@keyframes pulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.1); }
}
.button {
    animation: pulse 1s infinite;
}
```

### Q2: What is `will-change` and when to use it?

**Answer:**
`will-change` hints the browser to optimize for changes.

```css
.element {
    will-change: transform, opacity;
}

/* Use sparingly! */
/* Apply before animation, remove after */
```

### Q3: How do you optimize animations for performance?

**Answer:**
1. Animate `transform` and `opacity` (GPU-accelerated)
2. Avoid animating `width`, `height`, `top`, `left` (causes reflow)
3. Use `will-change` sparingly
4. Reduce motion for accessibility

```css
/* Good (GPU-accelerated) */
.element {
    transform: translateX(100px);
    opacity: 0.5;
}

/* Bad (causes reflow) */
.element {
    left: 100px;
    width: 500px;
}
```

## 🚨 Common Pitfalls

1. **Animating expensive properties** (width, height, top, left)
2. **Too many simultaneous animations**
3. **Forgetting `transform-origin`**
4. **Not respecting prefers-reduced-motion**
5. **Overusing animations** (less is more)

## 🎓 Best Practices

1. **Use `transform` and `opacity`** for smooth animations
2. **Keep animations under 300ms** for UI interactions
3. **Test on low-end devices**
4. **Respect `prefers-reduced-motion`**
5. **Use `ease-out` for entrances**, `ease-in` for exits
6. **Provide meaningful animations**, not just decoration
7. **Use CSS instead of JavaScript** when possible

## 🔗 Related Topics

- [CSS Fundamentals](./02-css-fundamentals.md)
- [Advanced CSS](./08-advanced-css.md)

---

[← Back to HTML & CSS](./README.md)
