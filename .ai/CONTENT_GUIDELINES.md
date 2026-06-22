# Content Writing Guidelines

These guidelines ensure consistency in blog posts and maintain the author's authentic voice.

## Language & Style

### Language Preferences
- Use **American English** (not British English)
  - Examples: "organize" not "organise", "color" not "colour"
- Avoid long em dashes (––)
- Use simpler vocabulary (avoid overly sophisticated words)
- Write in a natural, conversational tone that doesn't sound overly formal or AI-generated

### Writing Style
Reference existing posts in `/content/posts/` for style consistency:
- Keep language clear and accessible
- Use practical examples when explaining concepts
- Balance technical accuracy with readability
- Write for both beginners and experienced readers
- Include personal experiences and observations when relevant

### Personal Technical Notes
For opinionated or experience-based posts, preserve the author's first-person voice:
- Keep concrete pain points and examples ("this took 40 minutes", "installer scripts broke my setup")
- Prefer direct personal framing over neutral tutorial language
- Add structure with headings, but do not over-polish the text into a generic article
- Use short bridge sentences to connect sections when the story moves from context to problems to decision
- Avoid stretching the category to fit the technology; use categories for the broad post type and tags for specific tools

## Post Structure

### Front Matter
All posts must include YAML front matter:
```yaml
---
title: "Post Title Here"
date: YYYY-MM-DDTHH:MM:SS+03:00
description: "Short summary for SEO and previews."
categories:
- category-name
tags:
- specific-topic
---
```

### Post Organization
1. **Opening paragraph** - Brief introduction or context
2. **`<!--more-->`** - Add this marker after the first paragraph for excerpt
3. **Main content** - Organized with clear headings
4. **Sections** - Use H2 (##) for main sections, H3 (###) for subsections

## Content Features

### Code Blocks
- Use fenced code blocks with language identifiers
- Syntax highlighting is enabled (Dracula theme)
- Example:
  ````markdown
  ```go
  package main
  ```
  ````

### Categories & Tags
- Use categories for broad topics (e.g., "opensource", "kubernetes", "golang")
- Categories should be lowercase and hyphenated
- For personal notes about a setup or tool, `note` can be a better category than forcing a broad engineering category
- Use tags for concrete technologies, tools, and searchable terms (e.g., `nix`, `nix-darwin`, `home-manager`, `homebrew`, `dotfiles`)
- Common categories based on site keywords:
  - kubernetes
  - software-engineering
  - golang
  - devops
  - sre
  - self-hosting
  - homelab
  - mac
  - note

### Links
- Use descriptive link text
- External links open normally (theme handles this)
- Internal links use relative paths

### Images
- Store images in `/static/` directory
- Reference with absolute paths from root: `/path/to/image.png`
- Include alt text for accessibility

## Topics & Themes

Based on site configuration, suitable topics include:
- Software engineering practices
- Kubernetes and container orchestration
- DevOps and SRE concepts
- Self-hosting and homelab setups
- Open source contributions
- Golang development
- Technical tutorials and guides
- Personal experiences with technology

## SEO Considerations

- **Title**: Clear, descriptive, under 60 characters when possible
- **Description**: Add a concise `description` to new posts. It should explain the specific angle of the post, not just repeat the title.
- **First paragraph**: Should summarize the post (used as excerpt)
- **Keywords**: Naturally incorporate relevant keywords from site config
- **Tags**: Prefer specific searchable tags over broad categories for tools and technologies
- **Headings**: Use proper hierarchy (H1 is title, start content with H2)

## Date Format

- Front matter uses ISO 8601 format with timezone: `2025-12-18T04:14:12+03:00`
- Display format (from config): `2 January 2006`

## Special Features

### Table of Contents
- Automatically generated from headings
- Appears on right side by default
- Opens by default
- No special markup needed

### Reading Time
- Automatically calculated
- No special markup needed

### Share Buttons
- Automatically added to posts
- No configuration needed

## Creating New Posts

Use Hugo's archetype system:
```bash
hugo new posts/post-slug-here.md
```

This creates a new post with the default front matter template from `/archetypes/default.md`.

## Common Pitfalls to Avoid

1. Don't use overly complex vocabulary just to sound technical
2. Avoid British English spellings
3. Don't forget the `<!--more-->` marker for excerpts
4. Keep code examples practical and working
5. Test local build before committing: `hugo server -D -E -F --forceSyncStatic -e dev`
