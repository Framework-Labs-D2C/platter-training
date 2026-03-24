# platter-training

Platter internal training flowcharts — interactive visual guides for understanding key frontend architecture patterns.

## Flowcharts

### [Product Card Rendering — Static vs Dynamic Flow](https://framework-labs-d2c.github.io/platter-training/product-card-rendering-flowchart.html)

How product cards render across static and dynamic paths. Covers Liquid template branching, metaobject data resolution, JavaScript hydration, and fallback strategies.

### [Modal & Dynamic Card Lazy Loading — SWR Pipeline](https://framework-labs-d2c.github.io/platter-training/modal-swr-lazy-loading-flowchart.html)

The lazy loading pipeline for modals and dynamic product cards. Covers SWR cache lifecycle (stale/fresh/miss), Liquid server rendering, fetch revalidation, and modal trigger flows.

### [Product JSON Data Loading — Three-Tier Hydration Cascade](https://framework-labs-d2c.github.io/platter-training/product-json-data-loading-flowchart.html)

How product JSON data is sourced and loaded via a three-tier cascade. Covers server-side Liquid rendering, DOM parsing, IndexedDB caching, and fetch-based revalidation.

### [Page Transitions, Render Tabs & Liquid Variable Hacking](https://framework-labs-d2c.github.io/platter-training/page-transitions-flowchart.html)

SPA-like page transitions using Barba.js. Covers the layout wrapper structure, theme settings configuration, initialization flow, the 6 navigation lifecycle hooks, and route exclusions.

### [Theme Editor Integration & Building for Customization](https://framework-labs-d2c.github.io/platter-training/theme-editor-integration-flowchart.html)

How the Shopify Theme Editor works under the hood and how to build features that are fully customizable inside it. Covers the iframe architecture, design mode detection, the six editor events, schema-to-UI mapping, block select & tab focus, design-mode branching patterns, live updates vs re-renders, and testing.

### [How to Add a Block to Product Cards — Testimonial Block Example](https://framework-labs-d2c.github.io/platter-training/adding-a-block-flowchart.html)

Step-by-step guide for adding a new block to product cards, using a testimonial block as an example. Covers schema definition, Liquid rendering, CSS styling, JavaScript hydration, and dynamic card support.
