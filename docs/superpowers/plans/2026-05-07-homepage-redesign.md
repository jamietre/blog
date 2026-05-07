# Homepage Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the home page blog-list with a full-content render of the most recent blog post, add a sidebar post index and feed notes widget, and create a new `HomePage` layout.

**Architecture:** Four changes: two new sidebar components (`PostIndex`, `FeedSidebar`), one new layout (`HomePage.astro` based on `BlogPost.astro` without the back button and with the extended sidebar), and a rewrite of `index.astro` to fetch + render the latest post. The existing `/blog/1` all-posts page is untouched.

**Tech Stack:** Astro 5, Tailwind CSS, `astro:content` render API, TypeScript, pnpm

---

## File Map

| Action | Path | Responsibility |
|--------|------|----------------|
| Create | `src/components/PostIndex.astro` | Sidebar widget: all blog posts as links + "All posts →" |
| Create | `src/components/FeedSidebar.astro` | Sidebar widget: recent feed notes + "All notes →" |
| Create | `src/layouts/HomePage.astro` | Layout for home page (BlogPost without back button, extended sidebar) |
| Modify | `src/pages/index.astro` | Fetch latest post, render full content via HomePage layout |

---

## Task 1: Create `PostIndex.astro`

**Files:**
- Create: `src/components/PostIndex.astro`

- [ ] **Step 1: Create the component**

```astro
---
import {getCollectionByName} from "@/utils/getCollectionByName";
import {sortPostsByDate} from "@/utils/sortPostsByDate";
import getUrl from "@/utils/getUrl";

const blogs = await getCollectionByName('blog');
const sortedPosts = sortPostsByDate(blogs);
---

<div>
  <div class="aside-widget">
    <i class="ri-file-list-line menu-icon"></i> Posts
  </div>
  <div class="flex flex-col">
    {
      sortedPosts.map((post) => (
        <a
          href={getUrl("/blog/") + post.id}
          class="truncate mt-1 hover:text-skin-active"
          title={post.data.title}
        >
          {post.data.title}
        </a>
      ))
    }
  </div>
  <a href={getUrl("/blog/1")} class="text-xs mt-2 hover:text-skin-active block">All posts →</a>
</div>
```

- [ ] **Step 2: Verify it builds**

```bash
cd /home/jamiet/code/blog && pnpm build 2>&1 | tail -20
```

Expected: no errors mentioning `PostIndex.astro`

- [ ] **Step 3: Commit**

```bash
git add src/components/PostIndex.astro
git commit -m "feat: add PostIndex sidebar component"
```

---

## Task 2: Create `FeedSidebar.astro`

**Files:**
- Create: `src/components/FeedSidebar.astro`

Feed items have no `title` field — only `data.date` and `id` (the filename, e.g. `2026-05-06`). The link text is the formatted date.

- [ ] **Step 1: Create the component**

```astro
---
import {getCollectionByName} from "@/utils/getCollectionByName";
import {formatDate} from "@/utils/formatDate";
import {site} from "@/consts";
import getUrl from "@/utils/getUrl";

const feedItems = await getCollectionByName('feed');
feedItems.sort((a, b) => {
  const aDate = a.data.date ? new Date(a.data.date).valueOf() : 0;
  const bDate = b.data.date ? new Date(b.data.date).valueOf() : 0;
  return bDate - aDate;
});
const recentFeed = feedItems.slice(0, site.recentBlogSize);
---

<div>
  <div class="aside-widget">
    <i class="ri-lightbulb-flash-line menu-icon"></i> Recent Notes
  </div>
  <div class="flex flex-col">
    {
      recentFeed.map((post) => (
        <a
          href={getUrl("/feed/") + post.id}
          class="truncate mt-1 hover:text-skin-active"
          title={post.data.date ? formatDate(post.data.date) : post.id}
        >
          {post.data.date ? formatDate(post.data.date) : post.id}
        </a>
      ))
    }
  </div>
  <a href={getUrl("/feed/1")} class="text-xs mt-2 hover:text-skin-active block">All notes →</a>
</div>
```

- [ ] **Step 2: Verify it builds**

```bash
cd /home/jamiet/code/blog && pnpm build 2>&1 | tail -20
```

Expected: no errors mentioning `FeedSidebar.astro`

- [ ] **Step 3: Commit**

```bash
git add src/components/FeedSidebar.astro
git commit -m "feat: add FeedSidebar sidebar component"
```

---

## Task 3: Create `HomePage.astro`

**Files:**
- Create: `src/layouts/HomePage.astro`

Based on `src/layouts/BlogPost.astro`. Differences: no back button, sidebar includes `PostIndex` and `FeedSidebar` below `Profile`/`Toc`.

- [ ] **Step 1: Create the layout**

```astro
---
import type { MarkdownHeading } from 'astro';
import BaseHead from '@/components/BaseHead.astro';
import Header from '@/components/Header.astro';
import Footer from '@/components/Footer.astro';
import Profile from '@/components/Profile.astro';
import Comment from '@/components/Comment.astro';
import Toc from '@/components/Toc.astro';
import PostIndex from '@/components/PostIndex.astro';
import FeedSidebar from '@/components/FeedSidebar.astro';
import ScrollToTop from '@/components/ScrollToTop.astro';
import { comment, config } from '@/consts';

interface Props {
  frontmatter: {
    title: string;
    description?: string;
    comment?: boolean;
    toc?: boolean;
    mathjax?: boolean;
    mermaid?: boolean;
    ogImage?: string;
  };
  headings: MarkdownHeading[];
}

const {
  frontmatter = {
    title: '',
    comment: false,
    toc: false,
    mathjax: false,
    mermaid: false,
  },
  headings,
} = Astro.props;
---

<html lang={config.lang}>
  <BaseHead
    title={frontmatter.title}
    description={frontmatter.description}
    mathjax={frontmatter.mathjax}
    mermaid={frontmatter.mermaid}
    ogImage={frontmatter.ogImage}
  />

  <body class="bg-skin-secondary flex flex-col min-h-screen">
    <Header />
    <main class="container relative p-4 pb-32 text-skin-base flex-1" id="app">
      <div class="grid grid-cols-4 gap-8">
        <div class="col-span-4 space-y-4 xl:col-span-3 first:space-y-2">
          <slot />
          {frontmatter.comment && comment.enable && <Comment />}
        </div>
        <div class="relative hidden col-span-1 xl:block space-y-6">
          <Profile />
          {frontmatter.toc && <Toc headings={headings} />}
          <PostIndex />
          <FeedSidebar />
        </div>
        <ScrollToTop />
      </div>
      <Footer />
    </main>
  </body>
</html>

<script>
  import { Fancybox } from '@fancyapps/ui';
  import '@fancyapps/ui/dist/fancybox/fancybox.css';

  Fancybox.bind('[data-fancybox]');

  const markdownBody = document.querySelector('.markdown-body');
  if (markdownBody) {
    let images = markdownBody.querySelectorAll('img');
    const callback = (entries: IntersectionObserverEntry[]) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const image = entry.target as HTMLImageElement;
          const data_src = image.getAttribute('data-src');
          const data_alt = image.getAttribute('data-alt');
          image.setAttribute('data-fancybox', 'gallery');
          if (data_src) image.setAttribute('src', data_src);
          if (data_alt) image.setAttribute('alt', data_alt);
          observer.unobserve(image);
        }
      });
    };
    const observer = new IntersectionObserver(callback);
    images.forEach((image) => observer.observe(image));

    const links = markdownBody.querySelectorAll('a');
    for (let i = 0; i < links.length; i++) {
      const names = links[i].getAttributeNames();
      if (!names.includes('data-footnote-backref') && !names.includes('data-footnote-ref')) {
        links[i].setAttribute('target', '_blank');
        links[i].setAttribute('rel', 'nofollow noreferrer');
      }
    }
  }
</script>
```

- [ ] **Step 2: Verify it builds**

```bash
cd /home/jamiet/code/blog && pnpm build 2>&1 | tail -20
```

Expected: no errors mentioning `HomePage.astro`

- [ ] **Step 3: Commit**

```bash
git add src/layouts/HomePage.astro
git commit -m "feat: add HomePage layout"
```

---

## Task 4: Rewrite `src/pages/index.astro`

**Files:**
- Modify: `src/pages/index.astro`

Fetch the latest blog post, render its full content, and show prev/next navigation. The home page always shows the latest post so there is never a `nextPost`; `prevPost` is the second-most-recent post.

- [ ] **Step 1: Replace `src/pages/index.astro` entirely**

```astro
---
import HomePage from '@/layouts/HomePage.astro'
import PostTitle from "@/components/PostTitle.astro";
import BlogFooter from '@/components/BlogFooter.astro'
import {getCollectionByName} from "@/utils/getCollectionByName";
import {render} from "astro:content";
import {sortPostsByDate} from "@/utils/sortPostsByDate";
import getUrl from "@/utils/getUrl";

const blogs = await getCollectionByName("blog");
const sortedPosts = sortPostsByDate(blogs);

const entry = sortedPosts[0];
const {Content, headings, remarkPluginFrontmatter} = await render(entry);

const lastModified = remarkPluginFrontmatter?.lastModified;
const readingTime = remarkPluginFrontmatter?.readingTime;

const prevPost = sortedPosts[1] ?? null;
---

<HomePage frontmatter={entry.data} headings={headings}>
  <div>
    <PostTitle {...entry.data} lastModified={lastModified} readingTime={readingTime}></PostTitle>
    <div class="divider-horizontal"></div>
    <article class="markdown-body">
      <Content/>
    </article>
    <div class="divider-horizontal"></div>
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
    <BlogFooter
      title={entry.data.title}
      url={getUrl('/') + entry.collection + '/' + entry.id}
      date={entry.data.date}
    ></BlogFooter>
  </div>
</HomePage>
```

- [ ] **Step 2: Build and verify**

```bash
cd /home/jamiet/code/blog && pnpm build 2>&1 | tail -30
```

Expected: build completes with no errors; output mentions pages being generated including `/`

- [ ] **Step 3: Start dev server and check the home page**

```bash
cd /home/jamiet/code/blog && pnpm dev
```

Open http://localhost:4321 and verify:
- The most recent blog post title and content appear
- The sidebar shows Profile, ToC (if the post has headings), post list, and recent feed notes
- "All posts →" and "All notes →" links are present in the sidebar
- Clicking "All posts →" goes to `/blog/1` (the existing blog list)
- `← [previous post title]` navigation link appears at the bottom if there is a previous post

- [ ] **Step 4: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: render latest blog post on home page"
```
