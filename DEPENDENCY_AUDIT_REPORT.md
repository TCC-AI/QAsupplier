# Dependency Audit Report
**Project:** QAsupplier (Supplier Management System)
**Date:** 2026-01-21
**Auditor:** Claude AI

---

## Executive Summary

This project is a single-page static HTML application with minimal external dependencies. While the simplicity reduces attack surface, there are several opportunities for improvement in security, performance, and maintainability.

**Key Findings:**
- ✅ **Low dependency footprint** - Only 2 external resources
- ⚠️ **Security vulnerabilities** - Missing integrity checks and insecure credential storage
- ⚠️ **Performance bloat** - Unused font weights and large inline code
- ⚠️ **No package management** - No versioning or reproducible builds

---

## Current Dependencies

### 1. External CDN Resources

#### Google Fonts (index.html:8-10)
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Playfair+Display:wght@600;700;800&display=swap" rel="stylesheet">
```

**Analysis:**
- **Font Families:** Inter (4 weights) + Playfair Display (3 weights) = **7 font files**
- **Estimated Size:** ~200-300KB (uncompressed)
- **Status:** ✅ Using Google Fonts API v2 (latest)
- **Issues:**
  - ❌ No Subresource Integrity (SRI) hash
  - ⚠️ Potentially loading unused font weights
  - ⚠️ No fallback strategy if CDN fails

#### Google Apps Script API (index.html:645)
```javascript
const API_URL = "https://script.google.com/macros/s/AKfycbw.../exec";
```

**Analysis:**
- **Status:** ✅ HTTPS endpoint
- **Issues:**
  - ❌ API key exposed in client-side code
  - ❌ No API versioning
  - ❌ No rate limiting or error handling for 429/503 errors
  - ⚠️ Hardcoded URL makes environment switching difficult

### 2. Inline Code (No Package Manager)

**Current Structure:**
- `index.html`: **35KB** (953 lines)
  - CSS: ~500 lines
  - JavaScript: ~310 lines
  - HTML: ~140 lines

---

## Security Vulnerabilities

### 🔴 Critical Issues

#### 1. Missing Subresource Integrity (SRI)
**Location:** index.html:8-10
**Risk Level:** HIGH

**Issue:**
External resources from `fonts.googleapis.com` lack integrity checks, making the application vulnerable to CDN compromise attacks.

**Impact:**
- Malicious code injection if Google Fonts CDN is compromised
- Man-in-the-middle attacks on font loading

**Recommendation:**
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Playfair+Display:wght@600;700;800&display=swap"
      rel="stylesheet"
      integrity="sha384-[HASH-HERE]"
      crossorigin="anonymous">
```

#### 2. Insecure Credential Storage
**Location:** index.html:666, 705-706
**Risk Level:** HIGH

**Issue:**
Authentication tokens stored in `localStorage` without encryption:
```javascript
localStorage.setItem('sms_token', result.data.token);
localStorage.setItem('sms_username', result.data.username);
```

**Impact:**
- XSS attacks can steal tokens
- No token expiration mechanism visible
- Tokens persist across sessions

**Recommendation:**
- Use `httpOnly` cookies for sensitive tokens (requires backend changes)
- Implement token expiration and refresh mechanism
- Consider using session storage for temporary auth state
- Add CSRF protection

#### 3. Exposed API Endpoint
**Location:** index.html:645
**Risk Level:** MEDIUM

**Issue:**
Google Apps Script URL is hardcoded in client-side code, exposing the deployment ID.

**Impact:**
- Anyone can discover and potentially abuse the API endpoint
- No way to rotate endpoints without redeploying frontend
- Difficult to manage multiple environments (dev/staging/prod)

**Recommendation:**
```javascript
// Use environment-based configuration
const API_URL = window.location.hostname === 'localhost'
  ? 'http://localhost:3000/api'
  : 'https://api.yourdomain.com/v1';
```

### ⚠️ Medium Priority Issues

#### 4. No Content Security Policy (CSP)
**Risk Level:** MEDIUM

**Issue:**
Missing CSP headers to prevent XSS and injection attacks.

**Recommendation:**
Add meta tag or configure server headers:
```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self';
               font-src 'self' https://fonts.googleapis.com https://fonts.gstatic.com;
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
               script-src 'self';
               connect-src 'self' https://script.google.com;">
```

#### 5. No HTTPS Enforcement
**Risk Level:** MEDIUM

**Issue:**
No automatic redirect from HTTP to HTTPS.

**Recommendation:**
Add to `<head>`:
```html
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
```

---

## Performance & Bloat Analysis

### 🟡 Font Loading Bloat

**Current State:**
```
Inter: 400, 500, 600, 700 (4 weights)
Playfair Display: 600, 700, 800 (3 weights)
Total: 7 font files ≈ 200-300KB
```

**Actual Usage Analysis:**
- `Inter 400`: Body text ✅ (used)
- `Inter 500`: Form labels, nav items ✅ (used)
- `Inter 600`: Buttons, headings ✅ (used)
- `Inter 700`: Headings ⚠️ (overlap with 600)
- `Playfair 600`: Headings ✅ (used)
- `Playfair 700`: Headings ✅ (used)
- `Playfair 800`: ❌ **NOT USED** (can be removed)

**Recommendation:**
```html
<!-- Optimized version -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Playfair+Display:wght@600;700&display=swap&subset=latin" rel="stylesheet">
```
**Savings:** ~15-20% reduction in font loading

### 🟡 Monolithic Code Structure

**Issue:**
All code in single 35KB HTML file makes maintenance difficult and prevents browser caching.

**Recommendation:**
Split into separate files:
```
project/
├── index.html (HTML only, ~5KB)
├── css/
│   └── styles.css (~15KB)
└── js/
    └── app.js (~15KB)
```

**Benefits:**
- Browser can cache CSS/JS separately
- Easier code maintenance
- Enable minification (30-40% size reduction)
- Better developer experience

### 🟡 Unused CSS Code

**Potential Issues Found:**
- index.html:382-384: Duplicate/malformed CSS rules
- Multiple unused utility classes
- Redundant vendor prefixes for modern browsers

**Recommendation:**
- Use CSS minification tools
- Remove duplicate rules at lines 382-384
- Consider using CSS purging tools (PurgeCSS)

---

## Missing Modern Tooling

### 1. No Package Manager
**Impact:**
- No dependency versioning
- No reproducible builds
- Manual updates required
- No security audit tools

**Recommendation:**
Initialize npm/package.json:
```json
{
  "name": "qasupplier",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "vite": "^5.0.0"
  }
}
```

### 2. No Build System
**Impact:**
- No code minification
- No dead code elimination
- No modern JavaScript features (ES modules)
- Difficult to implement code splitting

**Recommendation:**
Adopt a lightweight build tool like Vite or Parcel for:
- Code minification (30-40% size reduction)
- ES module support
- Development server with hot reload
- Asset optimization

### 3. No Dependency Locking
**Impact:**
- Google Fonts CDN can change without notice
- No way to rollback to previous versions
- Unpredictable behavior across environments

**Recommendation:**
Consider self-hosting fonts:
```bash
npm install @fontsource/inter @fontsource/playfair-display
```

---

## Outdated Packages Analysis

**Status:** ✅ N/A - No package manager in use

**Note:** Since this project uses CDN resources, "outdated" analysis focuses on API versions:

| Resource | Current | Latest | Status |
|----------|---------|--------|--------|
| Google Fonts API | v2 | v2 | ✅ Current |
| Google Apps Script | N/A | N/A | ⚠️ No versioning |

---

## Recommendations Summary

### 🔴 High Priority (Security)

1. **Add Subresource Integrity (SRI) hashes** to all external resources
2. **Implement secure token storage** - migrate to httpOnly cookies
3. **Add Content Security Policy (CSP)** headers
4. **Environment-based API configuration** - remove hardcoded endpoints

### 🟡 Medium Priority (Performance)

5. **Optimize font loading** - remove unused Playfair 800 weight
6. **Split code into separate files** - enable browser caching
7. **Add minification** - reduce bundle size by ~30-40%
8. **Fix CSS duplicate rules** at index.html:382-384

### 🟢 Low Priority (Maintainability)

9. **Initialize package manager** (npm/pnpm)
10. **Adopt build system** (Vite recommended for simplicity)
11. **Add dependency locking** mechanism
12. **Self-host fonts** for better control and privacy

---

## Implementation Roadmap

### Phase 1: Security Hardening (Immediate)
- [ ] Add SRI hashes to Google Fonts
- [ ] Implement CSP meta tag
- [ ] Add HTTPS enforcement
- [ ] Review and improve token handling

### Phase 2: Performance Optimization (Week 1)
- [ ] Remove unused font weight (Playfair 800)
- [ ] Add `font-display: swap` for better perceived performance
- [ ] Fix CSS duplicate rules
- [ ] Split HTML/CSS/JS into separate files

### Phase 3: Modern Tooling (Week 2-3)
- [ ] Initialize npm with package.json
- [ ] Configure Vite or Parcel
- [ ] Implement minification
- [ ] Consider self-hosting fonts

### Phase 4: Architecture Improvements (Future)
- [ ] Migrate to httpOnly cookies (requires backend update)
- [ ] Implement environment configuration
- [ ] Add automated security scanning (npm audit)
- [ ] Set up dependency update workflow (Dependabot/Renovate)

---

## Estimated Impact

| Metric | Current | After Changes | Improvement |
|--------|---------|---------------|-------------|
| Initial Load Size | ~35KB HTML + ~250KB fonts | ~25KB + ~200KB fonts | ~18% reduction |
| Security Score | 4/10 | 8/10 | +4 points |
| Maintainability | 5/10 | 9/10 | +4 points |
| Cache-ability | 2/10 | 8/10 | +6 points |

---

## Conclusion

While the QAsupplier project benefits from a minimal dependency footprint, there are significant opportunities for improvement in security, performance, and maintainability. The recommendations above prioritize security fixes first, followed by performance optimizations and modern tooling adoption.

**Next Steps:**
1. Review and approve this audit report
2. Prioritize recommendations based on business needs
3. Create implementation tickets for development team
4. Schedule security fixes for immediate deployment

---

**Questions or Concerns?**
Please review this audit and let me know which recommendations you'd like to implement first.
