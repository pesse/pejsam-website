# Claude Context: ROQ Blog Project

**Purpose**: This document provides comprehensive context for Claude Code sessions to avoid repeated analysis and reduce token costs.

**Last Updated**: 2025-10-22

---

## ⚠️ Important User Preferences

- **Git Commits**: User handles ALL git operations (add, commit, push) themselves. NEVER create commits or push to git. Only create/modify files as requested.

---

## Quick Facts

- **Project Name**: pejgan-website
- **Version**: 1.0.0-SNAPSHOT
- **Framework**: Quarkus 3.28.4
- **ROQ Version**: 1.10.1
- **Java Version**: 21
- **Build Tool**: Maven 3.x
- **Theme**: pejsam (custom theme)

---

## What is ROQ?

ROQ is a static site generator (SSG) built on Java/Quarkus that makes building websites and blogs fast and straightforward. It uses **Qute templating** and offers:
- Fast performance leveraging Quarkus's development mode
- Built-in search capabilities
- Sitemap generation
- Blog post series support
- Simple, readable templating with Qute
- Extensibility through Quarkus extensions

---

## Project Structure

```
roq-with-blog/
├── content/                    # Static content files (markdown, HTML)
│   ├── 404.html               # Custom 404 page
│   ├── index.html             # Home page template
│   ├── about.md               # About page (markdown)
│   └── posts/                 # Blog posts directory
│       └── YYYY-MM-DD-slug/   # Date-based folder naming
│           ├── index.md       # Blog post content
│           └── [images]       # Associated media
├── data/                      # Configuration and data files (YAML)
│   ├── authors.yml            # Author profiles
│   └── menu.yml               # Navigation menu configuration
├── public/                    # Static assets (images, CSS, JS)
│   └── images/               # Image assets
├── docs/                      # Project documentation
│   └── CLAUDE_CONTEXT.md     # This file
├── src/
│   ├── main/
│   │   ├── docker/           # Docker configuration files
│   │   ├── java/             # Java source code (optional customization)
│   │   └── resources/        # Application properties
│   └── test/                 # Test files
├── target/                   # Build output (auto-generated, gitignored)
│   └── roq-templates/        # Generated templates
├── .github/workflows/        # CI/CD configuration
│   └── deploy.yml            # GitHub Pages deployment
├── pom.xml                   # Maven project configuration
├── mvnw / mvnw.cmd          # Maven wrapper scripts
└── README.md                # Project documentation
```

---

## Key Configuration Files

### pom.xml
Maven build configuration defining:
- Dependencies (quarkus-roq, quarkus-roq-theme-default)
- Java version (21)
- Quarkus platform version
- Build plugins

### application.properties
Located at: `src/main/resources/config/application.properties`
Quarkus runtime configuration (currently empty in starter)

### data/authors.yml
Author profiles with structure:
```yaml
author-id:
  name: Full Name
  nickname: Display Name
  avatar: URL or path
  bio: Short biography
  github: username
  twitter: handle
  linkedin: profile
```

### data/menu.yml
Navigation menu configuration:
```yaml
- title: Menu Item
  url: /path
  icon: fa-icon-name  # Font Awesome icons supported
```

---

## Content Management

### Blog Posts

**Location**: `content/posts/YYYY-MM-DD-slug/index.md`

**Frontmatter Structure**:
```yaml
---
layout: :theme/post
title: Post Title
description: Meta description for SEO
image: blog-image.jpg
tags:
  - tag1
  - tag2
author: author-id  # Reference from authors.yml
---

Post content in Markdown...
```

**Supported Frontmatter Fields**:
- `layout` - Template selection (`:theme/post`, `:theme/page`, etc.)
- `title` - Page/post title
- `description` - Meta description
- `image` - Featured image (relative to post directory)
- `tags` - Array of tags/categories
- `author` - Author ID from authors.yml
- Custom fields accessible via `{page.data.*}` in templates

### Static Pages

**Location**: `content/page-name.md` or `content/page-name.html`

Pages use similar frontmatter structure but typically use `layout: :theme/page`

### Date Formatting

Use Qute expressions in templates:
```
{page.date.format('YYYY-MM-DD')}
{page.date.format('MMMM DD, YYYY')}
```

---

## Templating with Qute

### Template Locations
- Theme templates: Built into `quarkus-roq-theme-default` dependency
- Custom templates: Can be placed in project to override theme
- Generated output: `target/roq-templates/`

### Available Layouts
- `:theme/main` - Base layout
- `:theme/index` - Homepage layout
- `:theme/post` - Blog post layout
- `:theme/page` - Static page layout
- `:theme/tag` - Tag page layout
- `:theme/404` - Error page layout

### Data Access in Templates

**CDI (Context and Dependency Injection)**:
```
{#for id in cdi:authors.fields}
  {cdi:authors.get(id).name}
{/for}
```

**Page Data**:
```
{page.title}
{page.data.customField}
{page.date.format('YYYY')}
```

---

## Custom Theme: "pejsam"

### ⚠️ CRITICAL: Theme File Locations

**All layout and theme changes MUST be made in these directories:**

- **Partials**: `src/main/resources/templates/partials/pejsam/`
- **Layouts**: `src/main/resources/templates/theme-layouts/pejsam/`

**DO NOT modify the default theme files. Only modify files in the pejsam theme directories.**

### Theme Structure

**Partials** (reusable components):
- `404.html` - 404 error page content
- `head.html` - HTML head section (meta tags, CSS links)
- `pagination.html` - Pagination controls
- `sidebar-about.html` - About section in sidebar
- `sidebar-contact.html` - Contact section in sidebar
- `sidebar-copyright.html` - Copyright footer
- `sidebar-menu.html` - Sidebar navigation menu

**Layouts** (page templates):
- `404.html` - 404 error page layout
- `default.html` - Base/default layout
- `index.html` - Homepage layout
- `main.html` - Main layout wrapper
- `page.html` - Static page layout
- `post.html` - Blog post layout
- `tag.html` - Tag archive layout

### Using the Custom Theme

In content frontmatter, reference layouts with `:pejsam/` prefix:
```yaml
---
layout: :pejsam/post
title: My Post
---
```

---

## Development Workflow

### Starting Development Server
```bash
./mvnw quarkus:dev
```
- Live reload enabled
- Dev UI available at `http://localhost:8080/q/dev/`
- Site served at `http://localhost:8080`

### Building for Production
```bash
./mvnw package
```
Output in `target/` directory

### Native Compilation
```bash
./mvnw package -Dnative
```
Creates GraalVM native image for minimal size/startup time

---

## Deployment

### GitHub Pages (Configured)

**Workflow**: `.github/workflows/deploy.yml`
- **Trigger**: Push to `master` branch or manual workflow dispatch
- **Steps**:
  1. Build site with Maven (`./mvnw package quarkus:run` with `QUARKUS_ROQ_GENERATOR_BATCH=true`)
  2. Upload artifact from `target/roq/` directory
  3. Deploy to GitHub Pages using `actions/deploy-pages@v4`
- **Requirements**: 
  - GitHub token (automatic via `GITHUB_TOKEN`)
  - GitHub Pages enabled in repository settings
  - Pages source set to "GitHub Actions"
- **Build Details**:
  - Java 21 (Temurin distribution)
  - Maven caching enabled for faster builds
  - Static site output: `target/roq/`

### Docker Support

Three Dockerfile variants available:
- `src/main/docker/Dockerfile.jvm` - Full JVM deployment
- `src/main/docker/Dockerfile.native` - Native image
- `src/main/docker/Dockerfile.native-micro` - Ultra-minimal native

---

## Current State

### What's Implemented
- ✅ Basic project structure
- ✅ Sample blog post (2024-10-13-the-first-roq)
- ✅ Author management system
- ✅ Navigation menu
- ✅ Homepage (index.html)
- ✅ About page (about.md)
- ✅ Custom 404 page
- ✅ Default theme
- ✅ GitHub Pages deployment pipeline

### What's Empty/Customizable
- Java source code (`src/main/java/`) - Optional for advanced customization
- Application properties - Can add Quarkus configuration
- Custom CSS/JS - Can be added to `public/` directory
- Additional pages/posts - Ready to be added

---

## Common Customizations

### Adding a New Blog Post
1. Create directory: `content/posts/YYYY-MM-DD-slug/`
2. Create file: `index.md` with frontmatter
3. Add images in same directory
4. Reference images relatively: `![alt](image.jpg)`

### Adding a New Author
Edit `data/authors.yml`:
```yaml
new-author-id:
  name: Full Name
  nickname: Display Name
  avatar: /images/avatar.jpg
  bio: Biography text
  github: username
```

### Modifying Navigation
Edit `data/menu.yml`:
```yaml
- title: New Page
  url: /new-page
  icon: fa-solid fa-icon-name
```

### Custom Styling
Add CSS files to `public/css/` and reference in layouts or frontmatter

---

## Key Dependencies

From `pom.xml`:
- `io.quarkiverse.roq:quarkus-roq:1.10.1`
- `io.quarkiverse.roq:quarkus-roq-theme-default:1.10.1`
- `io.quarkus:quarkus-arc` (Dependency Injection)
- `io.quarkus:quarkus-junit5` (Testing)

---

## Useful Resources

- ROQ Official Site: https://iamroq.com/
- Quarkus Documentation: https://quarkus.io/guides/
- Qute Templating: https://quarkus.io/guides/qute

---

## Notes for Future Sessions

- ROQ uses Quarkus's hot reload - changes to content/data files are reflected immediately in dev mode
- Frontmatter is YAML between `---` markers
- Font Awesome icons are available (regular and solid variants)
- Tag pages are automatically generated from post tags
- Author cards can be rendered using theme components
- Static files in post directories are served relative to the post URL
