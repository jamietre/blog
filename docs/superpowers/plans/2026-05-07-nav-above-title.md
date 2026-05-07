# Nav Above Title Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Move the prev/next navigation block from below the article body to above the post title in both the home page and individual blog post pages.

**Architecture:** Pure structural reposition in two page files — the nav markup is unchanged, only its position in the template moves. No new files, no new components.

**Tech Stack:** Astro 5, TypeScript

---

## File Map

| Action | Path | Change |
|--------|------|--------|
| Modify | `src/pages/index.astro` | Move nav block above PostTitle |
| Modify | `src/pages/blog/[...slug].astro` | Move nav block above PostTitle |

---

## Task 1: Move nav above title in `index.astro`

**Files:**
- Modify: `src/pages/index.astro`

- [ ] **Step 1: Edit the template section**

Replace the entire template section (everything from `<HomePage ...>` to `</HomePage>`) with:

```astro
<HomePage frontmatter={entry.data} headings={headings}>
  <div>
    <div class="h-8 text-skin-base">
      {
        prevPost ? (
          <a
            class="truncate w-auto max-w-[40%] float-left"
            href={getUrl('/') + prevPost.collection + '/' + prevPost.id}
            title={prevPost.data.title}
          >
            <i class="ri-arrow-left-s-fill"/>
            {prevPost.data.title}
          </a>
        ) : (
          <div/>
        )
      }
    </div>
    <PostTitle {...entry.data} lastModified={lastModified} readingTime={readingTime}></PostTitle>
    <div class="divider-horizontal"></div>
    <article class="markdown-body">
      <Content/>
    </article>
    <div class="divider-horizontal"></div>
    <BlogFooter
      title={entry.data.title}
      url={getUrl('/') + entry.collection + '/' + entry.id}
      date={entry.data.date}
    ></BlogFooter>
  </div>
</HomePage>
```

The frontmatter section (lines 1–20) is **unchanged** — only the template below the `---` moves.

- [ ] **Step 2: Verify the build passes**

```bash
cd /home/jamiet/code/blog && pnpm build 2>&1 | tail -10
```

Expected: `21 page(s) built` with no errors.

- [ ] **Step 3: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: move prev nav above post title on home page"
```

---

## Task 2: Move nav above title in `[...slug].astro`

**Files:**
- Modify: `src/pages/blog/[...slug].astro`

- [ ] **Step 1: Edit the template section**

Replace the entire template section (everything from `<BlogPost ...>` to `</BlogPost>`) with:

```astro
<BlogPost frontmatter={entry.data} headings={headings}>
  <div>
    <div class="h-8 text-skin-base">
      {
        prevPost ? (
          <a
            class="truncate  w-auto max-w-[40%] float-left"
            href={getUrl('/') + prevPost.collection + '/' + prevPost.id}
            title={prevPost.data.title}
          >
            <i class="ri-arrow-left-s-fill"/>
            {prevPost.data.title}
          </a>
        ) : (
          <div/>
        )
      }
      {
        nextPost ? (
          <div class="flex item-center float-right w-auto max-w-[40%] text-right">
            <a class="truncate " href={getUrl('/') + nextPost.collection + '/' + nextPost.id} title={nextPost.data.title}>
              {nextPost.data.title}
            </a>
            <i class="float-right ri-arrow-right-s-fill"/>
          </div>
        ) : (
          <div/>
        )
      }
    </div>
    <PostTitle {...entry.data} lastModified={lastModified} readingTime={readingTime}></PostTitle>
    <div class="divider-horizontal"></div>
    <article class="markdown-body">
      <Content/>
    </article>
    <div class="divider-horizontal"></div>
    <BlogFooter title={entry.data.title} url={getUrl('/') + entry.collection + '/' + entry.id} date={entry.data.date}></BlogFooter>
    {
      donate.enable && entry.data.donate &&
      <Donate></Donate>
    }
  </div>

</BlogPost>
```

The frontmatter section (lines 1–38) is **unchanged** — only the template below the `---` moves.

- [ ] **Step 2: Verify the build passes**

```bash
cd /home/jamiet/code/blog && pnpm build 2>&1 | tail -10
```

Expected: `21 page(s) built` with no errors.

- [ ] **Step 3: Commit**

```bash
git add src/pages/blog/[...slug].astro
git commit -m "feat: move prev/next nav above post title on blog post pages"
```
