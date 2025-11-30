# Web Performance

Master web performance optimization techniques essential for senior frontend roles. Performance is a critical factor in user experience and business metrics, making it a key interview topic at major companies.

## 📚 Topics Covered

### Performance Metrics
1. **[Core Web Vitals](./01-core-web-vitals.md)**
   - LCP (Largest Contentful Paint)
   - FID (First Input Delay) / INP (Interaction to Next Paint)
   - CLS (Cumulative Layout Shift)
   - Measurement and optimization

### Loading Optimization
2. **[Lazy Loading](./02-lazy-loading.md)**
   - Image lazy loading
   - Component lazy loading
   - Route-based code splitting
   - Intersection Observer API

3. **[Code Splitting](./03-code-splitting.md)**
   - Dynamic imports
   - Bundle analysis
   - Webpack/Vite configuration
   - React lazy and Suspense

4. **[Bundle Optimization](./06-bundle-optimization.md)**
   - Tree shaking
   - Minification
   - Compression (gzip, brotli)
   - Analyzing bundle size

### Resource Optimization
5. **[Image Optimization](./05-image-optimization.md)**
   - Modern formats (WebP, AVIF)
   - Responsive images
   - Image compression
   - CDN usage

6. **[Caching Strategies](./04-caching-strategies.md)**
   - Browser caching
   - Service workers
   - CDN caching
   - Cache invalidation

### Monitoring & Analysis
7. **[Performance Monitoring](./07-performance-monitoring.md)**
   - Lighthouse
   - Web Vitals API
   - Performance Observer
   - Real User Monitoring (RUM)

8. **[Rendering Optimization](./08-rendering-optimization.md)**
   - Virtual DOM optimization
   - Debouncing and throttling
   - requestAnimationFrame
   - CSS containment

---

## 🎯 Interview Focus Areas

### Most Important
1. Core Web Vitals (LCP, FID/INP, CLS)
2. Lazy loading techniques
3. Code splitting strategies
4. Bundle optimization
5. Image optimization

### Very Important
6. Caching strategies
7. Performance measurement
8. Rendering optimization
9. Network optimization
10. Critical rendering path

### Good to Know
11. Service workers
12. HTTP/2 and HTTP/3
13. Resource hints (preload, prefetch)
14. Performance budgets

---

## 💡 Quick Reference

### Common Interview Questions
1. "What are Core Web Vitals?"
2. "How do you optimize images for web?"
3. "Explain code splitting and lazy loading"
4. "What caching strategies do you use?"
5. "How to measure web performance?"
6. "What is the critical rendering path?"
7. "Difference between debounce and throttle?"
8. "How to optimize bundle size?"

### Essential Patterns
```javascript
// Lazy Loading Component
const LazyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
    return (
        <Suspense fallback={<Loading />}>
            <LazyComponent />
        </Suspense>
    );
}

// Image Lazy Loading
<img
    src="image.jpg"
    loading="lazy"
    alt="Description"
/>

// Debounce Function
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func(...args), wait);
    };
}

// Performance Measurement
const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        console.log(entry);
    }
});

observer.observe({ entryTypes: ['largest-contentful-paint'] });

// Code Splitting
const loadModule = async () => {
    const module = await import('./heavyModule.js');
    module.doSomething();
};
```

---

## 📊 Performance Targets

### Core Web Vitals Thresholds
- **LCP**: < 2.5s (Good), 2.5-4s (Needs Improvement), > 4s (Poor)
- **FID/INP**: < 100ms (Good), 100-300ms (Needs Improvement), > 300ms (Poor)
- **CLS**: < 0.1 (Good), 0.1-0.25 (Needs Improvement), > 0.25 (Poor)

### General Guidelines
- First Contentful Paint (FCP): < 1.8s
- Time to Interactive (TTI): < 3.8s
- Total Blocking Time (TBT): < 200ms
- Bundle size: < 200KB (initial)

---

## 🔗 External Resources

- [Web.dev Performance](https://web.dev/performance/)
- [MDN Performance](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [Chrome DevTools Performance](https://developer.chrome.com/docs/devtools/performance/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [WebPageTest](https://www.webpagetest.org/)
- [Bundle Analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)

---

[← Back to Frontend](../README.md)
