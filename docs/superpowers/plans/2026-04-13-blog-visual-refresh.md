# Blog Visual Refresh Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add visual character to the Outsharked blog via a dark header, amber left-border post accents, coloured tag/category links, and cleaner section labels — across both light and dark themes.

**Architecture:** Pure CSS/Tailwind changes only. No layout restructuring, no new components. The existing CSS variable system (`--color-text-active`, etc.) already provides the right accent colours for both themes; we just wire them to the new visual elements.

**Tech Stack:** Astro, Tailwind CSS, CSS custom properties. Dev server: `pnpm dev` (runs on `http://localhost:4321` by default).

**Spec:** `docs/superpowers/specs/2026-04-13-blog-visual-refresh-design.md`

---

## File Map

| File | Change |
|---|---|
| `src/styles/index.css` | Add `--color-tag` variable; add `.section-label` component class; update `.aside-widget` class |
| `src/components/Header.astro` | Replace `bg-skin-fill` with fixed dark background `#1e1e1e` |
| `src/components/PostViewTitle.astro` | Wrap post block in amber left-border; colour category links amber, tag links with `--color-tag` |
| `src/pages/index.astro` | Replace hardcoded grey `<h2>` banner with `.section-label` div |
| `src/layouts/BlogPost.astro` | Add `text-skin-active` to the back button |

---

## Task 1: CSS foundations — variables, `.section-label`, `.aside-widget`

**Files:**
- Modify: `src/styles/index.css`

- [ ] **Step 1: Add `--color-tag` CSS variable to both themes**

In `src/styles/index.css`, find the `:root, html[data-theme="light"]` block and add the tag colour. Then add it to the dark theme block too.

```css
/* In :root, html[data-theme="light"] block — add after --color-border-active line */
--color-tag: 106, 154, 106;

/* In html[data-theme="dark"] block — add after --color-border-active line */
--color-tag: 142, 192, 124;
```

- [ ] **Step 2: Add `.section-label` component class**

In `src/styles/index.css`, inside the existing `@layer components { }` block, add after the last existing class:

```css
.section-label {
  @apply text-xs font-bold tracking-widest uppercase pb-1.5;
  color: rgba(var(--color-text-dodge), 1);
  border-bottom: 1px solid rgba(var(--color-text), .15);
  margin-bottom: 0.75rem;
}
```

- [ ] **Step 3: Update `.aside-widget` class to use uppercase tracking**

Find the existing `.aside-widget` rule in `src/styles/index.css`:

```css
/* Find this: */
.aside-widget {
  @apply flex items-center w-full mb-2;
  border-bottom:  1px solid rgba(var(--color-text), .3);
}

/* Replace with: */
.aside-widget {
  @apply flex items-center w-full mb-2 text-xs font-bold tracking-widest uppercase;
  color: rgba(var(--color-text-dodge), 1);
  border-bottom: 1px solid rgba(var(--color-text), .2);
}
```

- [ ] **Step 4: Add `text-skin-tag` to Tailwind config**

In `tailwind.config.js`, add `tag` to `textColor.skin` and `active` to `borderColor.skin`:

```js
textColor: {
  skin: {
    base: withOpacity('--color-text'),
    dodge: withOpacity('--color-text-dodge'),
    active: withOpacity('--color-text-active'),
    tag: withOpacity('--color-tag'),   // ← add this line
  },
},

// Also update borderColor.skin:
borderColor: {
  skin: {
    normal: withOpacity('--color-text'),
    base: withOpacity('--color-border'),
    dodge: withOpacity('--color-text-dodge'),
    active: withOpacity('--color-text-active'),   // ← add this line (needed for border-skin-active in PostViewTitle)
  },
},
```

- [ ] **Step 5: Start dev server and verify CSS loads without errors**

```bash
pnpm dev
```

Open `http://localhost:4321`. No visual difference yet — just confirming no build errors in the terminal. Stop the server (`Ctrl+C`) when done.

- [ ] **Step 6: Commit**

```bash
git add src/styles/index.css tailwind.config.js
git commit -m "style: add --color-tag variable, .section-label class, update .aside-widget"
```

---

## Task 2: Dark header

**Files:**
- Modify: `src/components/Header.astro`

- [ ] **Step 1: Replace the header background colour**

In `src/components/Header.astro`, find the opening `<header>` tag on line 19:

```astro
<!-- Find: -->
<header class="z-10 w-full bg-skin-fill text-skin-base">

<!-- Replace with: -->
<header class="z-10 w-full text-skin-base" style="background-color: #1e1e1e;">
```

- [ ] **Step 2: Make nav link text light so it's readable on dark background**

In the same file, find the desktop nav link container (the `<div class="flex items-center pr-4 space-x-5">` inside the hidden xl:block div). The `HeaderLink` components use `text-skin-base` which will be warm dark in light mode — override the container:

```astro
<!-- Find: -->
<div class="flex items-center pr-4 space-x-5">

<!-- Replace with: -->
<div class="flex items-center pr-4 space-x-5" style="color: #ccc;">
```

- [ ] **Step 3: Verify in browser**

```bash
pnpm dev
```

Open `http://localhost:4321`. The header should be near-black (`#1e1e1e`) in both light and dark mode. Nav links should be visible. Toggle the theme — header stays dark, nav text remains readable.

- [ ] **Step 4: Commit**

```bash
git add src/components/Header.astro
git commit -m "style: fixed dark header background across all pages and themes"
```

---

## Task 3: Amber left-border accent on post entries + coloured links

**Files:**
- Modify: `src/components/PostViewTitle.astro`

- [ ] **Step 1: Wrap the post block in a left-border accent div**

In `src/components/PostViewTitle.astro`, the outermost element is `<div class="text-skin-base">`. Wrap its contents in a bordered div:

```astro
<!-- Find: -->
<div class="text-skin-base">
  <!-- title -->
  <div class="flex items-center">

<!-- Replace with: -->
<div class="text-skin-base border-l-[3px] border-skin-active pl-3">
  <!-- title -->
  <div class="flex items-center">
```

That is: add `border-l-[3px] border-skin-active pl-3` to the outer `<div class="text-skin-base">`.

- [ ] **Step 2: Colour category links amber**

In the same file, find the category `<a>` tag:

```astro
<!-- Find: -->
<a href={getUrl("/category/") + categoryName} class="text-wrap break-all hover:text-skin-active mr-2">

<!-- Replace with: -->
<a href={getUrl("/category/") + categoryName} class="text-wrap break-all text-skin-active hover:underline mr-2">
```

- [ ] **Step 3: Colour tag links with the tag colour**

Find the tag `<a>` tag:

```astro
<!-- Find: -->
<a href={getUrl("/tags/") + tagName} class="text-wrap break-all hover:text-skin-active mr-2">

<!-- Replace with: -->
<a href={getUrl("/tags/") + tagName} class="text-wrap break-all text-skin-tag hover:underline mr-2">
```

- [ ] **Step 4: Verify in browser**

```bash
pnpm dev
```

Open `http://localhost:4321`. Each post in the list should have an amber vertical bar on its left edge. Category links should be amber, tag links green. Toggle to dark mode — the border and link colours should shift to gold (`#fabd2f`) and lighter green (`#8ec07c`).

- [ ] **Step 5: Commit**

```bash
git add src/components/PostViewTitle.astro
git commit -m "style: amber left-border on post entries; colour category and tag links"
```

---

## Task 4: Replace grey section banner with clean label

**Files:**
- Modify: `src/pages/index.astro`

- [ ] **Step 1: Replace the hardcoded grey `<h2>` banner**

In `src/pages/index.astro`, find the "Recent Notes Feed" section:

```astro
<!-- Find: -->
<h2 class="text-lg font-semibold mb-2 px-3 py-2" style="background-color: #6b7280; color: white;">Recent Notes Feed</h2>

<!-- Replace with: -->
<div class="section-label">Recent Notes</div>
```

- [ ] **Step 2: Verify in browser**

```bash
pnpm dev
```

Open `http://localhost:4321`. The grey banner above the feed items should be gone, replaced by a small uppercase "RECENT NOTES" label with a faint bottom border. Check dark mode too — the label should be subdued but visible.

Also check `http://localhost:4321/blog/1` — the sidebar "Categories", "Tags", "Recent Articles" headings should now be uppercase/tracked (from the `.aside-widget` update in Task 1).

- [ ] **Step 3: Commit**

```bash
git add src/pages/index.astro
git commit -m "style: replace grey section banner with clean uppercase section label"
```

---

## Task 5: Amber back button on blog post pages

**Files:**
- Modify: `src/layouts/BlogPost.astro`

- [ ] **Step 1: Add `text-skin-active` to the back button**

In `src/layouts/BlogPost.astro`, find the back button (around line 57):

```astro
<!-- Find: -->
<button
  class="flex items-center outline-none cursor-pointer text-md hover:text-skin-active"
  style="background-color: inherit;"
  onclick="history.back()"
>

<!-- Replace with: -->
<button
  class="flex items-center outline-none cursor-pointer text-md text-skin-active hover:underline"
  style="background-color: inherit;"
  onclick="history.back()"
>
```

- [ ] **Step 2: Verify in browser**

```bash
pnpm dev
```

Navigate to a blog post (e.g. `http://localhost:4321/blog/hello-world`). The "← Back" button should be amber in light mode and gold in dark mode. The post title should have the same amber left-border accent as the list view (inherited from `PostViewTitle.astro` which is reused here).

- [ ] **Step 3: Final pass — check all pages**

With the dev server running, check each affected page type:
- `http://localhost:4321/` — homepage: dark header, amber post borders, section label
- `http://localhost:4321/blog/1` — blog list: dark header, amber post borders, styled sidebar headings
- `http://localhost:4321/blog/hello-world` — post detail: dark header, amber post title border, amber back button
- Toggle dark mode on each — accent shifts to gold, background darkens correctly

- [ ] **Step 4: Commit**

```bash
git add src/layouts/BlogPost.astro
git commit -m "style: amber back button on blog post pages"
```
