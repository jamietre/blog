# Prev/Next Navigation Above Title — Design Spec

**Date:** 2026-05-07  
**Status:** Approved

## Goal

Move the prev/next post navigation block from below the article body to above the post title, on both the home page and individual blog post pages.

## Changes

### `src/pages/index.astro` and `src/pages/blog/[...slug].astro`

Same structural change in both files:

Move the `<div class="h-8 text-skin-base">` nav row from its current position (after the second `divider-horizontal`, before `BlogFooter`) to the top of the content `<div>`, before `<PostTitle>`.

**Before:**
```
PostTitle
─── divider ───
article body
─── divider ───
nav row (prev / next)
BlogFooter
```

**After:**
```
nav row (prev / next)
PostTitle
─── divider ───
article body
─── divider ───
BlogFooter
```

The second `divider-horizontal` stays in place — it now separates the article from `BlogFooter` instead of from the nav. The nav markup itself is unchanged.

## Unchanged

- `src/layouts/HomePage.astro` — no changes
- `src/layouts/BlogPost.astro` — no changes
- All other pages and components
