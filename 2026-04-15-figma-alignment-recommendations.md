# Figma File Alignment — Deep Analysis & Improvement Options

**Date:** 2026-04-15
**Figma file:** `wyE5yCzCkqOREAa26brmGu` (Accessories Vertical)
**Code reference:** `themes/production/` (Platter v4.4.4)
**Prior analysis:** `.ai/notes/walkthrough.md` (10-step Figma→code translation test)
**Context:** Prior walkthrough proved what IS and ISN'T extractable from Figma. This doc inverts the question: **what should change in the Figma file to narrow the gap?**

---

## Coverage Snapshot

### Code inventory (via exhaustive audit)
- **54** `.liquid` section files + **7** `0_*.json` section groups (61 section definitions)
- **239** block files across 23 block families (biggest: `_main_product__*` = 36, `_main_search__*` = 19, `_card_product_card__*` = 18, `_main_collection__*` = 17, `_main_cart__*` = 16)
- **127** design-system component definitions in `0_global.json`: 70+ icons, 13+ swatches, 12+ scrollbar classes, 6+ typography classes, 8 button styles, 6 accordion variants
- **5 card types** in `0_card.json`: product, line_item, article, collection, page (+ blog, comment via family)
- **10 modal sections** in `0_modal.json`
- **7 CTA states** on product buttons: `default / variant / subscription / bundle / pdp / preorder / sold_out`
- **7 smart metafield keys** actively referenced via bracket-notation DSL (e.g., `[selected_variant.metafields.smart.images.0]`)
- **21 templates** (14 JSON + 7 customer JSON + 1 gift_card liquid)
- **Bracket DSL**: top patterns `[price]` (36×), `[compare_at_price]` (16×), `[product.title]` (10×), `[icon.lock]` (13×), `[icon.timer]` (9×), plus `{s}`, `[br]`, `[icon.*]` extensions

### Figma inventory (via MCP walk)
- **2 pages only**: `COVER` (15346:2) and `Designs` (5212:261) — everything lives on one canvas
- **Designs page breakdown**: 7 `<section>` groups (Homepage, Megamenu, Additional Assets, PDP, Cart Drawer, Quickshop, Collection Page) + several stray `<frame>`s (Hero Section, hero banner, hero banner desktop/mobile) orphaned at the canvas root
- **Each major layout doubled Desktop + Mobile as separate frames**, not variants (e.g., `Homepage Desktop` + `Homepage Mobile`, `PDP Desktop` + `PDP Mobile`, `Megamenu/Cases Desktop` + `Megamenu/Cases Mobile`)
- **Only 1 component with a proper 2D variant matrix**: Hotspot (`State=Default/Selected × Size=Desktop/Mobile` = 4 symbols)
- **~40 variables extracted** across typography, spacing, colors, effects — with notable duplication/inconsistency (see §2.3)
- **All 122 extracted images** are from page `Designs` — no dedicated Typography or Global Components page

---

## 1. Alignment Gap Analysis

### 1.1 Tokens (Figma variables ↔ CSS custom properties)

| Code CSS custom property family | Figma variable equivalent | Alignment |
|---|---|---|
| `--typography-family-primary` | `typography/family/primary` (Cormorant Garamond) | ✅ Exact 1:1 |
| `--typography-family-tertiary` | `typography/family/tertiary` (Clash Display) | ✅ Exact 1:1 |
| `--typography-family-secondary` | ❌ **MISSING from Figma** | 🔴 Gap — code has the slot, Figma doesn't |
| `--typography-richtext-md-h1..h6` | `typography/richtext-md/h1..h6` (56/42/28/24/16/12) | ✅ Exact 1:1 |
| `--typography-richtext-md-p1` | `typography/richtext-md/p1` (15) | ✅ Exact 1:1 |
| `--typography-single-type-styles-button` | `typography/single type styles/button` (14, or 12 in Filter Drawer) | ⚠️ Mostly 1:1 but **button size 14 ≠ 12** between contexts |
| `--typography-single-type-styles-nav-link` | `typography/single type styles/nav link` (14) | ✅ Exact 1:1 |
| `--colors-brand-colors-dark-700/800` | `colors/brand colors/dark-700` (#2c422a), `dark-800` (#493831) | ⚠️ Figma value diverges from production (#2c422a vs prod #385235 — see walkthrough.md) |
| `--colors-brand-colors-medium-400/600` | `colors/brand colors/medium-400` (#757068), `medium-600` (#cdc9c7) | ✅ 1:1 |
| `--colors-brand-colors-light-200/300` | `colors/brand colors/light-200` (#faf6f0), `light-300` (#e6e1df) | ✅ 1:1 |
| `--colors-neutrals-gray-100..300` | `colors/neutrals/gray-100/200/300` | ✅ Partial — code references higher indices `gray-*` that Figma doesn't define |
| `--colors-neutrals-white/black` | `colors/neutrals/white/black` | ✅ But see §1.3 — duplicates exist |
| `--color-text-main/accent/secondary` | `sections/text-main`, `text-accent`, `text-secondary` | ✅ Figma **has** these semantic slots — but naming `sections/*` doesn't match code's `--color-text-*` convention |
| `--color-bg-main/accent` | `sections/background-main`, `background-accent` | ✅ Figma has but naming diverges |
| `--color-error` | `components/component-error` (#771010) | ✅ Present |
| `--container-md` / `--container-xs/sm/lg` | ❌ **Only** `spacing/Container Margin Spacing 2` (72) | 🔴 Code has multi-size container system; Figma has a single "margin spacing" value |
| `--grid-cols`, `--grid-gap` | ❌ Not defined | 🔴 |
| `--navigation-bar-height` | ❌ Not defined | 🔴 |
| `--announcement-bar-height` | ❌ Not defined | 🔴 |
| Drop shadow token | `Drop Shadow` effect (#0000001A, 0/0/6/2) | ✅ Present |

### 1.2 Page structure (Figma pages ↔ template files)

| Code artifact | Should map to Figma page | Currently in Figma |
|---|---|---|
| `templates/index.json` | `Homepage` page | ✅ `Designs → Homepage` section |
| `templates/product.json` | `PDP` page | ✅ `Designs → PDP` section |
| `templates/collection.json` | `Collection Page` page | ✅ `Designs → Collection Page` section |
| `templates/cart.json` | `Cart Page` page | 🔴 **Missing** — only Cart Drawer exists (modal) |
| `templates/article.json` | `Article` page | 🔴 **Missing** |
| `templates/blog.json` | `Blog` page | 🔴 **Missing** |
| `templates/search.json` | `Search` page | 🔴 **Missing** (only search icon/modal visible) |
| `templates/page.json` + `page.contact.json` + `page.bundle.json` | `Pages` (generic + contact + bundle) | 🔴 **Missing** |
| `templates/404.json`, `password.json` | `Error states` page | 🔴 **Missing** |
| `customers/*.json` (7 templates: account, login, register, order, addresses, activate, reset) | `Customer Account` page | 🔴 **Entirely absent** |
| `list-collections.json` | `Collection List` page | 🔴 **Missing** |
| `gift_card.liquid` | `Gift Card` page | 🔴 **Missing** |
| `sections/0_global.json` (127 component definitions) | `Global Components` + `Typography` pages | 🔴 **Missing** — no dedicated design-system pages |
| `sections/0_modal.json` (10 modals) | `Modals` page | ⚠️ Partial — only Cart Drawer + Quickshop + UGC Stories appear; 7 modal types not shown |
| `sections/0_megamenu.json` | `Megamenu` page | ✅ `Designs → Megamenu` (but 4 of ~top-level links only) |
| `sections/0_header.json` | `Header` page | 🔴 No dedicated frame — header appears inlined in every page layout |
| `sections/0_footer.json` | `Footer` page | 🔴 No dedicated frame — footer inlined in every layout |
| `sections/0_card.json` (5+ card configs) | `Card Configurations` page | 🔴 **Missing** |

**Single biggest gap:** 7 template files + all customer account screens + all standalone pages have zero Figma presence. For a theme sold to 60-80 clients, the Figma file only designs the "glamour" surfaces.

### 1.3 Variable hygiene (internal Figma issues)

| Symptom | Evidence | Impact |
|---|---|---|
| **Duplicate token names with different values** | `colors/neutrals/black transparent` = `#00000066` vs `Neutral/Black Transparent` = `#000000` (both exist) | 🔴 Critical — picking the wrong one silently breaks transparency |
| **Duplicate tokens same value** | `colors/neutrals/black` + `Neutral/Black` = `#000000` | ⚠️ Noise; confuses auto-extraction |
| **Orphan "2" suffixes** | `spacing/spacing-xl 2` (48), `spacing/spacing-large 2` (40), `spacing/Container Margin Spacing 2` (72) | 🔴 Suggests failed rename/merge; clutters token picker |
| **Inverted size ordering** | `spacing-xl` = 64 but `spacing-xl 2` = 48 (smaller) | 🔴 Breaks linear scale assumption |
| **Inconsistent casing** | `typography/weight/Medium` (cap.) vs `typography/weight/regular` / `bold` (lowercase) | ⚠️ Auto-scripts mis-lookup |
| **Context-sensitive same-named tokens** | `Page Width` = 1860 on most nodes, 960 in Bundler Module | 🔴 Looks like the same token; actually a different value bound |
| **`spacing-xl` vs `spacing-xl 2`** mapped to different mobile vs desktop spacing — inconsistent naming | Grep shows 48 used as mobile, 64 as desktop, but neither name says so | ⚠️ |
| **Missing secondary font slot** | Code references `--typography-family-secondary` but Figma only binds primary + tertiary | 🔴 Renders a default fallback in any layer that binds secondary |

### 1.4 Component library gaps

| Code concept | Figma presence | Gap severity |
|---|---|---|
| **Button states** (default/hover/focus/disabled × primary/secondary/search/tabs × sm/md) — code has 8 button classes | Inline button instances per frame, no consolidated component with variants | 🔴 — Walkthrough Step 2 already flagged this |
| **CTA button states** (default / variant / subscription / bundle / pdp / preorder / sold_out) | Figma shows 1 state (default) | 🔴 Invisible 6 states; AI must guess from code template |
| **Label variants** (`label-primary` / `label-low` / `label-sold` / `label-low-inventory` + metafield-driven discount/preorder/out-of-stock) | Static "NEW" text in one product card | 🔴 |
| **Product Card variants** (regular vs flat vs mini — inferred from UGC Stories showing `product card flat`) | "product card" + "product card flat" + "collection card flat" exist as instances with no shared variant system | ⚠️ Close — needs consolidation |
| **Line item card** (cart drawer) — 12 block types in code | Inside sealed Cart Drawer instance | ⚠️ |
| **Input types** (text, email, password, select, textarea, tag, button) | Filter Drawer hints at select + input but no enumerated input component | 🔴 |
| **Scrollbar / pagination classes** (12+ scrollbar variants, dot pagination) | Implicit in sliders, no dedicated component | 🔴 |
| **Accordion variants** (4: filters, pdp, sidebar, no) | Inline in PDP features / cart / filters — no enumerated component | 🔴 |
| **Swatch types** (13+ color + gradient swatches) | Inline squares in product cards — no enumerated swatch library | 🔴 |
| **Icon library** (70+ icons) | Embedded as instances but not in a grid/sheet; names not visible as Figma component names | 🔴 Can't verify all 70 exist |
| **Quantity selectors** (2 variants) | Single inline instance | ⚠️ |
| **Hotspot** (1 section, but mature) | ✅ 4 proper variants (State × Size) | ✅ **Only mature component** |
| **Modals** (10 types in code) | 3 visible (Cart Drawer, Quickshop, UGC Stories Modal) | 🔴 7 missing: search, country selector, sidebar menu, sidebar submenu, dynamic product, popup, back-in-stock, media gallery |

### 1.5 Dynamic-data concepts with no Figma equivalent

These are "by design" uncapturable in static design, but Figma can be *annotated* to surface them:

| Code concept | How it should be hinted in Figma |
|---|---|
| Bracket notation (`[product.title]`, `[price]`) | Text layers using placeholder syntax instead of sample values |
| Metafield source (`[product.metafields.smart.labels]`) | Annotation / description on label layers |
| CTA state variants (7 states) | Component variants explicitly named by state |
| Hide-if-empty behavior | Layer naming convention (e.g., prefix `?` for optional blocks) |
| Scroll reveal, scroll snap, auto-rotate | Component description field + reserved layer names |
| Preload / `sizes` / `max_width` hints | Image layer description field |
| `x-defer` / deferred hydration | Not needed — invisible to designer |
| Section disabled/enabled groups | Not needed |
| `hide_if_empty` at 3 levels (section/container/block) | Layer naming or description |

### 1.6 Naming alignment (Figma ↔ code section types)

| Figma name | Code section | Delta |
|---|---|---|
| `Slideshow` | `slideshow.liquid` | ✅ exact |
| `Product Slider` | `generic_section` + `slider` block + `cards_product` block | ⚠️ Figma hides 3-layer wiring |
| `Image with Text` / `image with text` (inconsistent caps) | `image_with_text.liquid` | ⚠️ Capitalization varies on 3 frames |
| `Collection Grid` | `generic_section` + `cards_collection` block (or `main_collection` on collection page) | ⚠️ Dual meaning — tool call can't tell which |
| `Hero Section` | `hero_banner.liquid` | 🔴 Name mismatch — code never calls it "Hero Section" |
| `buy box` | `main_product.liquid` | 🔴 "Buy box" is a subset of the main_product section (just the right column) |
| `Bundler Module` | `bundle.liquid` / `bundle_with_tabs.liquid` | 🔴 "Module" suffix non-canonical |
| `UGC Video Slider` | `cards_ugc_video` block type | ⚠️ The block exists; the "slider" wrapper is `slider` block |
| `Article Bento Grid` | No direct section — likely `generic_section` + richtext + cards_article | 🔴 No canonical name |
| `Press (Marquee Bar)` | Likely marquee/content_banner composition | 🔴 No canonical name |
| `Marquee Bar` | No dedicated section — reuses slider | 🔴 |
| `Sticky PDP Bar` | `main_product_bar.liquid` | 🔴 Name mismatch |
| `Image Banner` | `content_banner.liquid`? | 🔴 Unknown mapping |
| `Newsletter Subscription` | `form` blocks in `generic_section` | 🔴 No dedicated section |
| `Filter Bar` / `Filter Drawer` | `_main_collection__filter*` blocks | 🔴 Dual naming for similar concept |
| `Integration` (PDP) | `featured_benefits_slider.liquid`? | 🔴 Opaque |
| `hero banner` (mobile, lowercase) | Duplicate of `Hero Section` | 🔴 Casing + naming inconsistency |

---

## 2. Recommendations — Tiered Options

Each recommendation carries: **What**, **Why**, **Effort** (S/M/L/XL), **Tradeoff**.

---

### Tier 1 — Quick Wins (low effort, high AI-extraction payoff)

#### 1.1 Rename orphan tokens — **S**
- **What:** Delete or merge the `2`-suffix tokens. Canonical scale should be `spacing/xxs/xs/s/m/l/xl/2xl/3xl/4xl` with single values per step.
- **Merges:**
  - `spacing-xl` (64) stays; rename `spacing-xl 2` (48) → `spacing-lg` (reorder so lg<xl).
  - `spacing-large` (32) → `spacing-md-lg` or `spacing-28-32`.
  - `spacing-large 2` (40) → `spacing-lg-alt` or drop if unused.
  - `Container Margin Spacing 2` (72) → `container/gutter-md` (matches code's container system).
- **Why:** Auto-scripts that regex Figma tokens to CSS `--spacing-*` pick up `2`-suffixes as separate tokens, polluting the generated design-token file. Also, Figma's token picker shows all duplicates, so designers pick the wrong one.
- **Tradeoff:** A one-time rebinding pass in Figma; old layers reference-tracked via Figma's rename propagation.

#### 1.2 Resolve duplicate black/white tokens — **S**
- **What:** Delete `Neutral/Black` and `Neutral/Black Transparent`. Keep `colors/neutrals/black` and `colors/neutrals/black transparent`. (Or inverse — pick one path.)
- **Why:** `Neutral/Black Transparent` = `#000000` (no alpha) silently replaces the alpha token in any layer that rebinds; this is a real bug waiting to ship.
- **Tradeoff:** Check the 3-4 layers currently bound to the duplicates.

#### 1.3 Normalize weight casing — **S**
- **What:** `typography/weight/Medium` → `typography/weight/medium`.
- **Why:** Auto-extraction maps `typography/weight/*` → CSS `font-weight` lookup. `Medium` vs `medium` forces script edge cases.

#### 1.4 Fix `Page Width` context collision — **S**
- **What:** Rename `Page Width` (1860) → `breakpoint/desktop-max`. The 960 value in Bundler Module → `breakpoint/tablet-max` or its own token.
- **Why:** Single name, two values is a design-token cardinal sin.

#### 1.5 Add the missing secondary font slot — **S**
- **What:** Create `typography/family/secondary` even if same as primary for now, so layers can bind. Code already references `--typography-family-secondary`.
- **Why:** Prevents null-token fallback in code paths that use secondary.

#### 1.6 Consolidate Hero naming — **S**
- **What:** Rename `Hero Section` → `hero_banner` (matches `hero_banner.liquid`). Same for lowercase `hero banner` duplicates.
- **Why:** Automated Figma→section-type mapping is 1-step instead of lookup-table.

#### 1.7 Rename "buy box" frame to "main_product" or "Product Page Detail" — **S**
- **What:** The `buy box` frame inside PDP Desktop/Mobile currently hides that the left column (PDP Images) and the right column (PDP Buy Box) together ARE `main_product`. Combine them under one frame named `main_product`.
- **Why:** Walkthrough Step 7 flagged this: PDP is `main_product`, not two separate systems.

#### 1.8 Add a file-level README / About frame — **S**
- **What:** On the COVER page, add a description frame explaining: which pages cover which Shopify template, which components are canonical, token naming conventions, and a pointer to this repo's `CLAUDE.md`.
- **Why:** Anyone picking up the file (AI or human) wastes ~15 minutes orienting otherwise. Walkthrough Step 1 manually reconstructed this mapping.

---

### Tier 2 — Structural Cleanups (medium effort, unblocks automation)

#### 2.1 Split into dedicated pages per template/section-group — **M**
- **What:** Replace single `Designs` canvas with 8-10 pages. Suggested structure:

```
- 00 Cover / Readme
- 01 Foundations (Tokens, Color, Spacing, Grid)
- 02 Typography (richtext-md × mobile/desktop × all weights)
- 03 Global Components (buttons, labels, inputs, accordions, swatches, icons, pagination, scrollbars)
- 04 Card Configurations (product-card handles × product-card-flat, article, collection, line_item, page)
- 05 Header + Megamenu
- 06 Modals (cart, search, sidebar, quickshop, media gallery, popups, country selector, back-in-stock)
- 07 Footer
- 10 Homepage
- 11 PDP
- 12 Collection + Collection List
- 13 Cart Page
- 14 Search Results
- 15 Blog + Article
- 16 Standard Pages (page.json, contact, bundle)
- 17 Customer Account (account, login, register, order, addresses, activate, reset)
- 18 Error & System (404, password, gift card)
```
- **Why:** Mirrors template file layout. AI can Figma→page→code-template in a lookup instead of guessing.
- **Tradeoff:** Reorganizes ~100 existing frames. Preserves URLs via Figma's stable node IDs, but bookmarks break.

#### 2.2 Unify Desktop + Mobile into component variants — **M→L**
- **What:** Every section that currently has `X Desktop` + `X Mobile` as separate top-level frames becomes ONE component with `breakpoint=desktop/mobile` property. Affects: Homepage, PDP, Collection Page, Cart (Empty/Filled × Desktop/Mobile = 4 currently), Megamenu (Cases/Wallets/Gear/About × Desktop/Mobile = 8 currently), hero banner, etc.
- **Why:**
  - Changes to a section only need one edit, not two.
  - Component variants are first-class in Figma's design-spec export; walkthrough showed the Hotspot's `State × Size` matrix was the best-understood component.
  - AI can diff "desktop vs mobile" by reading one component's properties, not by cross-referencing two separate frames.
- **Tradeoff:** Large one-time conversion. Benefit compounds across every future edit.

#### 2.3 Build a real button component with all 8 classes × 3 states × 2 sizes — **M**
- **What:** One Figma component, 8 × 3 × 2 = 48 variants: `class={primary, secondary, search, tabs, filter, icon, pagination, ghost}` × `state={default, hover, disabled}` × `size={md, sm}`.
- **Why:** Walkthrough Step 2 noted button color divergence and inability to extract hover states; this component gives the full matrix. Code has exactly these 8 classes — the mapping is already 1:1.
- **Tradeoff:** Audit to confirm the exact 8 classes (code currently shows `button-primary`, `button-secondary`, `button-search`, `button-tabs` at minimum — confirm the other 4 or collapse to 4).

#### 2.4 Build a Product Card master with CTA state variants — **M**
- **What:** One ProductCard component with properties matching the code's block composition:

```
- variant       = { product-card, product-card-flat, product-card-mini }
- showButton    = bool
- showColorSwatches = bool
- showLabels    = bool
- showPrice     = bool
- showReviews   = bool
- showDropdown  = bool
- showTextVariants = bool
- showToggle    = bool
- cta_state     = { default, variant, subscription, bundle, pdp, preorder, sold_out }
- inventory_state = { in_stock, low_inventory, sold_out }
```
- **Why:** Walkthrough Step 5 identified this as THE critical wiring signal. Currently `showX` props exist in one variant but not others; CTA state is entirely absent.
- **Tradeoff:** The 7 CTA states × 3 cards × 3 inventory states = 63 sub-variants is unwieldy. Compromise: CTA state as a discrete variant axis (7 values) + inventory as overlay badges, collapsing combinatorics.

#### 2.5 Label component with the 4+ classes — **S**
- **What:** `Label` component with `variant={primary, low, sold, low-inventory, preorder, discount}`.
- **Why:** Walkthrough Step 2 noted labels diverge heavily; a component pins them. Also removes static "NEW" text that's really a dynamic metafield.

#### 2.6 Documentation annotation on dynamic-content layers — **M**
- **What:** For every text layer that will be dynamic in code, rename the layer with a DSL-style placeholder instead of sample text:
  - `"$189.00"` → `"[price]"`
  - `"Case - Black"` → `"[product.title]"`
  - `"5M+ reviews"` → `"[reviews_heading]"` (or keep literal if it's static)
  - `"NEW"` label → `"[product.metafields.smart.labels]"`
- **Why:** Removes the ambiguity Walkthrough Step 5 called out: Figma's static "$189" silently becomes `[price]` with discount logic. Layer names shown in metadata extract let the AI skip this translation step.
- **Tradeoff:** Requires discipline. Consider a Figma plugin that annotates via description instead of name.

#### 2.7 Consolidate Marquee Bar / Press / Image Slider / Marquee variants — **M**
- **What:** Identify what Figma calls separately that all resolve to the same section type in code:
  - `Marquee Bar`, `Press (Marquee Bar)`, `UGC Video Slider`, `Image Slider`, `Image Banner` → likely all `generic_section` + `slider` block with different content.
  - Pick canonical names that match the code section/block vocabulary.
- **Why:** AI sees 5 unique component names and can't tell they're the same section template with different configs. One canonical "horizontal content slider" component with a content-type prop unifies.

---

### Tier 3 — Aspirational (high effort, systemic payoff)

#### 3.1 Publish a proper Design System library — **L**
- **What:** Extract Foundations + Global Components into a shared Figma library (separate file). The Accessories Vertical (and future Gains-in-Bulk, etc.) files consume from the library.
- **Why:** Currently every client's Figma is a standalone copy. Any foundation update (e.g., a new label variant) must be manually replicated. A library gives:
  - Centralized token + component updates
  - "Override color" workflow for each client
  - Clean separation: library = code-shape, client files = merchandising choices
- **Tradeoff:** Requires org-level Figma plan + team discipline. Biggest payoff for Felix's 60-80-client fleet model.

#### 3.2 Code Connect mapping — **L**
- **What:** Use Figma's Code Connect feature (already available via the MCP server) to annotate each canonical component with its production Liquid section/block name. E.g., `Product Card` → `_card_product_card__*` blocks; `Slideshow` → `slideshow.liquid`.
- **Why:** Walkthrough identified component-name → block-type mapping as a top-5 friction. Code Connect makes it machine-readable.
- **Tradeoff:** Ongoing mapping maintenance. Each new block family needs a Code Connect entry.

#### 3.3 Separate Merchandising-Intent frames from Design-System frames — **M**
- **What:** Every design lives in EITHER:
  - **System frames**: represent the code's component shapes (what a `card_product_card__container` can render)
  - **Merchandising frames**: represent a specific client's configuration (what THIS product should show)
- **Why:** Today, the Figma designs conflate both. A frame showing "Case - Black, $189" is a merchandising choice, not a design-system definition. If the designer later decides swatches should move above price (a system-level change), it only needs to happen once in the system frames, not in every merchandising instance.

#### 3.4 Add hidden "spec" layers with metadata — **M**
- **What:** Behind every section frame, add a hidden "spec" frame with text like:
  ```
  section: main_product
  template: product.json
  blocks: gallery, container(rating,title,price,options,cta,features)
  data: product.metafields.smart.labels, selected_variant.metafields.smart.images[0..2]
  preload_images: 3
  scroll_snap: item
  ```
- **Why:** All the code-only concepts (preload, x-defer, hide_if_empty, metafield sources) become machine-readable without cluttering the visual design.
- **Tradeoff:** Must stay in sync with code. Consider auto-generating these from a single source of truth.

#### 3.5 Token doc page with mobile/tablet/desktop columns — **M**
- **What:** On the Typography page (Tier 2 split), lay out each class in 3 columns: `<768px`, `768-1200px`, `>1200px`. Today, only Desktop is visible in vars; mobile comes from frame-height derivation. Middle breakpoint (1200-768px) exists in Figma but code only uses `@media (max-width: 768px)` — either collapse Figma to match or add the middle breakpoint in code (decision point for Felix).
- **Why:** Walkthrough IMPORTANT note 1.2: "Middle breakpoint (1200-768px) exists in Figma but production only uses a single `@media (max-width: 768px)`. The tablet breakpoint from Figma is unused."

#### 3.6 Build missing modal designs — **L**
- **What:** Design the 7 modal types currently absent from Figma:
  - `modal_search` (search overlay)
  - `modal_country_selector`
  - `modal_sidebar_menu` + `modal_sidebar_submenu` (mobile nav)
  - `modal_media_gallery` (PDP lightbox)
  - `modal_dynamic_product`
  - `modal_popup` (announcement / promo)
  - `modal_back_in_stock_notification`
- **Why:** Currently all 7 are either "dev picks whatever" or "copy from another theme." A designed reference reduces implementation ambiguity and gives QA a visual target.

#### 3.7 Customer Account templates — **L**
- **What:** Design the 7 customer account screens.
- **Why:** Shopify stores ship with unstyled customer account pages by default. These are routinely skipped in design handoffs and end up with ugly defaults in production.

#### 3.8 Cart Page (separate from Cart Drawer) — **M**
- **What:** Design the `cart.json` page layout — 2-column grid with sticky sidebar (per code inspection in walkthrough Step 8).
- **Why:** Walkthrough confirmed: `main_cart` is context-aware, drawer vs page differ structurally. Drawer alone is designed.

---

### Tier 4 — Process / Governance (mostly not in Figma itself)

#### 4.1 Token → CSS variable auto-export — **M (tooling)**
- **What:** Build a script (can be a companion to `.ai/scripts/figma-images.ts`) that reads all Figma variables via REST API and regenerates the `0_global.json` design-token block. Delta review before commit.
- **Why:** Eliminates the "Figma says #2c422a, production uses #385235" divergence. Source of truth becomes Figma.
- **Tradeoff:** Only works AFTER token hygiene (Tier 1). Tokens must be clean before they become source of truth.

#### 4.2 Naming lint on Figma file — **S**
- **What:** Another script: scan the Figma file via REST API, flag layers whose names don't map to a known code vocabulary (section names, block names, card handles, token names).
- **Why:** Catches drift. E.g., a designer adds a "Hero v2" frame without renaming → lint flags.

#### 4.3 Walkthrough → workflow doc integration — **S**
- **What:** The findings in `.ai/notes/walkthrough.md` (already thorough) + this doc should be distilled into `.ai/workflows/figma-alignment.md` as a living checklist that evolves as Figma improves.
- **Why:** These are one-shot analyses today. Need a "current state of alignment" doc the team refers to.

---

## 3. Recommended Sequencing (for Felix)

Given that the repo is the platform for 60-80 clients and the Figma file is the source for design decisions:

### Phase A — Unblock AI extraction (2-4 hours of work)
1. Fix the Tier 1 quick wins (§1.1 through §1.8). One sitting, one PR.
2. Add hero naming fix + file README.
3. **Outcome:** Token extraction scripts and AI workflow stop producing garbage.

### Phase B — Page reorganization (1 day)
1. §2.1 page split.
2. §2.2 Desktop+Mobile variant conversion for the top 3 section types (Hero, Product Slider, Image with Text).
3. **Outcome:** Figma becomes navigable matching the code layout.

### Phase C — Component library bootstrap (1-2 days)
1. §2.3 Button component + §2.4 Product Card component + §2.5 Label component.
2. §2.6 dynamic-content annotation on the top 20 layers.
3. **Outcome:** AI can auto-generate Product Card JSON from Figma props (reaching ~60% automation that Walkthrough Step 5 promised).

### Phase D — Missing-surface design (ongoing, scoped)
1. §3.6 Cart Page + modal types as they come up in client work.
2. §3.7 Customer Account screens — lowest priority unless a client asks.
3. §3.8 Error / Standard pages.

### Phase E — Automation (parallel, depends on A) 
1. §4.1 Token export script.
2. §4.2 Naming lint.
3. §3.1 Design system library (big commitment, but right model for 60-80 clients).

---

## 4. Quantified impact estimates (per walkthrough's automation spectrum)

| Alignment item | Current automation | After Tier 1 | After Tier 2 | After Tier 3 |
|---|---|---|---|---|
| Typography CSS generation | 95% | 100% | 100% | 100% |
| Token → CSS variable export | 40% (hand-edit) | 90% | 95% | **100% (auto)** |
| Block selection from Figma props | 20% | 20% | **80%** | 90% |
| Section type mapping (Figma frame → section.liquid) | 40% | 60% | **95%** | 100% |
| CTA state extraction | 0% | 0% | **70%** | 90% |
| Label variant extraction | 0% | 0% | **70%** | 90% |
| Button structural CSS (padding, dimensions) | 70% (walkthrough) | 70% | **95%** | 100% |
| Button colors + hover states | 0% (walkthrough) | 30% | 70% | **95% (via library)** |
| Cart, modal, customer pages | 10% | 10% | 10% | **70%** |
| Responsive (desktop+mobile from one spec) | 40% (dual lookup) | 40% | **85% (variants)** | 90% |
| Dynamic-data placeholder recognition | 0% | 0% | **60%** | 85% |

**Overall AI-generation confidence for a new client theme build: 40% today → ~75% after Tier 1+2 → ~90% after Tier 3.**

---

## 5. Out of scope (explicit non-recommendations)

These came up during the analysis but are **NOT recommended**:

1. **Don't move pure behavior settings into Figma.** auto_rotate timing, scroll snap, pause_on_hover, hide_if_empty, x-defer modes — these remain in code templates. Attempting to capture them in Figma adds noise without payoff.
2. **Don't design every product metafield combination.** Figma shows one canonical state per card; let the code template handle discount/low-stock/preorder/subscription overlays via CTA state variants (§2.4), not individual designed screens.
3. **Don't enforce pixel-perfect color match between Figma and prod.** Walkthrough noted every color diverged slightly. The design system library (§3.1) is the right tool — flag intentional deviations, don't hand-sync.
4. **Don't design "all" icons.** 70+ icons is a lot; an icon sheet showing the 15-20 most-used is enough. Build a codebase pattern for "add new icon" instead.
5. **Don't design the email templates, admin screens, checkout.** Shopify owns those surfaces; theme control is minimal.

---

## 6. Open questions for Felix

1. **Is the secondary font slot (`--typography-family-secondary`) actually used in production, or a dead code path?** If dead, drop from code instead of adding to Figma.
2. **Does the tablet breakpoint (1200-768px) belong in the system?** Figma has it; code doesn't. Pick one.
3. **Which of the 8 button classes are distinct design-wise vs just CSS-class-renamed variants of 4?** Confirm the 48-variant matrix is right-sized.
4. **Is the intent to use one library for all 60-80 clients with overrides, or a per-client duplicate that diverges over time?** Drives whether Tier 3.1 (design system library) is phase D or phase A.
5. **Owner of the Figma file?** Tier 1 fixes are irreversible within a Figma file; confirm who OKs the rename pass.
