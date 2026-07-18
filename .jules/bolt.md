## 2023-10-27 - [Astro getStaticPaths Pagination Anti-pattern]
**Learning:** In Astro's `[...page].astro` dynamic routes, redundantly fetching and sorting collections (`getCollection`) in the component script is a significant performance bottleneck and memory waste. The `getStaticPaths` function already fetches, sorts, and paginates the data, exposing it via the `page` prop (`Astro.props.page.data`).
**Action:** When working with Astro paginated routes, always check if `getCollection` is being called in the render scope. Refactor to use `const { page } = Astro.props` and map over `page.data` instead to avoid O(n log n) overhead on every render.

## 2023-10-27 - [Parallelize Multiple Astro Content Collection Fetches]
**Learning:** Sequential `getCollection` calls in Astro pages (e.g., `const blogs = await getCollection('blog'); const projects = await getCollection('project');`) cause unnecessary I/O and parsing bottlenecks during the build process or SSR, as each collection fetch blocks the next.
**Action:** Always use `Promise.all` when a page requires fetching multiple distinct collections simultaneously (e.g., `const [blogs, projects] = await Promise.all([getCollection('blog'), getCollection('project')]);`) to optimize data fetching performance.
## 2024-05-30 - Client Directives in Astro Above-the-fold
**Learning:** `client:visible` for components that are always visible initially (like headers) adds unnecessary `IntersectionObserver` overhead without deferring work.
**Action:** Use `client:idle` to actually defer hydration for non-critical interactive components that are immediately visible, freeing up the main thread during initial page load.

## 2026-06-20 - [Replace Unthrottled Resize Listeners with MatchMedia]
**Learning:** Using `window.addEventListener('resize', ...)` to track breakpoints in React components (like responsive nav menus) is a performance anti-pattern. It fires hundreds of times during a resize operation, causing synchronous evaluations of `window.innerWidth` on the main thread, which can lead to UI jank and layout thrashing.
**Action:** When tracking CSS breakpoints in JS/React, always use `window.matchMedia('(min-width: ...px)').addEventListener('change', ...)`. This native API is vastly more efficient as it fires exactly once when the specific breakpoint is crossed, completely eliminating the continuous overhead of the resize event.

## 2026-06-28 - [Cache Expensive Sync Operations During Astro Build]
**Learning:** In Astro static site generation, synchronous operations like `execSync` placed in layout or frequently used component scripts (like `Footer.astro`) execute individually for *every* page generated. This sequentially blocks the main thread, causing severe build slowdowns as the site grows.
**Action:** Always cache the results of expensive, invariant synchronous operations (like fetching git commit info) in the module scope or using `globalThis` so they execute only once per build process, significantly improving `pnpm build` times.

## 2023-10-27 - [Enable Astro Prefetching and ClientRouter]
**Learning:** Out-of-the-box Astro performs full page reloads on navigation. For content-heavy sites (like blogs or portfolios), this creates unnecessary latency and screen flashes.
**Action:** Always enable `prefetch` in `astro.config.ts` (e.g., `prefetch: { prefetchAll: true, defaultStrategy: 'hover' }`) to fetch resources before the user clicks, and include `<ClientRouter />` (from `astro:transitions`) in the `<head>` to enable SPA-like in-place DOM updates for near-instant navigations.

## 2024-05-30 - [Eliminate Redundant React Islands & Inline Theme Scripts]
**Learning:** Rendering multiple instances of the same React component (like `ThemeSwitcher` for desktop and mobile) inside Astro islands (`client:idle`) forces the client to download, parse, and hydrate multiple identical component states, duplicating DOM nodes and event listeners unnecessarily. Furthermore, calling theme initialization functions (like `colorMode()`) immediately after an identical inline `<script>` already blocked parsing in `<head>` causes redundant and expensive layout thrashing.
**Action:** Structure layouts to share a single React interactive component instance when possible, using CSS breakpoints (`hidden md:flex`) to display it contextually. Also, audit theme toggle scripts to ensure initial application logic only executes once per page load to prevent FOUC without duplicate execution overhead.

## 2024-05-30 - [Defer Hydration of Hidden Mobile Components]
**Learning:** Using `client:idle` for conditionally hidden components (like a mobile nav menu with `md:hidden`) causes desktop clients to needlessly download and hydrate JavaScript that they will never use, wasting bandwidth and CPU cycles.
**Action:** When a component is only visible on certain viewports via CSS breakpoints, use the corresponding Astro `client:media` directive (e.g., `client:media="(max-width: 767px)"`) to guarantee the JS is only loaded and executed when the component can actually be interacted with.
