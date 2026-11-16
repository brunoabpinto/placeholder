---
title: "I Built a Site That Finds Me Good Articles to Read"
slug: dailyread
publishDate: 16 Nov 2026
description: A small Astro site where Claude picks free, worth-reading articles and summarises them.
---

ChatGPT has a scheduled tasks feature. You can tell it something like "every Saturday, find me an exceptional free article published in the last two weeks and tell me why it's worth reading."

I liked that a lot. But the suggestions end up buried in a chat thread, and I wanted them somewhere I could actually browse later.

So I built **Daily Read**.

Repository: [github.com/brunoabpinto/dailyread](https://github.com/brunoabpinto/dailyread)

Live site: [readoftheday.netlify.app](https://readoftheday.netlify.app/)

## What I wanted

- One good article per post, free to read, published recently
- A short summary and a line on why it's worth my time
- A link to the original, always
- No database, no backend, no admin panel

Just files in a repo.

## How it works

Each post is a folder:

`src/content/posts/<title-slug>/index.md`

With an optional `cover.jpg` next to it.

To add a post, I run:

```bash
npm run new-post
```

The script asks Claude Code to:

1. Search for a free article published in the last two weeks.
2. Skip anything already featured.
3. Return the title, author, source, topic, a summary and why it's worth reading.

Then it grabs the article's preview image and writes the Markdown file.

If I want to publish it right away, `npm run new-post:push` does the same, then commits and pushes. Netlify rebuilds. Done.

## Why Astro

Content collections are exactly this use case. Every post has a typed schema, so if Claude returns something weird, the build fails instead of shipping a broken page.

Images get optimized at build time too. The cover for the first post went from 306kB to 70kB without me doing anything.

## The slow part

The script is not fast. Claude actually opens the candidate articles to check they're free and recent, and that takes a few minutes.

I could make it skip that step. But then I'd occasionally get a paywall, and the whole point is that you can just click and read.

## What I would improve next

- Run it on a schedule instead of by hand
- An RSS feed
- Filter by topic

But for now, it does the one thing I wanted: every time I open it, there's something worth reading.
