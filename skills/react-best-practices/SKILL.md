---
name: react-best-practices
description: Performance rules for React and Next.js — request waterfalls, bundle size, server-side cost, client data fetching, re-render churn, rendering/paint cost, JS hot paths. Use when writing or refactoring React components, reviewing a React PR for performance, diagnosing slow page loads or janky interactions, or deciding where to put Suspense, memo, dynamic imports, or a server/client boundary.
---

# React Best Practices

40+ performance rules with incorrect/correct code pairs and impact estimates.

## How to use this

**Load only the file you need.** Match the symptom below, read that one reference, stop.
Don't read them all — the whole point of the split is that you don't have to.

If you don't yet know which section applies, you don't have enough information to fix
anything. Measure first (network waterfall, profiler, bundle analyzer), then come back.

| # | Read this file | When | Impact |
|---|---|---|---|
| 1 | `references/1-waterfalls.md` | Sequential `await`s, slow initial load, request chains | **CRITICAL** |
| 2 | `references/2-bundle-size.md` | Large JS payload, slow TTI, heavy imports | **CRITICAL** |
| 3 | `references/3-server-side.md` | Slow server response, repeated/duplicate DB queries | HIGH |
| 4 | `references/4-client-data-fetching.md` | Redundant client requests, duplicated listeners | MEDIUM-HIGH |
| 5 | `references/5-re-renders.md` | Components re-render on unrelated state changes | MEDIUM |
| 6 | `references/6-rendering.md` | Jank, long lists, hydration flicker, SVG/animation cost | MEDIUM |
| 7 | `references/7-javascript.md` | CPU-bound loops, lookups, array work in hot paths | LOW-MEDIUM |
| 8 | `references/8-advanced-patterns.md` | Stale closures, unstable callback identity | LOW |

## Rule index

Enough detail to pick a file without opening one.

**1. Eliminating Waterfalls** — defer await until needed · dependency-based parallelization ·
prevent waterfall chains in API endpoints · `Promise.all()` for independent operations ·
strategic Suspense boundaries

**2. Bundle Size** — avoid barrel file imports · conditional module loading · defer
non-critical third-party libraries · dynamic imports for heavy components · preload on user
intent

**3. Server-Side** — cross-request LRU caching · minimize data transfer to client · parallel
fetching via component composition · per-request dedup with `React.cache()`

**4. Client-Side Data Fetching** — deduplicate global event listeners · SWR for automatic
deduplication

**5. Re-render Optimization** — defer state reads to usage point · extract to memoized
components · narrow effect dependencies · subscribe to derived state · functional `setState` ·
lazy state initialization · transitions for non-urgent updates

**6. Rendering Performance** — animate SVG wrapper not element · CSS `content-visibility` for
long lists · hoist static JSX · optimize SVG precision · prevent hydration mismatch without
flicker · `Activity` for show/hide · explicit conditional rendering

**7. JavaScript Performance** — batch DOM CSS changes · index maps for repeated lookups ·
cache property access in loops · cache repeated calls · cache storage API calls · combine
array iterations · early length check · early return · hoist RegExp · loop for min/max over
sort · `Set`/`Map` for O(1) lookups · `toSorted()` for immutability

**8. Advanced Patterns** — store event handlers in refs · `useLatest` for stable callback refs

## Priority

Fix in numeric order. The early sections dominate: a perfectly memoized tree still loads
slowly behind a request waterfall, and section 7 micro-optimizations are noise until 1 and 2
are clean.

## Before applying anything

These rules trade readability for speed. Spending that trade on a path that isn't hot is a
net loss. Confirm the bottleneck with a measurement before refactoring against a rule.

## References

- [react.dev](https://react.dev) · [swr.vercel.app](https://swr.vercel.app)
- [better-all](https://github.com/shuding/better-all) · [node-lru-cache](https://github.com/isaacs/node-lru-cache)
- [Optimizing package imports in Next.js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
- [Making the Vercel dashboard twice as fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)
