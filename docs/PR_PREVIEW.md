# PR Preview Deployment Documentation

## Overview

This repository now supports automatic static site preview deployments for pull requests. When a trusted contributor opens a PR, a preview site is automatically built and deployed to a unique URL, making it easy to review changes before merging.

## How It Works

### Preview URL Pattern
Each PR gets its own preview URL:
```
https://pesse.github.io/pejsam-website/pr-<PR-NUMBER>/
```

For example, PR #42 would be available at:
```
https://pesse.github.io/pejsam-website/pr-42/
```

### Workflow Lifecycle

1. **PR Opened/Updated**: When a PR is opened, synchronized (new commits pushed), or reopened:
   - The workflow builds a static ROQ site with PR-specific paths
   - Deploys to `gh-pages` branch in the `pr-<number>` subfolder
   - Posts/updates a comment on the PR with the preview URL

2. **PR Closed**: When a PR is closed (merged or not):
   - The cleanup workflow removes the PR-specific preview folder
   - Posts a cleanup notification comment

3. **Master Deployment**: When code is deployed to master:
   - The main deployment runs as usual
   - After deployment, a cleanup job removes preview folders for any closed PRs
   - This prevents accumulation of old preview folders

## Security & Access Control

### Trusted Contributors Only
Preview deployments only run for PRs from trusted contributors. Currently, this is restricted to:
- `pesse` (repository owner)

The restriction is enforced via workflow conditions:
```yaml
if: github.actor == 'pesse'
```

### Why This Restriction?
Running arbitrary code from untrusted PRs could:
- Consume excessive CI/CD resources
- Deploy malicious content to the Pages site
- Execute malicious build scripts

### Adding More Trusted Contributors
To add more trusted users, edit the workflow files and update the condition:
```yaml
# Single user
if: github.actor == 'pesse'

# Multiple users (OR condition)
if: github.actor == 'pesse' || github.actor == 'trusted-user'

# Alternative: Use labels
if: contains(github.event.pull_request.labels.*.name, 'preview-requested')
```

## Workflows

### 1. `pr-preview.yml` - Build and Deploy Preview
**Triggers:**
- `pull_request` events: `opened`, `synchronize`, `reopened`

**Jobs:**
- Checks out PR branch
- Sets up Java 21 and Maven
- Builds ROQ site with `QUARKUS_HTTP_ROOT_PATH=/pejsam-website/pr-<number>/`
- Deploys to `gh-pages` branch using `peaceiris/actions-gh-pages` action
- Posts preview URL comment on PR

### 2. `pr-preview-cleanup.yml` - Remove Preview on Close
**Triggers:**
- `pull_request` event: `closed`

**Jobs:**
- Checks out `gh-pages` branch
- Removes `pr-<number>` directory
- Commits and pushes the deletion
- Posts cleanup notification comment

### 3. `deploy.yml` - Main Deployment + Cleanup
**New Addition:**
- Added `cleanup-closed-prs` job that runs after main deployment
- Fetches list of open PRs via GitHub API
- Finds all `pr-*` directories in `gh-pages` branch
- Removes directories for closed PRs
- Commits and pushes cleanup

## Technical Details

### Path Handling
The key to making PR previews work is setting the correct base path for all generated links. This is done via the `QUARKUS_HTTP_ROOT_PATH` environment variable:

```bash
QUARKUS_HTTP_ROOT_PATH=/pejsam-website/pr-123/
```

This ensures all generated HTML links use the correct subfolder path, for example:
```html
<a href="/pejsam-website/pr-123/about">About</a>
```

### ROQ Build Configuration
The build uses the same configuration as the main deployment:
- Command: `./mvnw package quarkus:run`
- Environment: `QUARKUS_ROQ_GENERATOR_BATCH=true`
- Output: `target/roq/`

### GitHub Actions Used
- `actions/checkout@v4` - Check out code
- `actions/setup-java@v4` - Set up Java environment
- `peaceiris/actions-gh-pages@v4` - Deploy to gh-pages branch
- `actions/github-script@v7` - Run JavaScript to interact with GitHub API

## Permissions

The workflows require the following GitHub token permissions:

**Preview workflows:**
```yaml
permissions:
  contents: write        # Push to gh-pages branch
  pull-requests: write   # Comment on PRs
```

**Deploy workflow:**
```yaml
permissions:
  contents: write        # Push to gh-pages branch (for cleanup)
  pages: write           # Deploy to GitHub Pages
  id-token: write        # OIDC token for Pages deployment
```

## Troubleshooting

### Preview Not Building
- Check if the PR is from a trusted contributor
- Look at the Actions tab to see if the workflow ran
- Check for build errors in the workflow logs

### Preview Not Accessible
- Wait a few minutes after deployment completes for GitHub Pages to update
- Check if the `pr-<number>` folder exists in the `gh-pages` branch
- Verify the preview URL pattern matches: `https://pesse.github.io/pejsam-website/pr-<number>/`

### Old Previews Not Cleaned Up
- Check if the PR is actually closed (not just draft)
- Verify the cleanup workflow ran (check Actions tab)
- If cleanup failed, the next master deployment will try again

### Links Not Working in Preview
- All links must use `{site.url('path')}` in templates, never hardcoded paths
- Build must set `QUARKUS_HTTP_ROOT_PATH` correctly
- Check generated HTML to verify paths start with `/pejsam-website/pr-<number>/`

## Future Enhancements

Potential improvements for the future:

1. **Team-Based Access**: Use GitHub team membership instead of individual usernames
2. **Label-Based Triggers**: Require a `preview-requested` label for additional safety
3. **Preview Expiration**: Automatically remove previews older than X days
4. **Status Checks**: Report preview deployment status as a commit status check
5. **Environment Variables**: Store trusted users list in repository secrets

## Testing Locally

To test the build command locally with a PR-specific path:

```bash
QUARKUS_ROQ_GENERATOR_BATCH=true \
QUARKUS_HTTP_ROOT_PATH=/pejsam-website/pr-123/ \
./mvnw package quarkus:run -DskipTests
```

Then check the generated HTML files in `target/roq/` to verify paths are correct:

```bash
grep -o 'href="[^"]*"' target/roq/index.html | head -20
```

All links should start with `/pejsam-website/pr-123/`.
