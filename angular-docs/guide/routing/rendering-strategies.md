# Rendering Strategies
> Source: https://angular.dev/guide/routing/rendering-strategies

## Overview

This guide helps developers select appropriate rendering strategies for different parts of Angular applications. Each approach involves tradeoffs between initial load performance, interactivity, SEO capabilities, and server resource usage.

Angular supports three primary rendering strategies:

- **Client-Side Rendering (CSR)** -- Content renders entirely in the browser
- **Static Site Generation (SSG/Prerendering)** -- Content is pre-rendered at build time
- **Server-Side Rendering (SSR)** -- Content renders on the server for initial route requests

---

## Client-Side Rendering (CSR)

CSR is Angular's default approach where content renders entirely in the browser after JavaScript loads.

### When to Use CSR

**Suitable for:**
- Interactive applications (dashboards, admin panels)
- Real-time applications
- Internal tools where SEO does not matter
- Single-page applications with complex client-side state

**Less suitable for:**
- Public-facing content requiring SEO
- Pages where initial load performance is critical

### Trade-offs

| Aspect | Impact |
|--------|--------|
| **SEO** | Poor -- content invisible to crawlers until JS executes |
| **Initial load** | Slower -- must download and execute JavaScript first |
| **Interactivity** | Immediate once loaded |
| **Server needs** | Minimal beyond configuration |
| **Complexity** | Simplest with minimum configuration |

---

## Static Site Generation (SSG/Prerendering)

SSG pre-renders pages at build time into static HTML files. The server sends pre-built HTML for initial page loads. Following hydration, the app runs entirely client-side like a traditional SPA -- subsequent navigation and API calls happen without server rendering.

### When to Use SSG

**Suitable for:**
- Marketing pages and landing pages
- Blog posts and documentation
- Product catalogs with stable content
- Content that does not change per-user

**Less suitable for:**
- User-specific content
- Frequently changing data
- Real-time information

### Trade-offs

| Aspect | Impact |
|--------|--------|
| **SEO** | Excellent -- full HTML available immediately |
| **Initial load** | Fastest -- pre-generated HTML |
| **Interactivity** | After hydration completes |
| **Server needs** | None for serving (CDN-friendly) |
| **Build time** | Longer -- generates all pages upfront |
| **Content updates** | Requires rebuild and redeploy |

**Implementation:** See "Customizing build-time prerendering" in the SSR guide.

---

## Server-Side Rendering (SSR)

SSR generates HTML on the server for the initial request for a route, providing dynamic content with strong SEO capabilities. The server renders HTML and sends it to the client.

After the client receives the page, Angular hydrates the app, which then runs entirely client-side like a traditional SPA -- subsequent navigation and API calls happen without additional server rendering.

### When to Use SSR

**Suitable for:**
- E-commerce product pages (dynamic pricing/inventory)
- News sites and social media feeds
- Personalized content that changes frequently

**Less suitable for:**
- Static content (use SSG instead)
- When server costs are a concern

### Trade-offs

| Aspect | Impact |
|--------|--------|
| **SEO** | Excellent -- full HTML for crawlers |
| **Initial load** | Fast -- immediate content visibility |
| **Interactivity** | Delayed until hydration |
| **Server needs** | Requires server |
| **Personalization** | Full access to user context |
| **Server costs** | Higher -- renders on initial route requests |

**Implementation:** See "Server routing" and "Authoring server-compatible components" in the SSR guide.

---

## Choosing the Right Strategy

### Decision Matrix

| If you need... | Use this strategy | Why |
|---|---|---|
| **SEO + Static content** | SSG | Pre-rendered HTML, fastest load |
| **SEO + Dynamic content** | SSR | Fresh content on initial route requests |
| **No SEO + Interactivity** | CSR | Simplest, no server needed |
| **Mixed requirements** | Hybrid | Different strategies per route |

---

## Making SSR/SSG Interactive with Hydration

When using SSR or SSG, Angular "hydrates" the server-rendered HTML to make it interactive.

**Available strategies:**

- **Full hydration** -- Entire app becomes interactive at once (default)
- **Incremental hydration** -- Parts become interactive as needed (better performance)
- **Event replay** -- Captures clicks before hydration completes

**Learn more:**
- Hydration guide -- Complete hydration setup
- Incremental hydration -- Advanced hydration with `@defer` blocks

---

## Next Steps

- Server-Side Rendering
- Hydration
- Incremental Hydration
