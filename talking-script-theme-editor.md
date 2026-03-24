# Talking Script: Theme Editor Integration

**Session flow:** Tabs Focus wrap-up (5 min) → Theme Editor deep-dive (25-30 min)

---

## PART 1: Tabs Focus Wrap-Up (from Page Transitions)

> Last session we covered Render Tabs — how sections declare tabs with marker tags, how `_layout.render_tabs` processes the HTML at the layout level, and how `initTabs` wires up Alpine state. Today I want to close that topic with the most important piece: **what happens when someone clicks a block in the Theme Editor that's hidden inside a tab.**

### The Problem

> Imagine you've built a product page with three tabs — Description, Reviews, Testimonials. The merchant adds a Testimonial block and clicks it in the Theme Editor sidebar. The block lives inside the Testimonials tab, but the Description tab is active. The merchant clicks the block... and nothing visually happens. They think the block is broken.

> This is the number one "it's broken" support ticket for tabbed content. The editor correctly identifies the block in the DOM — it fires `shopify:block:select` — but the block element has `display: none` because it's behind an `x-show` that's currently false.

### The Solution — Reactive, Not Imperative

> Here's where our approach is different from most Shopify themes. We don't write `document.addEventListener("shopify:block:select", ...)` in our tab components. Instead, we have a centralized `editor.ts` file that handles ALL Shopify editor events and writes the result into an Alpine store called `$editor`.

> When `shopify:block:select` fires, `editor.ts` sets `$editor.select_block_id` to the selected block's ID. That's it — one store update.

> Then in each tab content block's Liquid template, we have an `x-effect` that watches `$editor.select_block_id`. When it matches the block's ID — or when the block *contains* an element with a matching `data-style-id` — the effect sets `current_tabs[index]` to that tab's label. The tab switches, the block becomes visible.

> *[Show the code from `tab_content.liquid` on the flowchart]*

> The same pattern works for accordions. In `collapsible.ts`, there's an Alpine effect that watches `$editor.select_block_id` and `$editor.select_section_id`. It constructs a composite `data-style-id` string and checks if the current collapsible — or anything inside it — matches. If it does, it force-opens the accordion and sets a `themeEditorControl` flag.

> That flag is key. On deselect, we only close the accordion if `themeEditorControl` is true — meaning the editor forced it open. If the merchant had already opened it manually, we leave it alone. That prevents a frustrating UX where clicking a different block suddenly collapses things the merchant intentionally opened.

> For carousels and horizontal scroll containers, we use a different technique. The Liquid template injects an `x-effect` only in design mode — `{% if request.design_mode %}` — that calls `utils.scrollToX()` to scroll the carousel to the selected block's position.

> So: tabs switch via store reactivity, accordions open via store reactivity with a control flag, carousels scroll via a design-mode-only x-effect. All driven by the same `$editor.select_block_id` store value. That wraps up tabs focus. Now let's get into the full Theme Editor picture.

---

## PART 2: Theme Editor Integration

### 2.1 — How the Theme Editor Works

> When a merchant clicks "Customize" in the Shopify admin, they're not looking at your theme directly. The admin UI — the sidebar with all the controls — runs in the parent frame. Your theme loads in an iframe. Two separate documents talking through postMessage.

> *[Point to the architecture diagram]*

> The critical thing: the admin can surgically re-render individual sections. Merchant changes a setting → Shopify takes that section's Liquid, renders it on the server with the new value → hot-swaps just that HTML chunk into the iframe. Everything else stays.

> Your JavaScript has to be ready for DOM nodes to appear and disappear at any time. An event listener attached to a button in a section? That button is a new DOM node after re-render. Your old listener is on a removed element. This is why we centralize everything in `editor.ts` and use Alpine's reactive model.

### 2.2 — Three Detection Flags, Not Two

> Most guides tell you there are two flags — `request.design_mode` in Liquid and `Shopify.designMode` in JS. In our codebase, there are effectively three you need to know about.

> **Flag 1: `request.design_mode`** — Liquid server-side. True when inside the Theme Editor. This gates extra HTML output, inline JSON, fallback styles, section group loading.

> **Flag 2: `request.visual_preview_mode`** — Also Liquid server-side, but different. This is true when Shopify renders a visual preview — like the theme thumbnails you see in the admin themes list. It's NOT the same as the editor. We use it alongside `design_mode` to create three rendering paths: full editor, visual preview, and production. You'll see these two flags combined all over `theme.liquid`.

> **Flag 3: `window.design_mode`** — Client-side. But here's the thing — we don't read `Shopify.designMode` directly. Instead, `_layout.global_settings.liquid` sets `window.design_mode` from `request.design_mode` in a script tag. All our TypeScript checks this flag. When it's true, `theme.ts` dispatches a `"theme:editor:init"` custom event, which kicks off the editor system.

### 2.3 — The Centralized Editor System

> All Theme Editor integration lives in one file: `editor.ts`. About 325 lines. It manages an Alpine store called `"editor"` with a magic property `$editor` available everywhere.

> *[Show the EditorStore type on the flowchart]*

> The store has eight fields: `load_section_id`, `unload_section_id`, `select_section_id`, `reorder_section_id`, `select_block_id`, `deselect_block_id`, `deselect_section_id`, and `inspector`. Every Shopify event updates the relevant field. Components react through Alpine effects.

> There's also dual state tracking — every store update writes to both `Alpine.store("editor")` and `window.Shopify.editor`. The Alpine store is for reactive components, the window object is for non-Alpine code.

> The initialization chain is: Alpine loads → `alpine:init` fires → `initTheme()` runs → checks `window.design_mode` → if true, dispatches `"theme:editor:init"` → `editor.ts` listens for that event and registers all 9 Shopify event listeners. In production, none of this runs.

### 2.4 — The Nine Editor Events

> Shopify dispatches nine events, not six like most docs mention.

> **Five section events:**

> `shopify:section:load` — the most complex handler. When a section is added or re-rendered, this fires. Our handler does a LOT: it parses tab marker elements from the new HTML, rebuilds Alpine tab button bindings, moves `<style>` tags into the section root to prevent CSS conflicts, replaces stale section IDs in old CSS selectors, sets the `x-data` attribute for Alpine init, and dispatches a custom `editor-load--{sectionId}` event.

> `shopify:section:unload` — section removed. Updates the store, dispatches `editor-unload--{sectionId}`.

> `shopify:section:select` and `deselect` — merchant clicks or clicks away from a section in the sidebar. No re-render, just store updates and custom event dispatch.

> `shopify:section:reorder` — fires when sections are dragged to reorder. Most guides miss this one.

> **Two block events:**

> `shopify:block:select` — sets `$editor.select_block_id`. This is the trigger for all the tab-switching, accordion-opening, carousel-scrolling behavior we covered in Part 1.

> `shopify:block:deselect` — clears the select state, sets deselect ID.

> **Two inspector events:**

> `shopify:inspector:activate` and `deactivate` — the inspector is that crosshair tool that lets merchants click elements in the preview to select them. We track its state in `$editor.inspector`.

> There's also a `window.message` listener for `StorefrontMessage::SelectElement`. When the merchant clicks an element via the inspector, this postMessage comes through with the block and section GIDs, and we update the store accordingly.

> **All nine events are re-dispatched as custom events.** `shopify:section:load` for section "template--123" becomes `editor-load--template--123` AND `editor_load`. This lets individual sections listen for their own specific events without filtering by ID. For example, the code editor preview component listens for `editor-load--{{ section.id }}` to reinitialize Ace.

### 2.5 — Schema Design in TypeScript

> In our codebase, schemas aren't written as JSON inside `{% schema %}` blocks. They're TypeScript files — `_schema.ts` and `_presets.ts` — that get compiled at build time.

> *[Show the side-by-side on the flowchart]*

> The `_schema.ts` exports a typed object using `ShopifyThemeBlock<T>` or `ShopifySection<T>`. Settings array, block name, category. The `_presets.ts` defines how the block/section appears in the editor's "Add" menu — default settings, nested block structure, category.

> Two things worth highlighting:

> **`visible_if`** — settings can be conditionally shown in the sidebar. A `visible_if` field takes a Liquid-like expression. If the expression is false, the setting is hidden. This declutters the UI so merchants only see settings relevant to their current choices. For example, a URL field only shows when the button type is "button" or "link", not when it's a metafield.

> **`universalBlockGlobals()`** — a shared utility that spreads common settings (text_color, background_color, custom_css) into every block. DRY across 20+ block types.

> And remember: every block MUST output `{{ block.shopify_attributes }}` on its wrapper element. Without it, the block is invisible to the editor — can't be highlighted, selected, or found by the inspector.

### 2.6 — Design-Mode Branching in Liquid

> Five patterns to know:

> **Pattern 1: Section group loading.** The most complex. `theme.liquid` has different rendering paths for head sections, global sections, card sections, header sections, and modal sections — each branching on `design_mode`, `visual_preview_mode`, and development toggle settings.

> **Pattern 2: Extra data.** Design mode embeds full product JSON inline. Production uses the fetch cascade. This ensures product cards render immediately in the editor without waiting for network requests.

> **Pattern 3: Editor-only Alpine effects.** The `x-effect` attributes for scroll-to-block are only rendered when `request.design_mode` is true. Completely absent from production HTML.

> **Pattern 4: Fallback styles.** When metaobject data isn't loaded yet, design mode provides inline `<style>` tags from block settings directly so blocks aren't unstyled during setup.

> **Pattern 5: Development toggles.** A set of `development__*` settings that block heavy section groups in the editor for faster iteration. `development__block_card_sections`, `development__block_modal_sections`, `development__disable_animations`, etc. These are developer ergonomics — not merchant-facing.

### 2.7 — Empty Block Handling

> In production, empty blocks output `<script type="block/hide"></script>` which triggers client-side hiding. In design mode, these hide scripts are NOT output — merchants need to see empty blocks to configure them.

> On top of that, `editor.ts` scans for `data-hide-if-empty-block`, `data-hide-if-empty-container`, and `data-hide-if-empty-section` attributes. When the content inside is empty, it adds a cyan outline and a label like "Empty Block Hidden" so merchants understand why something looks blank.

### 2.8 — CSS Scoping and Section:Load

> Every block uses `data-style-id="{{ section.id }}--{{ block.id }}"` for CSS scoping. Styles are written with `&` as a placeholder, and Liquid replaces `&` with the `[data-style-id="..."]` selector.

> When the editor re-renders a section, the `section:load` handler in `editor.ts` does CSS consolidation: moves all `<style>` tags into the section root, and replaces old section IDs in the CSS with a dead string so they don't conflict with the new section's styles.

> It also rebuilds the entire tab system — parsing `<tab-content-group>` marker elements, generating Alpine-bound buttons, storing the tab structure in `data-tabs`, and setting `x-data` for Alpine initialization.

### 2.9 — Features Disabled in the Editor

> Not everything should run. Page transitions (Barba.js) — always disabled. IndexedDB cache — disabled. Animations — optionally disabled via development toggle. Marker.io — skipped. Modals get a special path: design mode loads them as full section groups for editing, production loads them dynamically.

> The pattern is always: check `window.design_mode` at init. Either return early or switch to an editor-friendly mode.

### 2.10 — Testing

> *[Point to the testing checklist]*

> Add it, change every setting, click each block, test empty states, reorder, remove and re-add, test the inspector, switch templates. Eight checks.

> The golden rule: **build as if your section will be added, configured, removed, re-added, reordered, and clicked in rapid succession — because in the editor, it will be.**

---

## Wrap-Up

> To summarize: the Theme Editor is an iframe that re-renders sections on the fly. We detect it with three flags — `request.design_mode`, `request.visual_preview_mode`, and `window.design_mode`. We handle all nine events centrally in `editor.ts`, which updates a reactive `$editor` Alpine store. Components react to store changes via `x-effect` — tabs switch, accordions open, carousels scroll. Schemas are TypeScript with type safety and `visible_if` conditions. Design mode branching handles five patterns including development toggles. Empty blocks get visual feedback. CSS is scoped via `data-style-id`. And you test everything in the actual editor.

> The flowchart covers all of this with code examples pointing to the actual files in our codebase. Questions?
