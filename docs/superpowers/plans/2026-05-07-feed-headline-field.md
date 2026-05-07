# Feed Headline Field Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an optional `headline` field to feed entries and update `FeedSidebar` to display it instead of the date, while limiting the sidebar to notes from the last 30 days.

**Architecture:** Two file changes — add `headline` to the feed content schema, then update `FeedSidebar.astro` to use it as display text (falling back to date, then id) and filter items to the last 30 days using dayjs (already a project dependency).

**Tech Stack:** Astro 5, `astro/zod` for schema, dayjs for date math, TypeScript

---

## File Map

| Action | Path | Change |
|--------|------|--------|
| Modify | `src/content.config.ts` | Add `headline` field to feed schema |
| Modify | `src/components/FeedSidebar.astro` | 30-day filter + headline display |

---

## Task 1: Add `headline` to feed schema

**Files:**
- Modify: `src/content.config.ts`

- [ ] **Step 1: Add the `headline` field**

In `src/content.config.ts`, replace the feed schema object:

```typescript
const feed = defineCollection({
  loader: glob({ base: './src/content/feed', pattern: '**/*.{md,mdx}' }),
  schema: z.object({
    date: z.date().or(z.string()).optional().nullable(),
    headline: z.string().optional().nullable(),
    donate: z.boolean().default(false),
    comment: z.boolean().default(false),
  })
})
```

- [ ] **Step 2: Verify the build passes**

```bash
cd /home/jamiet/code/blog && pnpm build 2>&1 | tail -10
```

Expected: `21 page(s) built` with no errors. Existing feed entries without `headline` should still build fine — the field is optional.

- [ ] **Step 3: Commit**

```bash
git add src/content.config.ts
git commit -m "feat: add headline field to feed schema"
```

---

## Task 2: Update `FeedSidebar.astro`

**Files:**
- Modify: `src/components/FeedSidebar.astro`

Replace the entire file content:

- [ ] **Step 1: Rewrite `FeedSidebar.astro`**

```astro
---
import {getCollectionByName} from "@/utils/getCollectionByName";
import {formatDate} from "@/utils/formatDate";
import getUrl from "@/utils/getUrl";
import dayjs from 'dayjs';

const feedItems = await getCollectionByName('feed');
feedItems.sort((a, b) => {
  const aDate = a.data.date ? new Date(a.data.date).valueOf() : 0;
  const bDate = b.data.date ? new Date(b.data.date).valueOf() : 0;
  return bDate - aDate;
});

const thirtyDaysAgo = dayjs().subtract(30, 'day');
const recentFeed = feedItems.filter(item =>
  item.data.date && dayjs(item.data.date).isAfter(thirtyDaysAgo)
);
---

<div>
  <div class="aside-widget">
    <i class="ri-lightbulb-flash-line menu-icon"></i> Recent Notes
  </div>
  <div class="flex flex-col">
    {
      recentFeed.map((post) => {
        const label = post.data.headline ?? (post.data.date ? formatDate(post.data.date) : post.id);
        return (
          <a
            href={getUrl("/feed/") + post.id}
            class="truncate mt-1 hover:text-skin-active"
            title={label}
          >
            {label}
          </a>
        );
      })
    }
  </div>
  <a href={getUrl("/feed/1")} class="text-xs mt-2 hover:text-skin-active block">All notes →</a>
</div>
```

Key changes from the previous version:
- Removed `import {site} from "@/consts"` (no longer using `recentBlogSize`)
- Added `import dayjs from 'dayjs'`
- Replaced `feedItems.slice(0, site.recentBlogSize)` with a 30-day date filter
- Display label is `headline ?? formatDate(date) ?? id`

- [ ] **Step 2: Verify the build passes**

```bash
cd /home/jamiet/code/blog && pnpm build 2>&1 | tail -10
```

Expected: `21 page(s) built` with no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/FeedSidebar.astro
git commit -m "feat: show headline in feed sidebar, limit to last 30 days"
```
