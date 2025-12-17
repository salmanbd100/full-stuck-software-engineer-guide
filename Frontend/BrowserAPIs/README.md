# Browser APIs - Interview Preparation Guide

## Overview

This directory covers essential **Browser APIs** that are fundamental for modern web development and frequently tested in frontend engineering interviews. These APIs enable client-side data persistence, state management, and secure user authentication flows.

### What You'll Learn

- **Storage APIs**: localStorage, sessionStorage, and their appropriate use cases
- **Cookie Management**: Cookie attributes, SameSite policy, and security considerations
- **Security Best Practices**: XSS prevention, CSRF protection, and data privacy
- **Real-World Patterns**: Production-ready implementations for common scenarios
- **Browser Compatibility**: Cross-browser support and polyfills
- **Performance Optimization**: Storage quota management and efficient data handling

### Target Audience

This guide is designed for:
- Frontend engineers with 2-7 years of experience
- Candidates preparing for FAANG/multinational company interviews
- Developers working with client-side state management
- Engineers implementing authentication and session management

## Why Browser APIs Matter

Understanding browser storage and APIs is crucial because:

1. **State Persistence**: Enable offline-first applications and smooth user experiences
2. **Authentication**: Secure token storage and session management
3. **Performance**: Reduce server requests through intelligent caching
4. **Privacy Compliance**: GDPR, CCPA, and modern privacy requirements
5. **Security**: Prevent XSS, CSRF, and session hijacking attacks

## =Ú Topics Covered

### 1. Storage APIs (`01-storage-apis.md`)
**Time**: 4-5 hours | **Difficulty**: Intermediate

Learn about browser storage mechanisms:
- localStorage and sessionStorage comparison
- API methods and usage patterns
- Storage events and cross-tab communication
- Quota management and size limits
- Security implications and best practices
- Serialization strategies
- Real-world use cases

### 2. Cookies and SameSite Policy (`02-cookies-same-site.md`)
**Time**: 5-6 hours | **Difficulty**: Intermediate to Advanced

Master cookie management and security:
- Cookie attributes (Secure, HttpOnly, SameSite, Domain, Path)
- SameSite values: Strict, Lax, None
- Third-party cookies and privacy
- GDPR/CCPA compliance
- CSRF protection strategies
- Implementation patterns (server + client)
- Cookie vulnerabilities and mitigation

---

## =Ö Study Plan

### Beginner Track (8-10 hours)
For developers new to browser storage APIs:

**Week 1: Storage Fundamentals**
- Day 1-2: localStorage and sessionStorage basics (2-3 hours)
  - Read sections 1-3 of `01-storage-apis.md`
  - Implement basic storage operations
  - Practice serialization/deserialization
- Day 3-4: Storage patterns and best practices (2-3 hours)
  - Read sections 4-6 of `01-storage-apis.md`
  - Build a simple shopping cart with localStorage
  - Implement storage quota detection
- Day 5-6: Cookie fundamentals (2-3 hours)
  - Read sections 1-3 of `02-cookies-same-site.md`
  - Understand cookie attributes
  - Set and read cookies in JavaScript
- Day 7: Review and practice (1-2 hours)
  - Answer interview questions from both files
  - Build a simple authentication flow

### Intermediate Track (10-12 hours)
For developers with basic storage knowledge:

**Week 1: Deep Dive into Storage**
- Day 1: localStorage vs sessionStorage patterns (2 hours)
  - Study all patterns in `01-storage-apis.md`
  - Implement cross-tab communication
- Day 2: Storage security (2 hours)
  - XSS vulnerabilities and prevention
  - Encryption strategies
- Day 3-4: Cookie security (3-4 hours)
  - SameSite policy deep dive
  - CSRF protection implementation
  - Study `02-cookies-same-site.md` sections 4-7
- Day 5: Real-world implementations (2-3 hours)
  - Build authentication with cookies
  - Implement cookie consent UI
- Day 6-7: Interview preparation (2-3 hours)
  - Practice all interview questions
  - Review security best practices

### Advanced Track (12-15 hours)
For experienced developers targeting senior roles:

**Week 1: Mastery and Real-World Scenarios**
- Day 1-2: Advanced storage patterns (3-4 hours)
  - IndexedDB comparison
  - Storage quota APIs
  - Service Worker integration
- Day 3-4: Security architecture (4-5 hours)
  - Design secure authentication flows
  - Implement token rotation
  - CSRF protection strategies
  - Study all security sections thoroughly
- Day 5-6: Privacy compliance (3-4 hours)
  - GDPR/CCPA implementation
  - Cookie consent management platforms
  - Third-party cookie alternatives
- Day 7: System design integration (2-3 hours)
  - Design a complete authentication system
  - Optimize for performance and security
  - Practice explaining trade-offs

---

## < Browser Compatibility

### localStorage and sessionStorage

| Browser | Version | Support |
|---------|---------|---------|
| Chrome | 4+ | Full |
| Firefox | 3.5+ | Full |
| Safari | 4+ | Full |
| Edge | All | Full |
| IE | 8+ | Full |

**Notes**:
- All modern browsers support 5-10MB storage
- Private/Incognito mode may have restrictions
- Some browsers clear storage on session end

### Cookies

| Feature | Chrome | Firefox | Safari | Edge |
|---------|--------|---------|--------|------|
| Basic cookies | All | All | All | All |
| SameSite | 51+ | 60+ | 12+ | 16+ |
| SameSite=None requires Secure | 80+ | 69+ | 13+ | 80+ |

**Important**: Always check `navigator.cookieEnabled` before using cookies.

### Storage Event

| Browser | Support | Notes |
|---------|---------|-------|
| Chrome | 5+ | Full support |
| Firefox | 45+ | Full support |
| Safari | 4+ | Full support |
| Edge | All | Full support |

**Limitation**: Storage events only fire in OTHER tabs, not the tab that made the change.

---

## = Security Considerations

### Critical Security Rules

1. **Never Store Sensitive Data in localStorage/sessionStorage**
   - Accessible via JavaScript (XSS vulnerability)
   - Not encrypted
   - Visible in DevTools
   - Use HttpOnly cookies for sensitive tokens

2. **Always Use Secure Cookies for Authentication**
   ```javascript
   // Good: Secure, HttpOnly, SameSite
   Set-Cookie: token=abc123; Secure; HttpOnly; SameSite=Strict

   // Bad: Accessible via JavaScript
   localStorage.setItem('token', 'abc123');
   ```

3. **Implement Proper SameSite Policy**
   - Use `SameSite=Strict` for same-site requests
   - Use `SameSite=Lax` for reasonable cross-site navigation
   - Only use `SameSite=None` when absolutely necessary (with Secure)

4. **Validate and Sanitize All Stored Data**
   - Never trust data from storage
   - Validate on retrieval
   - Sanitize before rendering

5. **Implement Content Security Policy (CSP)**
   ```html
   <meta http-equiv="Content-Security-Policy"
         content="default-src 'self'; script-src 'self'">
   ```

### Common Vulnerabilities

| Vulnerability | Impact | Mitigation |
|---------------|--------|------------|
| XSS in localStorage | Token theft | Use HttpOnly cookies, sanitize input |
| CSRF without SameSite | Unauthorized actions | SameSite=Strict/Lax, CSRF tokens |
| Third-party cookie tracking | Privacy violation | Block third-party cookies, use alternatives |
| Session fixation | Account hijacking | Regenerate session ID on login |
| Cookie injection | Session hijacking | Validate origin, use Secure flag |

---

## =€ Quick Reference

### localStorage/sessionStorage API

```javascript
// Set item
localStorage.setItem('key', 'value');
sessionStorage.setItem('key', JSON.stringify({ data: 'value' }));

// Get item
const value = localStorage.getItem('key');
const obj = JSON.parse(sessionStorage.getItem('key'));

// Remove item
localStorage.removeItem('key');

// Clear all
localStorage.clear();

// Get number of items
const count = localStorage.length;

// Get key by index
const key = localStorage.key(0);

// Check if key exists
if (localStorage.getItem('key') !== null) {
  // Key exists
}
```

### Cookie API

```javascript
// Set cookie (client-side)
document.cookie = "name=value; path=/; max-age=3600; SameSite=Strict; Secure";

// Get all cookies
const cookies = document.cookie;

// Parse cookies
function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
}

// Delete cookie
document.cookie = "name=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
```

### Storage Event Listener

```javascript
// Listen for storage changes (cross-tab communication)
window.addEventListener('storage', (e) => {
  console.log('Key:', e.key);
  console.log('Old value:', e.oldValue);
  console.log('New value:', e.newValue);
  console.log('URL:', e.url);
  console.log('Storage area:', e.storageArea);
});
```

### Quota Detection

```javascript
// Check available storage
if ('storage' in navigator && 'estimate' in navigator.storage) {
  navigator.storage.estimate().then(({ usage, quota }) => {
    const percentUsed = (usage / quota) * 100;
    console.log(`Using ${usage} bytes out of ${quota} (${percentUsed.toFixed(2)}%)`);
  });
}
```

---

## <¯ Common Interview Questions

### Storage APIs
1. **What's the difference between localStorage and sessionStorage?**
   - Lifetime, scope, use cases

2. **How do storage events work for cross-tab communication?**
   - Event propagation, limitations, use cases

3. **What are the security risks of localStorage?**
   - XSS vulnerabilities, token theft, mitigation strategies

4. **How do you handle storage quota exceeded errors?**
   - Try-catch, quota detection, cleanup strategies

5. **Can you encrypt data in localStorage?**
   - Client-side encryption, Web Crypto API, limitations

### Cookies and SameSite
1. **Explain the three SameSite values (Strict, Lax, None).**
   - Security implications, use cases, browser behavior

2. **Why is SameSite important for CSRF protection?**
   - Attack mechanism, prevention, limitations

3. **What's the difference between HttpOnly and Secure flags?**
   - JavaScript access, HTTPS requirement, security implications

4. **How do you implement GDPR-compliant cookie consent?**
   - Legal requirements, UX patterns, implementation

5. **What are third-party cookies and why are they being phased out?**
   - Privacy concerns, browser policies, alternatives

### Practical Scenarios
1. **Design an authentication system using cookies.**
   - Token storage, refresh tokens, security considerations

2. **Implement a shopping cart that persists across sessions.**
   - localStorage vs cookies, serialization, sync strategies

3. **How would you implement cross-tab synchronization?**
   - Storage events, BroadcastChannel API, WebSocket fallback

4. **Design a cookie consent manager for a global website.**
   - Regional requirements, user preferences, implementation

5. **How do you migrate from localStorage to cookies for auth tokens?**
   - Migration strategy, backward compatibility, testing

---

##  Best Practices Summary

### Do's
- Use localStorage for non-sensitive, persistent data
- Use sessionStorage for temporary, tab-specific data
- Use HttpOnly, Secure cookies for authentication tokens
- Implement SameSite=Strict or Lax for cookies
- Always try-catch localStorage operations
- Validate and sanitize data from storage
- Check for storage availability before use
- Implement proper error handling
- Use JSON serialization for complex data
- Monitor storage quota usage

### Don'ts
- Never store passwords or sensitive data in localStorage
- Don't store auth tokens in localStorage (use HttpOnly cookies)
- Don't trust data from storage without validation
- Don't exceed storage quotas without handling errors
- Don't use cookies without SameSite attribute
- Don't set cookies without Secure flag in production
- Don't ignore GDPR/CCPA compliance requirements
- Don't use synchronous APIs in critical paths
- Don't store large binary data in localStorage
- Don't forget to clean up expired data

---

## = External Resources

### Official Documentation
- [MDN - Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [MDN - Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [MDN - SameSite cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [WHATWG - Storage Living Standard](https://storage.spec.whatwg.org/)

### Security Resources
- [OWASP - Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP - Cross-Site Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [web.dev - SameSite cookies explained](https://web.dev/samesite-cookies-explained/)

### Privacy and Compliance
- [GDPR Official Text](https://gdpr-info.eu/)
- [CCPA Official Information](https://oag.ca.gov/privacy/ccpa)
- [Cookie Consent Tools Comparison](https://www.cookiebot.com/)

### Browser Compatibility
- [Can I Use - localStorage](https://caniuse.com/namevalue-storage)
- [Can I Use - SameSite cookies](https://caniuse.com/same-site-cookie-attribute)

### Articles and Guides
- [The Complete Guide to LocalStorage](https://blog.logrocket.com/localstorage-javascript-complete-guide/)
- [Understanding SameSite Cookies](https://web.dev/samesite-cookies-explained/)
- [Authentication Best Practices](https://blog.bytebytego.com/p/ep91-authentication-session-cookie)

---

## < Portfolio Reference

For more frontend engineering resources and interview preparation materials, visit [salmanrahman.com](https://salmanrahman.com).

---

## Navigation

### Within This Directory
- **[01-storage-apis.md](./01-storage-apis.md)** - localStorage and sessionStorage deep dive
- **[02-cookies-same-site.md](./02-cookies-same-site.md)** - Cookie management and SameSite policy

### Related Topics
- **[Frontend/JavaScript/](../JavaScript/)** - Core JavaScript concepts
- **[Frontend/React/](../React/)** - React state management integration
- **[Frontend/NextJS/](../NextJS/)** - Next.js authentication patterns
- **[Frontend/WebPerformance/](../WebPerformance/)** - Performance optimization with caching

### Other Domains
- **[Backend/](../../Backend/)** - Server-side authentication and session management
- **[SystemDesign/](../../SystemDesign/)** - Authentication system design
- **[DevOps/](../../DevOps/)** - Security and compliance in production

---

**Last Updated**: December 2025
**Maintainer**: Salman Rahman
**Repository**: [interview-preparation](https://github.com/salman-rahman/interview-preparation)
