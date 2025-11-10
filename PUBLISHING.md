# Publishing Guide

This guide explains how to publish the Sen Protocol documentation to GitHub and GitBook.

## 1. Create GitHub Repository

### Using GitHub CLI

```bash
cd /Users/vuln/code/sen-fi/gitbook

# Create repository in sen-fi org
gh repo create sen-fi/gitbook --public --source=. --remote=origin --push

# Or if you prefer to do it manually:
# gh repo create sen-fi/gitbook --public
# git remote add origin git@github.com:sen-fi/gitbook.git
# git branch -M main
# git push -u origin main
```

### Using GitHub Web UI

1. Go to https://github.com/organizations/sen-fi/repositories/new
2. Repository name: `gitbook`
3. Description: "Sen Protocol Documentation - Privacy-first agent-to-agent payment protocol"
4. Visibility: **Public**
5. Click "Create repository"

Then push your code:

```bash
cd /Users/vuln/code/sen-fi/gitbook
git remote add origin git@github.com:sen-fi/gitbook.git
git branch -M main
git push -u origin main
```

## 2. Set Up GitBook

### Option A: GitBook Cloud (Recommended)

1. Go to https://www.gitbook.com/
2. Sign in with GitHub account
3. Click "New Space"
4. Choose "Import from GitHub"
5. Select `sen-fi/gitbook` repository
6. Choose `main` branch
7. Set root directory: `/` (or leave default)
8. Click "Import"

GitBook will automatically:
- Read your `SUMMARY.md` for structure
- Use `README.md` as landing page
- Apply settings from `.gitbook.yaml`
- Auto-sync on push to `main`

### Option B: Self-Hosted GitBook

```bash
# Install GitBook CLI
npm install -g gitbook-cli

# Install plugins
cd /Users/vuln/code/sen-fi/gitbook
gitbook install

# Build static site
gitbook build

# Output is in _book/ directory
# Deploy to any static hosting (Netlify, Vercel, GitHub Pages, etc.)
```

### Custom Domain Setup (docs.sen.fi)

**On GitBook**:
1. Go to Space Settings → Domain
2. Add custom domain: `docs.sen.fi`
3. Note the CNAME target (e.g., `hosting.gitbook.io`)

**DNS Configuration**:
```
Type: CNAME
Name: docs
Value: hosting.gitbook.io
TTL: 3600
```

**SSL**:
GitBook automatically provisions SSL certificates via Let's Encrypt.

## 3. GitHub Actions for Auto-Deploy

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Documentation

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install GitBook
        run: npm install -g gitbook-cli

      - name: Install plugins
        run: gitbook install

      - name: Build book
        run: gitbook build

      - name: Deploy to GitHub Pages
        if: github.ref == 'refs/heads/main'
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_book
          cname: docs.sen.fi  # Optional: for custom domain
```

## 4. Configure GitHub Pages

1. Go to repository Settings → Pages
2. Source: Deploy from branch `gh-pages`
3. Folder: `/ (root)`
4. Custom domain: `docs.sen.fi` (optional)
5. Enforce HTTPS: ✅

## 5. Keep Documentation Updated

### Editing Workflow

```bash
# 1. Create feature branch
cd /Users/vuln/code/sen-fi/gitbook
git checkout -b docs/update-privacy-section

# 2. Make changes
# Edit markdown files...

# 3. Test locally
gitbook serve
# Preview at http://localhost:4000

# 4. Commit and push
git add .
git commit -m "docs: update privacy architecture section"
git push origin docs/update-privacy-section

# 5. Create pull request
gh pr create --title "docs: update privacy architecture section" \
  --body "Updated privacy guarantees and added TEE attestation details"

# 6. After review, merge to main
gh pr merge
```

### Syncing with Codebase

To keep documentation in sync with code changes:

```bash
# Option 1: Keep docs in main repo (symlink)
cd /Users/vuln/code/sen/sen-app
ln -s /Users/vuln/code/sen-fi/gitbook docs/gitbook

# Option 2: Git submodule
cd /Users/vuln/code/sen/sen-app
git submodule add git@github.com:sen-fi/gitbook.git docs/gitbook

# Option 3: Automated sync script
# Create .github/workflows/sync-docs.yml to copy relevant docs
```

## 6. Version Management

### Tagging Releases

```bash
# Tag documentation versions matching protocol releases
git tag -a v1.0.0 -m "Documentation for Sen Protocol v1.0.0"
git push origin v1.0.0
```

### GitBook Versioning

1. In GitBook UI: Settings → Versions
2. Create version: `v1.0`
3. Set as default or archived
4. Users can switch versions in docs

## 7. Analytics & Monitoring

### GitBook Analytics

GitBook provides built-in analytics:
- Page views
- Search queries
- Popular pages
- User engagement

### Custom Analytics

Add to `.gitbook.yaml`:

```yaml
plugins:
  - ga  # Google Analytics

pluginsConfig:
  ga:
    token: "UA-XXXXXXXXX-X"
```

Or add to `book.json`:

```json
{
  "plugins": ["ga"],
  "pluginsConfig": {
    "ga": {
      "token": "UA-XXXXXXXXX-X"
    }
  }
}
```

## 8. Maintenance

### Regular Updates

- **Weekly**: Check for broken links, outdated code examples
- **Monthly**: Review and update roadmap, FAQs
- **Quarterly**: Major version updates, restructuring

### Link Checking

```bash
# Install link checker
npm install -g markdown-link-check

# Check all markdown files
find . -name "*.md" -exec markdown-link-check {} \;
```

### Spell Checking

```bash
# Install spell checker
npm install -g cspell

# Check spelling
cspell "**/*.md"
```

## 9. Troubleshooting

### GitBook not syncing

1. Check webhook in GitHub: Settings → Webhooks
2. Verify `.gitbook.yaml` is valid
3. Check GitBook integration: Settings → Integrations
4. Manual sync: GitBook UI → "Sync with GitHub"

### Build failures

```bash
# Clear cache
rm -rf _book node_modules
gitbook install
gitbook build
```

### Plugin issues

```bash
# Update plugins
gitbook install --force
```

## 10. Resources

- **GitBook Documentation**: https://docs.gitbook.com/
- **GitHub Pages**: https://docs.github.com/en/pages
- **Markdown Guide**: https://www.markdownguide.org/
- **GitBook Plugins**: https://plugins.gitbook.com/

---

## Quick Reference

### Common Commands

```bash
# Serve locally
gitbook serve

# Build static site
gitbook build

# Install/update plugins
gitbook install

# View in browser
open http://localhost:4000

# Push to GitHub
git push origin main

# Create new version tag
git tag -a v1.1.0 -m "Version 1.1.0"
git push origin v1.1.0
```

### File Structure

```
.
├── .gitbook.yaml      # GitBook config
├── book.json          # Plugin config
├── README.md          # Landing page
├── SUMMARY.md         # Table of contents (IMPORTANT!)
├── architecture/      # Architecture docs
├── protocol/          # Protocol specs
├── marketplace/       # Marketplace docs
├── api/              # API reference
└── resources/        # Glossary, FAQ, etc.
```

### Important Notes

- `SUMMARY.md` defines the structure - keep it updated!
- Images should be in `/images` or relative paths
- Code blocks should specify language for syntax highlighting
- Internal links use relative paths: `[Link](../other-section/page.md)`
- External links use full URLs: `[Link](https://example.com)`
