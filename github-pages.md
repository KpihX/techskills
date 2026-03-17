# 🌐 Publish a Markdown Repo with Docsify + GitHub Pages

> **Machine:** KpihX-Ubuntu (Ubuntu 24.04 LTS)
> **Lived on:** 2026-03 · **Status:** Production-stable (this very site)

---

## 🧩 Context & Problem

You have a folder of Markdown files — field notes, tutorials, docs — and want to publish them as a browseable website **without a build step**.

```
Problem:  Most static site generators (Hugo, Jekyll, MkDocs) require:
          - A build step
          - A CI/CD pipeline
          - Knowledge of their templating system

Goal:     Push Markdown → GitHub Pages serves it instantly.
          No build. No pipeline. No Node modules committed.
```

**Solution: Docsify** — a single `index.html` loads your `.md` files client-side via JavaScript. GitHub Pages serves the raw files, Docsify renders them in the browser.

---

## 🏗️ Architecture & Concepts

```
Your repo (tutos_live/)
│
├── index.html       ← Docsify bootstrap (just a CDN script tag)
├── _sidebar.md      ← Navigation structure
├── README.md        ← Site homepage (loaded as /README)
└── *.md             ← All tutorials, loaded on demand

        ↓  git push

GitHub Pages
└── Serves files as-is from branch root
    No build, no processing

        ↓  browser visits https://kpihx.github.io/tutos_live/

Docsify (CDN JS)
└── Fetches README.md → renders HTML in-browser
    Fetches _sidebar.md → builds navigation
    User clicks link → fetches tailscale.md → renders
```

Zero build. Zero CI. The CDN JS does all the work client-side.

---

## 🔧 Setup

### 1. Create `index.html`

This is the only non-Markdown file you need. It tells GitHub Pages "this is a Docsify site" and loads everything from CDN.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Your Site Title</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/style.min.css"
        title="docsify-darklight-theme" type="text/css" />
</head>
<body>
  <div id="app">Loading...</div>
  <script>
    window.$docsify = {
      name: '🐧 Your Site Name',
      repo: 'https://github.com/yourname/yourrepo',
      loadSidebar: true,
      subMaxLevel: 2,
      auto2top: true,
      themeColor: '#007acc',
      search: {
        maxAge: 86400000,
        paths: 'auto',
        placeholder: '🔍 Search...',
        noData: '❌ No results.',
        depth: 6
      }
    }
  </script>
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/index.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-bash.min.js"></script>
</body>
</html>
```

> **Dark theme** (VS Code-style): the `docsify-darklight-theme` plugin adds a toggle button. `defaultTheme: 'dark'` sets the initial state. Full config in this repo's `index.html`.

---

### 2. Create `_sidebar.md`

Docsify reads this file to build the left-side navigation:

```markdown
- **My Site**

- **Tutorials**
  - [🏠 Home](README.md)
  - [🔒 Tutorial 1](tutorial-1.md)
  - [🌐 Tutorial 2](tutorial-2.md)

- **Links**
  - [GitHub](https://github.com/yourname/yourrepo)
```

Rules:
- Indented items become sub-entries
- Links are relative paths to `.md` files
- Update this file every time you add a new tutorial

---

### 3. Preview Locally

No Node modules needed globally — use `npx`:

```bash
cd ~/Work/tutos_live

# Serve on http://localhost:3000
npx docsify-cli serve .
```

```
Serving /home/kpihx/Work/tutos_live now.
Listening at http://localhost:3000
```

Open `http://localhost:3000` in your browser. Changes to `.md` files are reflected on reload — no rebuild needed.

> **One-shot, no install:** `npx` downloads `docsify-cli` on first run and caches it. Never commits to your repo.

---

### 4. Create the GitHub Repo & Push

Two approaches — same result, different tools:

```
Method A — Classic (git + github.com web UI)
  git init → create repo on github.com → git remote add → git push

Method B — gh CLI (everything from the terminal, zero browser)
  gh repo create → git remote add → git push
```

---

#### Method A — Classic: git + github.com

```bash
cd ~/Work/tutos_live

# 1. Commit your files
git add .
git commit -m "feat: initial Docsify setup"
```

Then go to **https://github.com/new** in your browser:
- Repository name: `tutos_live`
- Visibility: Public
- **Do NOT** initialize with README (you already have one)
- Click **Create repository**

GitHub shows you the remote URL. Copy the SSH one:

```bash
# 2. Add remote (SSH — always)
git remote add github git@github.com:kpihx/tutos_live.git

# 3. Push
git push -u github master
```

---

#### Method B — gh CLI (terminal only)

```bash
cd ~/Work/tutos_live

# 1. Commit your files
git add .
git commit -m "feat: initial Docsify setup"

# 2. Create repo on GitHub AND get the SSH remote URL
gh repo create kpihx/tutos_live \
  --public \
  --description "Ubuntu-focused knowledge base"

# 3. Add remote manually (gh create does NOT add it automatically)
git remote add github git@github.com:kpihx/tutos_live.git

# 4. Push
git push github HEAD
```

> **Tip:** `gh repo create` accepts `--clone` to clone immediately, but since you're working from an existing local repo, adding the remote manually is cleaner.

---

### 5. Enable GitHub Pages

Two methods here as well:

```
Method A — Web UI:  github.com Settings → Pages → point to branch
Method B — gh API:  one command, zero browser
```

---

#### Method A — Web UI

1. Go to `https://github.com/kpihx/tutos_live`
2. **Settings** → **Pages** (left sidebar)
3. Under **Build and deployment**:
   - Source: **Deploy from a branch**
   - Branch: `master` · Folder: `/ (root)`
4. Click **Save**

---

#### Method B — gh api (terminal only, instant)

```bash
gh api repos/kpihx/tutos_live/pages \
  --method POST \
  -f "source[branch]=master" \
  -f "source[path]=/"
```

Response confirms activation:
```json
{
  "html_url": "https://kpihx.github.io/tutos_live/",
  "source": { "branch": "master", "path": "/" },
  "public": true
}
```

Check status later:
```bash
gh api repos/kpihx/tutos_live/pages
```

Change branch or path after the fact:
```bash
gh api repos/kpihx/tutos_live/pages \
  --method PUT \
  -f "source[branch]=main" \
  -f "source[path]=/"
```

After ~1 minute, site is live:

```
https://kpihx.github.io/tutos_live/
```

```
GitHub Pages
└── Branch: master, root /
    ├── index.html   → served as entry point
    ├── README.md    → Docsify loads as homepage
    └── *.md         → loaded on demand by Docsify JS
```

> **No `gh-pages` branch needed.** Serving from `master` root is the simplest setup for a docs-only repo.
> **For more on `gh`**, see the dedicated tutorial → [gh.md](gh.md)

---

## 🐛 Debugging

### Site shows raw Markdown instead of rendered HTML

`index.html` is missing or has a syntax error. Verify it exists at repo root and that the `<script>` tag for docsify CDN is present.

### Sidebar not showing

`_sidebar.md` must be at repo root AND `loadSidebar: true` must be set in `window.$docsify`.

### Local preview: port 3000 already in use

```bash
npx docsify-cli serve . --port 3001
```

### GitHub Pages shows 404

- Check that Pages is enabled (Settings → Pages)
- Verify branch is `main` and folder is `/` (root)
- Wait 1-2 minutes after first push — first deploy takes time

### Search not working locally

Browser security blocks XHR from `file://` — always use `npx docsify-cli serve .` for local preview, never open `index.html` directly.

---

## ✅ Verification

```bash
# Local
npx docsify-cli serve .
# → http://localhost:3000  — navigate, search, theme toggle

# Remote (after GitHub Pages enabled)
curl -s -o /dev/null -w "%{http_code}" https://kpihx.github.io/tutos_live/
# → 200
```

---

## 📚 References

- [Docsify documentation](https://docsify.js.org)
- [docsify-darklight-theme](https://github.com/boopathikumar018/docsify-darklight-theme)
- [GitHub Pages documentation](https://docs.github.com/en/pages)
