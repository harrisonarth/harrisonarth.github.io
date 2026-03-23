+++
title = "Hosting a Website with Hugo & GitHub Pages"
date = 2026-03-23
tags = ['hugo', 'github', 'web']
+++

I built this website completely free, with no server to maintain. I only pay the small amount of money to own my own domain name. Everything is hosted as a repo on Github.

---

### What Is a Static Site?

A static site is a collection of plain HTML, CSS, and JavaScript files served directly to the browser. There's no database, no backend, and no server-side rendering. This makes them fast, cheap to host, and easy to secure.

### What Is Hugo?

[Hugo](https://gohugo.io) is a static site generator. You write content in Markdown, Hugo applies a theme and template layout, and it outputs a folder of ready-to-serve HTML files. It's fast, has no runtime dependencies, and is a single binary.

### What Is GitHub Pages?

[GitHub Pages](https://pages.github.com) is a free hosting service built into GitHub. Point it at a branch or folder in your repo, and GitHub will serve those files publicly at `<username>.github.io`. No cost, no infra.

---

### Prerequisites
- [Git](https://git-scm.com) installed locally
- A [GitHub](https://github.com) account
- [Hugo](https://gohugo.io/installation/) installed

### 1. Create a GitHub Repository

Create a new repo named exactly `<username>.github.io` (e.g. `harrisonarth.github.io`). This naming convention is required for GitHub Pages to serve it at your root domain.

### 2. Create a New Hugo Site

```bash
hugo new site my-site
cd my-site
git init
git remote add origin https://github.com/<username>/<username>.github.io.git
```

Add a theme. Hugo themes are typically added as Git submodules:

```bash
git submodule add https://github.com/<theme-repo>.git themes/<theme-name>
```

Set the theme in `hugo.toml`:

```toml
theme = "<theme-name>"
```

### 3. Add Content

```bash
hugo new posts/my-first-post.md
```

Edit the file in `content/posts/`. When ready to publish, set `draft = false` in the front matter (or remove the draft line entirely).

### 4. Configure GitHub Actions for Deployment

Hugo sites need to be built before they can be served. GitHub Actions automates this on every push.

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: latest
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 5. Enable GitHub Pages

In your repo on GitHub: **Settings → Pages → Source → GitHub Actions**. Push to `main` and the action will build and deploy your site automatically.

---

### Bonus: Using a Custom Domain

If you own a domain name already and want to use it instead of the default github.io domain.

#### 1. Add a CNAME File

Create a file named `CNAME` in your `static/` folder containing only your domain:

```
yourdomain.com
```

Hugo will copy this to the root of the built site, which GitHub Pages requires to recognize your custom domain.

Also update `baseURL` in `hugo.toml`:

```toml
baseURL = "https://yourdomain.com/"
```

#### 2. Configure DNS

At your domain registrar, add the following records:

| Type | Name | Value |
|------|------|-------|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | `<username>.github.io` |

These IPs are GitHub's Pages servers. DNS changes can take up to 24 hours to propagate.

#### 3. Set the Custom Domain in GitHub

In **Settings → Pages → Custom domain**, enter your domain and save. GitHub will automatically provision a free TLS certificate via Let's Encrypt once DNS propagates. Enable **Enforce HTTPS** once the cert is issued.
