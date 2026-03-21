# Repository Structure

This is a Hugo-based blog repository for alikhil.dev, powered by the PaperModX theme.

## Overview

- **Site URL**: https://alikhil.dev/
- **Static Site Generator**: Hugo
- **Theme**: PaperModX (https://reorx.github.io/hugo-PaperModX/docs/f)
- **Language**: English (American)
- **Author**: Alik Khilazhev

## Key Directories

### `/content/`
Contains all the content for the blog, organized into subdirectories:
- **`/content/posts/`** - Blog posts in markdown format
- **`/content/pages/`** - Static pages (e.g., talks, projects)
- **`/content/archives.md`** - Archive page
- **`/content/search.md`** - Search page

### `/layouts/`
Custom Hugo layout overrides for the theme:
- **`/layouts/partials/`** - Custom partial templates
- **`/layouts/shortcodes/`** - Custom Hugo shortcodes

### `/static/`
Static assets served directly (images, files, etc.)

### `/themes/`
Hugo theme directory (PaperModX theme installed as git submodule)

### `/public/`
Generated static site output (created by Hugo build, not committed to git)

### `/archetypes/`
Templates for new content (used by `hugo new` command)

### `/assets/`
Theme assets that are processed by Hugo

### `/data/`
Data files for Hugo templates

### `/resources/`
Hugo's resource cache directory

## Configuration

### `config.yml`
Main Hugo configuration file containing:
- Site metadata (title, base URL, language)
- Theme configuration (PaperModX)
- Menu structure (Home, Posts, Projects, Talks, Archives, Search)
- Author information
- Social media links
- Site parameters (Google Analytics, comments, date format, etc.)
- Markup settings (syntax highlighting with Dracula theme)
- Taxonomies (tags, categories, archives)

### `config.toml.backup`
Backup of previous TOML configuration

## Content Front Matter

Blog posts use YAML front matter with the following standard fields:
```yaml
---
title: "Post Title"
date: YYYY-MM-DDTHH:MM:SS+03:00
categories:
- category-name
---
```

## Build & Development

### Local Development
```bash
hugo server -D -E -F --forceSyncStatic -e dev
```

### Production Build
```bash
hugo
```

### Deployment
Use `deploy.sh` script for deployment

## Site Features

- Home page with author profile
- Blog posts with categories and tags
- Search functionality (using Fuse.js)
- Archive page
- Static pages (Projects, Talks)
- RSS feed
- Social sharing buttons
- Reading time display
- Table of contents (right-side, open by default)
- Breadcrumbs
- Comments support
- Google Analytics integration
- Syntax highlighting (Dracula theme)
- Emoji support

## Theme Customization

The site uses PaperModX theme with customizations in:
- Custom partials in `/layouts/partials/`
- Custom shortcodes in `/layouts/shortcodes/`
- Site parameters in `config.yml`

## Git Submodules

The theme is installed as a git submodule. To clone with submodules:
```bash
git clone git@github.com:alikhil/hugo-blog.git
git submodule update --init --recursive
```
