# ktrysmt.github.io

Kotaro Yoshimatsu's personal tech blog (SRE / Security Governance). Built with Jekyll 4.4.x, deployed to GitHub Pages.

- URL: https://ktrysmt.github.io/blog/
- Language: Japanese (ja_JP)
- Timezone: Asia/Tokyo

## Tech Stack

- Ruby 3.3 / Jekyll 4.4.x
- Plugins: jekyll-seo-tag, jemoji, jekyll-mentions, jekyll-redirect-from, jekyll-sitemap, jekyll-feed
- Syntax highlighting: Highlight.js (external)
- Diagrams: Mermaid.js v11
- Custom plugin: `_plugins/tag_pages.rb` (auto-generates tag archive pages)

## Project Structure

```
_config.yml        # Jekyll configuration
_posts/            # Blog posts (112 articles, 2014-present)
_drafts/           # Draft posts (not published; previewed with `jekyll s --drafts`)
_layouts/          # default, post, list, tag, top, default-top
_includes/         # navigation, profile, share, footer, etc.
_plugins/          # tag_pages.rb (tag archive generator)
assets/css/        # style.css (custom, no framework)
assets/images/     # avatar, favicon, og-default, blog images
assets/js/         # highlight.pack.js
```

## Setup

```bash
mise install ruby 3.1
mise use ruby@3.1
bundle install
```

## Local Development

```bash
bundle exec jekyll s -P 8888 -H 0.0.0.0
curl http://localhost:8888/blog/
```

## Use Browser

* use `chrome-devtools mcp` or `/agent-browser skill`

## Build

```bash
bundle exec jekyll build
# Output: _site/
```

## Drafts

- 下書きは `_drafts/` に配置する（日付なしのファイル名でよい）
- リポジトリのルート直下に `blog-draft-*.md` のような下書きファイルを置かない
- プレビュー: `bundle exec jekyll s -P 8888 -H 0.0.0.0 --drafts`
- 公開時は `_posts/YYYY-MM-DD-title.md` にリネームして移動する

## Post Format

```yaml
---
layout: post
title: "Title"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [Category]  # see "Allowed Categories" below
published: true
description: "Summary"
tags: [tag1, tag2]
---
```

## Allowed Categories

Posts must use exactly one of the following categories:

- `AI Engineering`
- `Cloud Infrastructure`
- `Developer Tools`
- `Engineering Practices`
- `Programming`
- `Security Governance`

- Permalink: `/blog/:title/`
- Mermaid diagrams: use ```` ```mermaid ```` code blocks
- `.card-preview` links must use absolute URL (`https://ktrysmt.github.io/blog/...`). Relative paths are not supported by Microlink.

## CI/CD

- GitHub Actions (`.github/workflows/jekyll.yml`)
- Trigger: push to `master` or manual dispatch
- Build: `bundle exec jekyll build` with `JEKYLL_ENV=production`
- Deploy: GitHub Pages

## Notes

- No automated tests or linters configured
- `.gitignore`: `_site/`, `.jekyll-cache/`, `.DS_Store`
