# Feed Headline Field — Design Spec

**Date:** 2026-05-07  
**Status:** Approved

## Goal

Add an optional `headline` field to feed/notes entries so the `FeedSidebar` can display a short descriptive label instead of the formatted date. Limit the sidebar to notes from the last 30 days.

## Changes

### `src/content.config.ts`

Add `headline` to the feed schema:

```ts
headline: z.string().optional().nullable(),
```

Optional and nullable so existing entries without the field continue to work unchanged.

### `src/components/FeedSidebar.astro`

Two changes:

1. **Filter by date:** Replace the `site.recentBlogSize` slice with a 30-day filter. Keep only items where `data.date >= today minus 30 days`.

2. **Display text:** Use `post.data.headline ?? formatDate(post.data.date) ?? post.id` — headline wins if present, formatted date is the fallback, id is last resort.

## Unchanged

- All existing feed `.md` files — no headlines required; they fall back to date display
- `PostIndex.astro`, `HomePage.astro`, `index.astro` — unaffected
- The "All notes →" link and overall sidebar structure — unchanged
