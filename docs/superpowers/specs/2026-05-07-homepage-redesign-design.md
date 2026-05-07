# Home Page Redesign — Design Spec

**Date:** 2026-05-07  
**Status:** Approved

## Goal

Replace the current home page (a paginated blog post list + recent feed items) with a reading experience that renders the most recent blog post in full, with prev/next navigation and a sidebar that includes a post index and recent feed notes.

## Constraints

- Keep the current visual theme (no theme change)
- Reuse existing components and patterns wherever possible
- The existing `/blog/1` all-posts list page is unchanged

## Home Page (`/`)

The home page renders identically to an individual blog post page, with two differences:
1. No "go back" button (there is no meaningful "back" destination from the home page)
2. Extended sidebar (see below)

### Main column

- `PostTitle` component (title, date, reading time, tags — same as blog post pages)
- Horizontal divider
- `<article class="markdown-body"><Content /></article>` — full rendered markdown
- Horizontal divider
- Prev/next post navigation — same markup as `[...slug].astro`. Since the home page always shows the latest post, there is no "next" post; "prev" is the second-most-recent post.
- `BlogFooter`

### Sidebar column

From top to bottom:
1. `Profile` (existing component, unchanged)
2. `Toc` (existing component, conditional on `headings.length > 0`)
3. `PostIndex` (new component — see below)
4. `FeedSidebar` (new component — see below)

## New Layout: `HomePage.astro`

Based on `BlogPost.astro` with the following changes:
- Remove the back button (`history.back()` link)
- Accept `headings` prop for ToC (same as BlogPost)
- Sidebar renders `Profile`, `Toc`, `PostIndex`, `FeedSidebar` instead of `Profile` + optional `Toc` only

## New Component: `PostIndex.astro`

Sidebar widget listing all blog posts as links.

- Fetches all blog posts via `getCollectionByName('blog')`
- Sorts by date descending (`sortPostsByDate`)
- Renders each as a truncated link to `/blog/[id]`
- Styled with `aside-widget` heading class and same truncated link style as existing "Recent Articles" in `BlogAside`
- "All posts →" link to `/blog/1` at the bottom
- Shows all posts (blog is currently small; a cap can be introduced later if needed)

## New Component: `FeedSidebar.astro`

Sidebar widget listing recent feed notes.

- Fetches feed items via `getCollectionByName('feed')`
- Sorts by date descending
- Shows most recent `site.recentBlogSize` items (currently 5)
- Renders each as a truncated link to `/feed/[id]`
- Styled identically to `PostIndex`
- "All notes →" link to `/feed/1` at the bottom

## Modified: `src/pages/index.astro`

Rewritten to:
1. Fetch all blog posts and sort by date
2. Take index 0 as the current (latest) post; index 1 as `prevPost`; `nextPost` is always undefined
3. Call `render(entry)` from `astro:content` to get `Content`, `headings`, `remarkPluginFrontmatter`
4. Render using `HomePage` layout, passing `frontmatter`, `headings`
5. Render `PostTitle`, full `Content`, prev/next nav, `BlogFooter` in the main slot

## Unchanged

| File | Reason |
|------|--------|
| `src/pages/blog/[page].astro` | Already serves as the "All posts" list |
| `src/layouts/BlogPost.astro` | Individual post pages unaffected |
| `src/components/BlogAside.astro` | Still used in `IndexPage` layout for other pages |
| `src/consts.ts` | "Blog" nav link already points to `/blog/1` |
| All feed pages | Unaffected |

## File Summary

**New:**
- `src/layouts/HomePage.astro`
- `src/components/PostIndex.astro`
- `src/components/FeedSidebar.astro`

**Modified:**
- `src/pages/index.astro`
