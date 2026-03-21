# Development Guide

This guide covers setup, development workflow, and common tasks for the Hugo blog.

## Prerequisites

- **Hugo** - Static site generator
  - Install: https://gohugo.io/installation/
- **Git** - Version control
- **Text editor** - Any editor that supports Markdown

## Initial Setup

### Clone Repository
```bash
git clone git@github.com:alikhil/hugo-blog.git
cd hugo-blog

# Pull theme submodule
git submodule update --init --recursive
```

## Development Workflow

### Start Development Server
```bash
hugo server -D -E -F --forceSyncStatic -e dev
```

Flags explained:
- `-D` - Include draft content
- `-E` - Include expired content
- `-F` - Include future-dated content
- `--forceSyncStatic` - Force sync of static files
- `-e dev` - Use development environment

The server runs on http://localhost:1313 with live reload.

### Create New Post
```bash
hugo new posts/your-post-slug.md
```

This creates a new file in `content/posts/` using the template from `archetypes/default.md`.

### Create New Page
```bash
hugo new pages/your-page-name.md
```

### Build for Production
```bash
hugo
```

Output is generated in the `public/` directory.

## File Locations

### Adding Content
- **Blog posts**: `content/posts/post-name.md`
- **Static pages**: `content/pages/page-name.md`
- **Images/files**: `static/` directory

### Customizing Theme
- **Partials**: `layouts/partials/` (overrides theme partials)
- **Shortcodes**: `layouts/shortcodes/` (custom shortcodes)
- **Theme files**: `themes/PaperModX/` (git submodule, avoid editing)

### Configuration
- **Main config**: `config.yml`
- **Archetypes**: `archetypes/default.md`

## Common Tasks

### Preview Draft Posts
Drafts are posts with `draft: true` in front matter. They're included when using `-D` flag:
```bash
hugo server -D
```

### Update Theme
```bash
cd themes/PaperModX
git pull origin master
cd ../..
git add themes/PaperModX
git commit -m "Update PaperModX theme"
```

### Add Custom Shortcode
1. Create file in `layouts/shortcodes/shortcode-name.html`
2. Use in posts: `{{< shortcode-name >}}`

### Add Custom Partial
1. Create file in `layouts/partials/partial-name.html`
2. Reference in templates: `{{ partial "partial-name.html" . }}`

### Change Menu Items
Edit `menu.main` section in `config.yml`:
```yaml
menu:
  main:
    - weight: 1
      identifier: home
      name: Home
      url: /
```

## Deployment

### Using Deploy Script
```bash
./deploy.sh
```

This script handles the build and deployment process.

## Directory Structure Reference

```
hugo-blog/
├── archetypes/         # Content templates
│   └── default.md
├── content/            # All content
│   ├── posts/         # Blog posts
│   ├── pages/         # Static pages
│   ├── archives.md
│   └── search.md
├── layouts/           # Custom layouts
│   ├── partials/      # Custom partials
│   └── shortcodes/    # Custom shortcodes
├── static/            # Static assets
├── themes/            # Theme directory
│   └── PaperModX/    # Theme (git submodule)
├── public/            # Generated site (git ignored)
├── resources/         # Hugo cache
├── data/              # Data files
├── assets/            # Theme assets
├── config.yml         # Main configuration
└── deploy.sh          # Deployment script
```

## Troubleshooting

### Theme Not Loading
```bash
git submodule update --init --recursive
```

### Changes Not Appearing
- Clear Hugo cache: `rm -rf resources/`
- Restart development server
- Check file is in correct location

### Build Errors
- Validate YAML front matter syntax
- Check for unclosed shortcodes
- Review Hugo error messages in terminal

## Hugo Resources

- **Hugo Documentation**: https://gohugo.io/documentation/
- **PaperModX Theme Docs**: https://reorx.github.io/hugo-PaperModX/docs/f
- **Hugo Shortcodes**: https://gohugo.io/content-management/shortcodes/
- **Hugo Variables**: https://gohugo.io/variables/

## Configuration Highlights

### Key Settings in config.yml

- **Base URL**: `https://alikhil.dev/`
- **Theme**: PaperModX
- **Pagination**: 5 posts per page
- **Syntax Highlighting**: Dracula theme
- **Date Format**: `2 January 2006`
- **Google Analytics**: Enabled with custom analytics
- **Comments**: Enabled
- **TOC**: Right-side, open by default

### Site Parameters
- Author: Alik Khilazhev
- Environment: production
- Theme mode: auto (light/dark)
- Social icons: GitHub, Telegram, Stack Overflow, LinkedIn, RSS
- Search: Fuse.js integration

## Testing Before Deployment

1. Build locally: `hugo`
2. Check `public/` directory output
3. Verify no broken links
4. Test responsive design
5. Check syntax highlighting
6. Validate front matter in new posts
