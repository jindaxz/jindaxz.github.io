# Jinda Zhang’s academic homepage and blog

The academic homepage lives at `/`. The blog lives at `/blog/` on the same
GitHub Pages site. Posts are written in Markdown and rendered by Jekyll;
there is no database, browser-side Markdown renderer, or extra hosting service.
The original academic homepage uses Jon Barron’s
[website template](https://github.com/jonbarron/jonbarron_website).

## Write a post

Add `_posts/YYYY-MM-DD-short-title.md`:

```markdown
---
title: "Your post title"
date: 2026-10-22 12:00:00 -0700
description: "A short summary for the blog index and search previews."
tags: [Research, Engineering]
---

Write your post here in Markdown. Start sections with `##` because the layout
already displays the title as the main heading.
```

Use the actual publication date and time, including its UTC offset. Future-dated
posts stay hidden until that date and a subsequent build. The filename determines
the URL: `short-title` becomes `/blog/short-title/`. Keep filenames stable after
publication to preserve links. Put images in `assets/` and link to them with
`![Description](/assets/filename.png)`.

The blog index, reading time, RSS feed (`/blog/feed.xml`), and sitemap
(`/sitemap.xml`) update with each build. The site’s title, address, and timezone
are in `_config.yml`. Blog templates are in `_layouts/`; its styles are isolated
in `assets/blog.css`.

## Preview locally

With Ruby and Bundler installed:

```sh
bundle install
bundle exec jekyll serve
```

Open <http://localhost:4000/blog/>. To check a production build:

```sh
JEKYLL_ENV=production bundle exec jekyll build --strict_front_matter
```

## Publish with GitHub Pages

This setup works with GitHub Pages’ built-in Jekyll builder and supported
plugins. In the repository’s **Settings → Pages**, use **Deploy from a branch**,
select `main` and `/ (root)`. Commit and push the source changes to publish;
GitHub builds the Markdown automatically. No separate blog repository or DNS
change is needed. Do not add `.nojekyll`, which would disable Markdown processing.

If the repository instead uses a custom GitHub Actions deployment, that workflow
must run Jekyll and publish `_site/`, rather than upload the raw source files.
The existing hosting settings have not been changed by this implementation.
