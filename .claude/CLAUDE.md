# Claude Context: ROQ Blog Project

**Purpose**: This document provides comprehensive context for Claude Code sessions to avoid repeated analysis and reduce token costs.

**Last Updated**: 2025-10-22

---

## ⚠️ Important User Preferences

- **Git Commits**: User handles ALL git operations (add, commit, push) themselves. NEVER create commits or push to git. Only create/modify files as requested.

## ⚠️ Critical ROQ Gotchas

**See `docs/ROQ_LEARNINGS.md` for comprehensive details.** Most critical:
1. **Layout references**: Content uses `:theme/`, theme files use `theme-name/`
2. **Never hardcode paths**: Always use `{site.url('path')}` - see "Linking in ROQ" section below
3. **No java: namespace**: Use `{post.date.format('yyyy')}` or static values
4. **Test with non-root path** before deploying - see "Testing Changes" section below
5. **Make ONE change at a time**, then test

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
Quarkus/ROQ runtime configuration

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

## Styling and JavaScript

### Option 1: Use public css
Add css and scripts in your site static directory: /public/css, /public/js

### Option 2: Use web-bundling

ROQ comes with a built-in Web-Bundler (https://docs.quarkiverse.io/quarkus-web-bundler) and is able to bundle, compile and minify.

To use bundling scripts (js, ts) and styles (css, scss), locate them in src/main/resources/web/app/.

```
pejsam-roq/
├── src/main/resources/web/app/
│                           ├── scripts.js
│                           └── styles.scss
```

To include the generated bundle in your template, specify the bundle tag in the html>head tag:

layouts/head.html
```
<head>
  ...
  {#bundle /}
</head>
```
It will be rendered with the relevant <script> and <style> tags to include your bundle.

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

In content frontmatter, reference layouts with `:theme/` prefix:
```yaml
---
layout: :theme/post
title: My Post
---
```

**Note**: Use `:theme/` (not `:pejsam/`) in content files. ROQ resolves this to the active theme.

---

## ⚠️ CRITICAL: Linking in ROQ

**NEVER use hardcoded links starting with `/`** - they will break when the site is deployed to a non-root path.

### Correct Link Syntax

**In Templates and Content:**
- **Site root**: `{site.url}` or `{site.url('')}` - Links to the homepage
- **Internal pages**: `{site.url('path/to/page')}` - Links to specific pages
- **Resolve URLs**: `{site.url.resolve(variable)}` - Resolve a URL from a variable
- **Current page**: `{page.url}` - Link to current page
- **Next/previous**: `{page.next.url}`, `{page.prev.url}` - Navigation links
- **Collection items**: `{post.url}` - Links to posts/documents

**Examples:**
```html
<!-- Homepage link -->
<a href="{site.url}">Home</a>

<!-- Internal page links -->
<a href="{site.url('projects/my-project')}">View Project</a>
<a href="{site.url('about')}">About</a>

<!-- Resolve a URL from a variable -->
<a href="{site.url.resolve(event.link)}">More info</a>

<!-- Loop through posts -->
{#for post in site.collections.posts}
  <a href="{post.url}">{post.title}</a>
{/for}
```

**WRONG** ❌:
```html
<a href="/projects/my-project">Link</a>
<a href="/about">About</a>
<a href="{site.url}/projects/my-project">Link</a>
```

**CORRECT** ✅:
```html
<a href="{site.url('projects/my-project')}">Link</a>
<a href="{site.url('about')}">About</a>
```

---

## Development Workflow

### ⚠️ Testing Changes (ALWAYS USE THIS)
**Use this command to test all changes:**
```bash
QUARKUS_ROQ_GENERATOR_BATCH=true QUARKUS_HTTP_ROOT_PATH="/pejsam-roq/roq-with-blog/target/roq" ./mvnw package quarkus:run -DskipTests
```

This command:
- Sets `QUARKUS_ROQ_GENERATOR_BATCH=true` for batch generation
- Sets `QUARKUS_HTTP_ROOT_PATH` to a non-root path specific to this current local environment to verify links work correctly when deployed
- Packages and runs the site
- Skips tests for faster execution

**Why this matters:** The non-root path simulates the actual deployment environment and ensures that all links using `{site.url(...)}` work correctly.

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

## Key Dependencies

From `pom.xml`:
- `io.quarkiverse.roq:quarkus-roq:1.10.1`
- `io.quarkiverse.roq:quarkus-roq-theme-default:1.10.1`
- `io.quarkus:quarkus-arc` (Dependency Injection)
- `io.quarkus:quarkus-junit5` (Testing)
- `io.quarkiverse.web-bundler:quarkus-web-bundler:1.9.3` (Bundling JS and SCSS files)

---

## Useful Resources

- ROQ Official Site: https://iamroq.com/
- Quarkus Documentation: https://quarkus.io/guides/
- Qute Templating: https://quarkus.io/guides/qute
- **ROQ Learnings & Patterns**: `docs/ROQ_LEARNINGS.md` - Detailed patterns, examples, and common mistakes

---

## Notes for Future Sessions

- ROQ uses Quarkus's hot reload - changes to content/data files are reflected immediately in dev mode
- Frontmatter is YAML between `---` markers
- Font Awesome icons are available (regular and solid variants)
- Tag pages are automatically generated from post tags
- Author cards can be rendered using theme components
- Static files in post directories are served relative to the post URL
