# 🌐 Publish a Markdown Repo with Docsify + GitHub Pages

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Production-stable (this very site)

---

## 🧩 Context & Problem

You have a folder of Markdown files — field notes, tutorials, docs — and want to
publish them as a browseable website **without a build step**.

```
Problem:  Most static site generators (Hugo, Jekyll, MkDocs) require:
          ┌─────────────────────────────────────────┐
          │  build step → CI/CD pipeline → deploy   │
          │  + templating language to learn          │
          │  + dependencies to manage               │
          └─────────────────────────────────────────┘

Goal:     Push Markdown → GitHub Pages serves it instantly.
          ┌──────────────────────────────────────────┐
          │  git push → live site. That's it.        │
          │  No build. No pipeline. No npm in repo.  │
          └──────────────────────────────────────────┘
```

**Solution: Docsify** — a single `index.html` loads your `.md` files
client-side via JavaScript. GitHub Pages serves the raw files; Docsify
renders them in the browser.

---

## 🏗️ Architecture & Concepts

### End-to-end flow

```
 Local machine                GitHub                    Browser
 ─────────────               ────────               ─────────────────
  tutos_live/
  ├── .nojekyll    git push   GitHub Pages           https://kpihx.
  ├── index.html ──────────▶  serves files  ──────▶  github.io/
  ├── _sidebar.md             as-is (raw)            tutos_live/
  ├── README.md                    │                      │
  └── *.md                         │              Docsify (CDN JS)
                                   │              fetches .md files
                                   │              renders HTML
                                   │              client-side
                                   ▼                      ▼
                             No build step          Full site ✅
```

### The Jekyll trap (why `.nojekyll` is mandatory)

GitHub Pages runs **Jekyll by default** on every push. Jekyll is a static
site generator with one silent but critical rule:

```
Jekyll convention:
  Any file or folder whose name starts with _ is IGNORED.

                   ┌────────────────────────────────┐
  Without          │  _sidebar.md  ← IGNORED ❌     │
  .nojekyll:       │  _navbar.md   ← IGNORED ❌     │
                   │  index.html   → served ✅       │
                   └────────────────────────────────┘
  Result: Docsify starts, tries to fetch _sidebar.md
          → 404 → no sidebar, no navigation.

                   ┌────────────────────────────────┐
  With             │  .nojekyll present              │
  .nojekyll:       │  → Jekyll is DISABLED entirely  │
                   │  → ALL files served as-is ✅    │
                   └────────────────────────────────┘
  Result: Docsify fetches _sidebar.md → sidebar works.
```

> **`.nojekyll` is a zero-byte file.** Its presence alone disables Jekyll.
> No content needed — just the file.

---

## 🔧 Setup

### Step 1 — Create `index.html`

The Docsify bootstrap. One file, no build, everything loaded from CDN.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Your Site Title</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
  <!-- Dark/light theme CSS — title attribute is required for theme switching -->
  <link rel="stylesheet"
        href="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/style.min.css"
        title="docsify-darklight-theme"
        type="text/css" />
  <style>
    /* VS Code sidebar layout */
    .sidebar { display: flex !important; flex-direction: column !important; }
    .app-name { order: 1 !important; margin-top: 20px !important; font-weight: bold !important; }
    .search   { order: 2 !important; margin-bottom: 20px !important; }
    .sidebar-nav { order: 3 !important; flex: 1 !important; }
    /* Theme toggle button */
    #docsify-darklight-theme { top: 15px !important; right: 15px !important;
                               width: 28px !important; height: 28px !important; }
  </style>
</head>
<body>
  <div id="app">Loading...</div>
  <script>
    window.$docsify = {
      name: '🐧 Your Site Name',
      repo: 'https://github.com/yourname/yourrepo',
      loadSidebar: true,      // reads _sidebar.md for navigation
      subMaxLevel: 2,
      auto2top: true,
      themeColor: '#007acc',  // VS Code blue accent
      alias: { '/.*/_sidebar.md': '/_sidebar.md' },  // single sidebar for all pages
      search: {
        maxAge: 86400000,
        paths: 'auto',
        placeholder: '🔍 Search...',
        noData: '❌ No results.',
        depth: 6
      },
      darklightTheme: {
        defaultTheme: 'dark',   // start in dark mode
        dark: {
          accent: '#007acc',
          background: '#1e1e1e',
          sidebarBackground: '#252526',
          textColor: '#d4d4d4'
        },
        light: {
          accent: '#007acc',
          background: '#ffffff',
          textColor: '#333333'
        }
      }
    }
  </script>
  <!-- Load order matters: docsify core first, then plugins -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/index.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-bash.min.js"></script>
</body>
</html>
```

**Script load order:**

```
  docsify@4 (core)
      │
      ▼
  docsify-darklight-theme  ← registers theme plugin into docsify
      │
      ▼
  search.min.js            ← registers search plugin
      │
      ▼
  prism-bash.min.js        ← syntax highlighting for bash blocks
```

> **`title` on the `<link>` tag is not optional.** The darklight theme uses it
> to identify and swap the stylesheet at runtime. Remove it and the toggle breaks.

---

### Step 2 — Create `_sidebar.md`

Docsify fetches this file to build the left navigation panel:

```markdown
- **My Site**

- **Tutorials**
  - [🏠 Home](README.md)
  - [🔒 Tutorial 1](tutorial-1.md)
  - [🌐 Tutorial 2](tutorial-2.md)

- **Links**
  - [GitHub](https://github.com/yourname/yourrepo)
```

```
_sidebar.md structure → rendered panel
─────────────────────   ───────────────
- **Section**           ■ Section
  - [Label](file.md)     └─ Label        ← clickable
  - [Label](file.md)     └─ Label
```

Update this file every time you add a new tutorial.

---

### Step 3 — Create `.nojekyll`  ⚠️ Critical

```bash
# Zero-byte file — content doesn't matter, presence does
touch .nojekyll
```

```
repo root/
├── .nojekyll    ← tells GitHub Pages: skip Jekyll, serve everything as-is
├── index.html
├── _sidebar.md  ← will be served (Jekyll would have dropped it)
└── README.md
```

**Without this file: sidebar is invisible.** Jekyll runs, silently drops
`_sidebar.md`, Docsify gets a 404 when it tries to fetch it.

---

### Step 4 — Preview Locally

```bash
cd ~/Work/tutos_live

# One-shot, no global install needed
npx docsify-cli serve .
```

```
  Serving /home/kpihx/Work/tutos_live now.
  Listening at http://localhost:3000
                        │
              open in browser
                        │
          ┌─────────────▼──────────────┐
          │  sidebar ✅  │  content     │
          │  _sidebar.md │  README.md  │
          │  (rendered)  │  (rendered) │
          └─────────────────────────── ┘
```

> Changes to `.md` files are reflected on browser reload — no rebuild.
> `npx` caches `docsify-cli` on first run. Never add it to your repo.

---

### Step 5 — Create the GitHub Repo & Push

Two approaches — same result, different tools:

```
  ┌──────────────────────────────────────────────────────┐
  │  Method A — Classic                                  │
  │  git init → github.com/new (browser) →              │
  │  git remote add → git push                          │
  ├──────────────────────────────────────────────────────┤
  │  Method B — gh CLI (terminal only, zero browser)     │
  │  gh repo create → git remote add → git push         │
  └──────────────────────────────────────────────────────┘
```

#### Method A — Classic: git + github.com

```bash
# 1. Stage and commit everything (including .nojekyll!)
git add .
git commit -m "feat: initial Docsify setup"
```

Open **https://github.com/new** in your browser:
- Repository name: `tutos_live`
- Visibility: Public
- ❌ Do NOT check "Initialize this repository" (you already have files)
- Click **Create repository**

GitHub shows the remote URL — copy the **SSH** one:

```bash
# 2. Wire the remote
git remote add github git@github.com:yourname/tutos_live.git

# 3. Push
git push -u github master
```

#### Method B — gh CLI (terminal only)

```bash
# 1. Stage and commit everything (including .nojekyll!)
git add .
git commit -m "feat: initial Docsify setup"

# 2. Create the repo on GitHub
gh repo create yourname/tutos_live \
  --public \
  --description "My Ubuntu live tutorials"

# 3. Add remote (gh create does NOT add it automatically)
git remote add github git@github.com:yourname/tutos_live.git

# 4. Push
git push github HEAD
```

---

### Step 6 — Enable GitHub Pages

```
  ┌──────────────────────────────────────────────────────┐
  │  Method A — Web UI                                   │
  │  Settings → Pages → pick branch → Save              │
  ├──────────────────────────────────────────────────────┤
  │  Method B — gh api (one command, zero browser)       │
  │  gh api repos/.../pages --method POST ...           │
  └──────────────────────────────────────────────────────┘
```

#### Method A — Web UI

1. Go to your repo: `https://github.com/yourname/tutos_live`
2. **Settings** → **Pages** (left sidebar)
3. Under **Build and deployment**:
   - Source: `Deploy from a branch`
   - Branch: `master` · Folder: `/ (root)`
4. Click **Save**

#### Method B — gh api (instant, no browser)

```bash
gh api repos/yourname/tutos_live/pages \
  --method POST \
  -f "source[branch]=master" \
  -f "source[path]=/"
```

Response confirms activation:
```json
{
  "html_url": "https://yourname.github.io/tutos_live/",
  "source": { "branch": "master", "path": "/" },
  "public": true
}
```

Check status / update source later:
```bash
# Status
gh api repos/yourname/tutos_live/pages

# Change branch
gh api repos/yourname/tutos_live/pages \
  --method PUT \
  -f "source[branch]=main" \
  -f "source[path]=/"
```

After ~1 minute, site is live:

```
  git push
     │
     ▼
  GitHub Pages  (Jekyll disabled by .nojekyll)
  ├── index.html     → entry point
  ├── .nojekyll      → disables Jekyll
  ├── _sidebar.md    → served ✅ (not ignored)
  └── *.md           → served ✅
     │
     ▼
  https://yourname.github.io/tutos_live/
  ├── sidebar visible ✅
  ├── dark/light toggle ✅
  └── search working ✅
```

> **For more on `gh`**, see → [gh.md](gh.md)

---

## 🐛 Debugging

### Sidebar not showing

**Most likely cause: `.nojekyll` missing.**

```bash
# Check it exists
ls -la .nojekyll

# Create if missing
touch .nojekyll
git add .nojekyll
git commit -m "fix: add .nojekyll to disable Jekyll"
git push github HEAD
```

Wait ~1 min after push, then hard-refresh the browser (`Ctrl+Shift+R`).

Secondary cause: `loadSidebar: true` missing from `window.$docsify`.

### Theme toggle is a blank square / not working

The `title` attribute on the CSS `<link>` tag is missing or wrong:

```html
<!-- ✅ Correct — title must match exactly -->
<link rel="stylesheet"
      href="...docsify-darklight-theme.../style.min.css"
      title="docsify-darklight-theme"
      type="text/css" />

<!-- ❌ Wrong — no title → plugin can't find the stylesheet to swap -->
<link rel="stylesheet" href="...docsify-darklight-theme.../style.min.css" />
```

Also verify load order: `docsify@4` must load **before** the theme plugin JS.

### Site shows raw Markdown

`index.html` is missing, or the Docsify CDN `<script>` tag has a typo.
Check the browser console (F12) for 404s.

### Local preview: port 3000 already in use

```bash
npx docsify-cli serve . --port 3001
```

### Search not working locally

Browser security blocks XHR from `file://`. Always use
`npx docsify-cli serve .` — never open `index.html` directly in the browser.

### GitHub Pages shows 404 after enabling

- Wait 1-2 minutes — first deploy takes time
- Verify the branch and path in Settings → Pages
- Check that `index.html` is at repo root (not in a subfolder)

---

## ✅ Verification

```bash
# 1. Local — full feature check
npx docsify-cli serve .
# → sidebar visible, dark toggle works, search works

# 2. Remote — HTTP 200
curl -s -o /dev/null -w "%{http_code}" https://yourname.github.io/tutos_live/
# → 200

# 3. _sidebar.md reachable (Jekyll check)
curl -s -o /dev/null -w "%{http_code}" \
  https://yourname.github.io/tutos_live/_sidebar.md
# → 200  (would be 404 without .nojekyll)
```

---

## 📚 References

- [Docsify documentation](https://docsify.js.org)
- [docsify-darklight-theme](https://github.com/boopathikumar018/docsify-darklight-theme)
- [GitHub Pages documentation](https://docs.github.com/en/pages)
- [GitHub CLI (`gh`) full guide](gh.md)
