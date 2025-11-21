Purpose
-------
This file gives concise, project-specific instructions to help an AI coding agent be productive in the `pejsam-roq` repository. It summarizes the architecture, developer workflows, important conventions, and exact commands to run locally.

**Quick Start (local dev)**
- **Dev server**: Run `./mvnw quarkus:dev` for hot reload during development.
- **Production-like test (REQUIRED to validate links)**:
  - Use the exact command shown to test non-root deployment paths:
    `QUARKUS_ROQ_GENERATOR_BATCH=true QUARKUS_HTTP_ROOT_PATH=<path from webroot> ./mvnw package quarkus:run -DskipTests`
  - This simulates GitHub Pages and verifies all `{site.url(...)}` links.
  - For testing in GitHub: Always use `QUARKUS_HTTP_ROOT_PATH="/pejsam-website/"`

**Where to edit templates & theme**
- Theme files for the custom theme `pejsam` live in:
  - `src/main/resources/templates/partials/pejsam/` (partials/components)
  - `src/main/resources/templates/theme-layouts/pejsam/` (layouts)
- DO NOT edit default theme files from the dependency; only change files in the `pejsam` theme directories.

**Critical conventions & gotchas (must follow exactly)**
- **Never hardcode paths**: Always use `{site.url('path')}` or `{site.url}` in templates and content. Hardcoded `/path` breaks non-root deployment.
  - Wrong: `<a href="/about">About</a>`
  - Right: `<a href="{site.url('about')}">About</a>`
- **Layout reference rules**:
  - In content frontmatter use `layout: :theme/post` (the `:theme/` prefix resolves to the active theme).
  - In theme layout files reference other theme layouts with the actual theme name: `layout: pejsam/default` (no colon).
- **One change, test it**: Make a single focused change and run the production-like test above before doing more.

**Data & static assets**
- Content: `content/` (markdown and HTML pages)
- Authors and menus: `data/authors.yml`, `data/menu.yml`
- Static assets (CSS/JS/images): `public/css/`, `public/images/` (or use web-bundler assets in `src/main/resources/web/app/`)

**Build outputs & CI**
- Static site output: `target/roq/` (used by GitHub Pages workflow)
- Quarkus packaged app: `target/quarkus-app/` (contains `quarkus-run.jar`)
- CI deploy workflow: `.github/workflows/deploy.yml` — push to `master` or manual dispatch triggers Pages deployment.

**Helpful files to inspect when making changes**
- `CLAUDE.md` and `docs/ROQ_LEARNINGS.md` — comprehensive project rules and examples.
- `pom.xml` — Quarkus/ROQ versions and build plugins.
- Theme partials/layouts under `src/main/resources/templates/.../pejsam/` to see exact template patterns.

**Agent operational rules / user preferences**
- The human user performs all local `git` operations: do not create commits or push changes without explicit user instruction.
- When in doubt about live behavior, run the non-root production test command above.

Questions or missing details? Ask the repo owner which files are safe to edit or whether a quick CI run is allowed before committing.
