# ROQ & Quarkus Static Site Generator - Key Learnings

**Purpose**: This document captures critical knowledge about working with ROQ (Quarkus Static Site Generator) to avoid repeated mistakes and ensure consistent implementation.

**Last Updated**: 2025-11-03

---

## 🎯 Critical Rules

### 1. NEVER Use Hardcoded Paths

**❌ WRONG:**
```html
<a href="/projects/my-project">Link</a>
<a href="/about">About</a>
```

**✅ CORRECT:**
```html
<a href="{site.url('projects/my-project')}">Link</a>
<a href="{site.url('about')}">About</a>
<a href="{site.url}">Homepage</a>
```

**Why**: The site may be deployed to a non-root path (e.g., GitHub Pages with `/repository-name/`). Using `{site.url()}` ensures links work in all deployment scenarios.

---

## 🎨 Theme Structure & Naming

### Directory Layout
```
src/main/resources/templates/
├── theme-layouts/{theme-name}/     # Main layouts
│   ├── default.html                # Base layout
│   ├── main.html                   # Wrapper layout
│   ├── index.html                  # Homepage layout
│   ├── page.html                   # Static page layout
│   ├── post.html                   # Blog post layout
│   └── tag.html                    # Tag archive layout
└── partials/{theme-name}/          # Reusable components
    ├── head.html                   # <head> section
    ├── top-header.html             # Navigation header
    ├── footer.html                 # Footer
    └── ...
```

### Theme Configuration

**application.properties:**
```properties
site.theme=pejsam
```

### Layout References - THE CRITICAL DIFFERENCE

**From Content Files (content/*.html, content/*.md):**
```yaml
---
layout: :theme/index     # ← Use :theme/ prefix
---
```

**From Theme Layouts (theme-layouts/{theme-name}/*.html):**
```yaml
---
layout: pejsam/default   # ← Use {theme-name}/ prefix (no colon!)
---
```

**Why Different?**
- `:theme/` resolves to the active theme from `application.properties`
- `pejsam/` is the actual directory reference within the theme folder
- Mixing these up will cause "Layout not found" errors

---

## 📐 Layout Hierarchy & Structure

### Insert Points

**Basic insertion:**
```html
<!-- In parent layout -->
{#insert /}                    <!-- Default insertion point -->

<!-- In child layout -->
<p>This content goes here</p>
```

**Named insertions:**
```html
<!-- In parent layout -->
{#insert header}
  <!-- Default header content -->
{/}

<!-- In child layout -->
{#insert header}
  <custom-header>My Header</custom-header>
{/}
```

### Common Layout Structure

```
default.html (base HTML structure)
    ↓
main.html (adds header/footer/container)
    ↓
index.html | page.html | post.html (specific layouts)
    ↓
Content files
```

---

## 📝 Qute Templating Specifics

### What Works

**Page data:**
```html
{page.title}
{page.data.customProperty}
{page.data.heroTitle}
```

**Site data:**
```html
{site.url('path')}
{site.description}
{site.data.name}
```

**CDI beans:**
```html
{cdi:menu.items}
{cdi:authors.get(id).name}
```

**Rendering HTML without escaping:**
```html
{page.data.heroSubtitle.raw()}  <!-- Renders <strong> tags -->
```

**Date formatting:**
```html
{post.date.format('yyyy-MM-dd')}
{post.date.format('MMMM DD, YYYY')}
```

### What DOESN'T Work

**❌ Java namespaces:**
```html
{java:time.format('yyyy')}  <!-- FAILS! No java: namespace -->
```

**Solution:** Use date formatting on available objects:
```html
{post.date.format('yyyy')}  <!-- Format existing dates -->
```

---

## 🎯 CSS Organization

### Recommended Structure

**public/css/theme.css** - Base theme styles:
- Typography
- Navigation
- Footer
- Layout structure

**public/css/styles.css** - Component/page-specific:
- Hero sections
- Project cards
- Content sections
- Page-specific styling

### Including CSS in Templates

**In head.html:**
```html
<!-- PEJSAM Custom Styles -->
<link rel="stylesheet" href="{site.url('css/theme.css')}">
<link rel="stylesheet" href="{site.url('css/styles.css')}">
```

**Note:** Always use `{site.url()}` - never hardcoded paths!

---

## 🧪 Testing & Building

### Development Server
```bash
./mvnw quarkus:dev
```
- Hot reload enabled
- View at http://localhost:8080

### Production Testing (ALWAYS USE THIS!)
```bash
QUARKUS_ROQ_GENERATOR_BATCH=true \
QUARKUS_HTTP_ROOT_PATH="/pejsam-roq/roq-with-blog/target/roq" \
./mvnw package quarkus:run -DskipTests
```

**Why this matters:**
- Tests with a non-root path
- Simulates GitHub Pages deployment
- Verifies all `{site.url()}` links work correctly

### Production Build
```bash
./mvnw package
```
Output: `target/roq/`

---

## 🚫 Common Mistakes to Avoid

### 1. Multiple Changes Without Testing
- Make ONE change at a time
- Test after each change
- Don't assume multiple changes will work together

### 2. Hardcoded Paths
```html
<!-- ❌ -->
<a href="/about">About</a>

<!-- ✅ -->
<a href="{site.url('about')}">About</a>
```

### 3. Using Invalid Qute Expressions
```html
<!-- ❌ -->
{java:time.format('yyyy')}

<!-- ✅ -->
2025
```

---

## 📚 Useful Qute Patterns

### Conditional Rendering
```html
{#if page.data.heroTitle}
  <h1>{page.data.heroTitle}</h1>
{/if}

{#if site.data.containsKey('analytics')}
  {#ga4 tag=site.data.getJsonObject('analytics').getString('ga4', '') /}
{/if}
```

### Loops
```html
{#for item in cdi:menu.items}
  <a href="{site.url(item.path)}">{item.title}</a>
{/for}
```

### Fallback Values
```html
{site.data.name ?: 'PEJSAM'}  <!-- Use 'PEJSAM' if name is empty -->
```

---

## 📋 Quick Reference Checklist

When creating a new page/layout:

- [ ] Content uses `:theme/layout-name`
- [ ] Theme layouts use `theme-name/layout-name`
- [ ] All links use `{site.url('path')}`
- [ ] Images use `{site.url('images/file.jpg')}` or CSS `url('../images/file.jpg')`
- [ ] No `java:` or other invalid namespaces
- [ ] CSS files linked with `{site.url('css/file.css')}`
- [ ] Tested with production build command (non-root path)

---

## 🎓 Resources

- **ROQ Documentation**: https://iamroq.com/
- **Quarkus Qute Guide**: https://quarkus.io/guides/qute
- **Project Context**: See `.claude/CLAUDE.md` for project-specific details

---

**Remember:** When in doubt, check this file first. These patterns have been tested and verified to work with ROQ.
