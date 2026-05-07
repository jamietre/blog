# Sidebar List Arrow Prefix — Design Spec

**Date:** 2026-05-07  
**Status:** Approved

## Goal

Prepend `› ` to each item link in the `PostIndex` and `FeedSidebar` sidebar components.

## Change

In `src/components/PostIndex.astro`: change link text from `{post.data.title}` to `› {post.data.title}`.

In `src/components/FeedSidebar.astro`: change link text from `{label}` to `› {label}`.

No other changes.
