# Next.js

Master Next.js, the production-ready React framework for building modern web applications. Next.js is increasingly required for senior frontend roles and offers powerful features like SSR, SSG, and ISR.

## 📚 Topics Covered

### Fundamentals
1. **[Pages & Routing](./01-pages-routing.md)**
   - File-based routing
   - Dynamic routes
   - Catch-all routes
   - Link component

2. **[App Router](./02-app-router.md)**
   - App directory structure
   - Server components
   - Client components
   - Layouts and templates

### Rendering Strategies
3. **[Rendering Strategies](./03-rendering-strategies.md)**
   - SSR (Server-Side Rendering)
   - SSG (Static Site Generation)
   - ISR (Incremental Static Regeneration)
   - CSR (Client-Side Rendering)
   - When to use each

### Data Fetching
4. **[Data Fetching](./04-data-fetching.md)**
   - getServerSideProps
   - getStaticProps
   - getStaticPaths
   - App Router fetch patterns

5. **[API Routes](./05-api-routes.md)**
   - Creating API endpoints
   - Request handling
   - Middleware
   - Error handling

### Optimization
6. **[Image Optimization](./06-image-optimization.md)**
   - next/image component
   - Automatic optimization
   - Responsive images
   - Lazy loading

7. **[SEO Optimization](./07-seo-optimization.md)**
   - Meta tags
   - Open Graph
   - Sitemaps
   - robots.txt

### Advanced Topics
8. **[Middleware](./08-middleware.md)**
   - Route protection
   - Redirects and rewrites
   - Authentication
   - Headers modification

9. **[Deployment](./09-deployment.md)**
   - Vercel deployment
   - Environment variables
   - Build optimization
   - Performance monitoring

10. **[Best Practices](./10-best-practices.md)**
    - Project structure
    - Performance tips
    - Security practices
    - Testing strategies

---

## 🎯 Interview Focus Areas

### Most Important
1. Rendering strategies (SSR, SSG, ISR)
2. App Router vs Pages Router
3. Data fetching methods
4. File-based routing
5. Server vs Client components

### Very Important
6. Image optimization
7. API routes
8. Middleware
9. SEO optimization
10. Performance optimization

### Good to Know
11. Deployment on Vercel
12. Environment configuration
13. Internationalization
14. Edge functions

---

## 💡 Quick Reference

### Common Interview Questions
1. "Difference between SSR, SSG, and ISR?"
2. "When to use getServerSideProps vs getStaticProps?"
3. "Explain App Router vs Pages Router"
4. "How does Next.js optimize images?"
5. "What are server components?"
6. "How to implement authentication in Next.js?"
7. "Explain Next.js middleware"
8. "How to optimize Next.js for SEO?"

### Essential Patterns
```javascript
// Server-Side Rendering
export async function getServerSideProps(context) {
    const res = await fetch(`https://api.example.com/data`);
    const data = await res.json();

    return {
        props: { data }
    };
}

// Static Site Generation
export async function getStaticProps() {
    const res = await fetch('https://api.example.com/posts');
    const posts = await res.json();

    return {
        props: { posts },
        revalidate: 60 // ISR: regenerate every 60 seconds
    };
}

// App Router - Server Component
async function ServerComponent() {
    const data = await fetch('https://api.example.com/data');
    return <div>{/* render data */}</div>;
}

// Middleware
export function middleware(request) {
    if (!request.cookies.get('token')) {
        return NextResponse.redirect(new URL('/login', request.url));
    }
}
```

---

## 🔗 External Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js Learn Course](https://nextjs.org/learn)
- [Vercel Documentation](https://vercel.com/docs)
- [Next.js GitHub](https://github.com/vercel/next.js)
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)

---

[← Back to Frontend](../README.md)
