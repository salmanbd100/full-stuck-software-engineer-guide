# HTML & CSS

Master HTML and CSS fundamentals essential for frontend interviews. Strong HTML/CSS skills demonstrate your understanding of web fundamentals and ability to build accessible, responsive user interfaces.

## 📚 Topics Covered

### HTML Fundamentals
1. **[Semantic HTML](./01-semantic-html.md)**
   - HTML5 semantic elements
   - Document structure
   - SEO and accessibility benefits

2. **[Accessibility](./07-accessibility.md)**
   - ARIA roles and attributes
   - Keyboard navigation
   - Screen reader compatibility

### CSS Fundamentals
2. **[CSS Fundamentals](./02-css-fundamentals.md)**
   - Selectors and specificity
   - Box model
   - Cascade and inheritance

### Layout Systems
3. **[Flexbox](./03-flexbox.md)**
   - Flex container and items
   - Common layout patterns
   - Alignment and distribution

4. **[Grid](./04-grid.md)**
   - Grid container and items
   - Grid template areas
   - Responsive grid layouts

5. **[Responsive Design](./05-responsive-design.md)**
   - Media queries
   - Mobile-first approach
   - Viewport and units

### Advanced Topics
6. **[CSS Animations](./06-css-animations.md)**
   - Transitions
   - Keyframe animations
   - Transform and performance

8. **[Advanced CSS](./08-advanced-css.md)**
   - CSS variables (custom properties)
   - Pseudo-elements and pseudo-classes
   - CSS preprocessors (Sass, Less)

---

## 🎯 Interview Focus Areas

### Most Commonly Asked
1. Box model (content, padding, border, margin)
2. Flexbox vs Grid
3. Responsive design and media queries
4. CSS specificity and cascade
5. Semantic HTML and accessibility

### Frequently Asked
6. CSS positioning (static, relative, absolute, fixed, sticky)
7. CSS animations and transitions
8. CSS units (px, em, rem, %, vw/vh)
9. CSS selectors and combinators
10. Cross-browser compatibility

---

## 💡 Quick Reference

### Common Interview Questions
1. "Explain the CSS box model"
2. "When to use Flexbox vs Grid?"
3. "How does CSS specificity work?"
4. "What is semantic HTML?"
5. "How do you make a website responsive?"
6. "Difference between em and rem?"
7. "How do CSS animations work?"
8. "What are ARIA attributes?"

### Essential Patterns
```css
/* Centering with Flexbox */
.container {
    display: flex;
    justify-content: center;
    align-items: center;
}

/* Responsive Grid */
.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1rem;
}

/* Mobile-first Media Query */
.element {
    width: 100%;
}

@media (min-width: 768px) {
    .element {
        width: 50%;
    }
}
```

---

## 🔗 External Resources

- [MDN HTML Reference](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [MDN CSS Reference](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS Tricks](https://css-tricks.com/)
- [Flexbox Froggy](https://flexboxfroggy.com/) - Learn Flexbox
- [Grid Garden](https://cssgridgarden.com/) - Learn Grid
- [Can I Use](https://caniuse.com/) - Browser compatibility

---

[← Back to Frontend](../README.md)
