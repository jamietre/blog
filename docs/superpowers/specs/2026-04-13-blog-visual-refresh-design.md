# Blog Visual Refresh — Design Spec

**Date:** 2026-04-13  
**Status:** Approved  

## Overview

A visual refresh of the Outsharked blog (Astro + Tailwind CSS) to add more character and hierarchy without structural changes. Direction: "Refined Editorial" — dark header, amber left-border accents on post entries, coloured category/tag links, and cleaner section labels. Works across both light and dark themes.

All changes are CSS/Tailwind only. No layout restructuring, no content moves, no new components.

---

## 1. Header (`src/components/Header.astro`)

**Change:** Replace the current `bg-skin-fill` background (light grey in light mode, dark grey in dark mode) with a fixed dark background on all pages and in both themes.

- Background: `#1e1e1e`
- Logo/nav text: white (`#fff`) and muted (`#ccc`)
- The header already has `z-10 w-full` — keep as-is, just swap background colour

This gives the page a strong visual anchor at the top regardless of theme.

---

## 2. Post Entries (`src/components/PostViewTitle.astro`)

**Change:** Wrap the post title+metadata block in an amber left-border accent.

- Light mode border colour: `#b5761a` (existing `--color-text-active` light)
- Dark mode border colour: `#fabd2f` (existing `--color-text-active` dark)
- Use `border-l-[3px] border-skin-active pl-3` via Tailwind

**Additionally**, colour the category and tag links to use the theme's active colours:

- Category links: amber (`text-skin-active`) in both modes
- Tag links: green (`#6a9a6a` light / `#8ec07c` dark) — add a new CSS variable `--color-tag` for this

---

## 3. Section Labels (`src/pages/index.astro`, `src/styles/index.css`)

**Change:** Replace the hardcoded grey banner `<h2>` for "Recent Notes Feed" in `index.astro` with an uppercase small-caps label. Also update the existing `.aside-widget` class (used by `BlogAside.astro` sidebar headings) to use the same uppercase style.

New style (add as a Tailwind component class `.section-label`):
```css
.section-label {
  @apply text-xs font-bold tracking-widest uppercase text-skin-dodge pb-1.5;
  border-bottom: 1px solid rgba(var(--color-text), .15);
}
```

Update `.aside-widget` to add uppercase tracking:
```css
.aside-widget {
  @apply flex items-center w-full mb-2 text-xs font-bold tracking-widest uppercase text-skin-dodge;
  border-bottom: 1px solid rgba(var(--color-text), .2);
}
```

The existing grey banner (`bg-[#6b7280]`) in `index.astro` is removed and replaced with a `<div class="section-label">` element.

---

## 4. Blog Post Title Block (`src/components/PostView.astro` via `PostViewTitle.astro`, and `src/layouts/BlogPost.astro`)

**Change:** The same amber left-border treatment applied to post list entries (section 2) naturally applies to the post detail page too, since `PostViewTitle.astro` is reused there. No additional changes needed for the post page title.

The `← Back` button link should use `text-skin-active` so it picks up the amber/gold accent colour.

---

## 5. Colour Variables (no change needed)

The existing CSS variables already provide everything needed:

| Variable | Light | Dark |
|---|---|---|
| `--color-text-active` | `181, 118, 20` (#b5761a amber) | `250, 189, 47` (#fabd2f gold) |
| `--color-text` | `80, 73, 69` (warm dark) | `249, 244, 227` (warm cream) |
| `--color-fill` | `241, 241, 241` | `40, 40, 40` |

One addition: a `--color-tag` variable for tag link colour:
- Light: `106, 154, 106` (#6a9a6a muted green)
- Dark: `142, 192, 124` (#8ec07c lighter green)

---

## Files Changed

| File | Change |
|---|---|
| `src/components/Header.astro` | Fixed dark background on header |
| `src/components/PostViewTitle.astro` | Amber left-border on post block; colour category + tag links |
| `src/pages/index.astro` | Replace grey banner with `.section-label` |
| `src/styles/index.css` (`.aside-widget`) | Add uppercase/tracking to sidebar section headings |
| `src/layouts/BlogPost.astro` | Style back button with `text-skin-active` |
| `src/styles/index.css` | Add `.section-label` component class; add `--color-tag` variable |

---

## Out of Scope

- No changes to Markdown/article body styles
- No changes to pagination, search, archive, friends, or message pages
- No font changes (stays system sans-serif)
- No layout restructuring
