# Quick Reference Commands

Common commands for working with this ROQ blog project.

---

## Development

### Start Development Server
```bash
./mvnw quarkus:dev
```
- Runs site at `http://localhost:8080`
- Live reload enabled (content changes reflected immediately)
- Dev UI available at `http://localhost:8080/q/dev/`
- Press `Ctrl+C` to stop

### Start Dev Server in Background
```bash
./mvnw quarkus:dev &
```

---

## Building

### Build for Production
```bash
./mvnw package
```
- Generates static site in `target/` directory
- Output ready for deployment

### Clean Build
```bash
./mvnw clean package
```
- Removes previous build artifacts
- Fresh build from scratch

### Native Build (GraalVM)
```bash
./mvnw package -Dnative
```
- Creates native executable
- Faster startup, lower memory footprint
- Requires GraalVM installation

---

## Testing

### Run Tests
```bash
./mvnw test
```

### Run Tests with Clean
```bash
./mvnw clean test
```

---

## Content Management

### Add New Blog Post
```bash
# Create post directory
mkdir -p content/posts/$(date +%Y-%m-%d)-your-post-slug

# Create post file
touch content/posts/$(date +%Y-%m-%d)-your-post-slug/index.md
```

Then edit the `index.md` file with frontmatter:
```yaml
---
layout: :theme/post
title: Your Post Title
description: Post description
author: your-author-id
tags:
  - tag1
  - tag2
---

Your content here...
```

### Add New Page
```bash
# Create markdown page
touch content/your-page.md
```

Or create HTML page:
```bash
touch content/your-page.html
```

---

## Git Operations

### View Status
```bash
git status
```

### Commit Changes
```bash
git add .
git commit -m "Your commit message"
```

### Push to GitHub (Triggers Deployment)
```bash
git push origin main
```

---

## Docker

### Build JVM Docker Image
```bash
docker build -f src/main/docker/Dockerfile.jvm -t roq-blog:jvm .
```

### Build Native Docker Image
```bash
docker build -f src/main/docker/Dockerfile.native -t roq-blog:native .
```

### Run Docker Container
```bash
docker run -i --rm -p 8080:8080 roq-blog:jvm
```

---

## Maven Wrapper

### Update Maven Wrapper
```bash
./mvnw wrapper:wrapper
```

### Verify Maven Installation
```bash
./mvnw --version
```

---

## Useful Dev Mode Commands

When running `./mvnw quarkus:dev`, you can use these keys in the terminal:

- `r` - Re-run tests
- `h` - Display help
- `q` - Quit dev mode
- `s` - Force restart
- `i` - Toggle instrumentation based reload
- `l` - Toggle live reload
- `w` - Open dev UI in browser

---

## Project Information

### List Dependencies
```bash
./mvnw dependency:tree
```

### List Quarkus Extensions
```bash
./mvnw quarkus:list-extensions
```

### View Project Info
```bash
./mvnw quarkus:info
```

---

## Troubleshooting

### Clear Target Directory
```bash
rm -rf target/
```

### Fix Maven Wrapper Permissions (Linux/Mac)
```bash
chmod +x mvnw
```

### View Detailed Build Output
```bash
./mvnw package -X
```

### Check Java Version
```bash
java -version
```
Should be Java 21 or higher.

---

## GitHub Actions

### View Deployment Status
Check: https://github.com/YOUR-USERNAME/YOUR-REPO/actions

### Manually Trigger Deployment
Push to `main` branch or use GitHub UI to re-run workflow

---

## File Locations Reference

- Content: `content/`
- Blog Posts: `content/posts/YYYY-MM-DD-slug/`
- Data Files: `data/` (authors.yml, menu.yml)
- Static Assets: `public/`
- Images: `public/images/`
- Source Code: `src/main/java/`
- Resources: `src/main/resources/`
- Build Output: `target/`
- Docker Files: `src/main/docker/`

---

## Quick Workflow

**Typical development session**:
```bash
# 1. Start dev server
./mvnw quarkus:dev

# 2. Edit content in content/ or data/
# 3. View changes at http://localhost:8080

# 4. When satisfied, commit and push
git add .
git commit -m "Add new post"
git push origin main

# 5. GitHub Actions will automatically deploy
```
