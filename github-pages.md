# 🌐 Publishing Markdown Notes as a Website — Docsify + GitHub Pages

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Production-stable (this very site)

---

I had a pile of Markdown files — Ubuntu fixes, homelab notes, things I kept
looking up. I wanted to share them as a browseable site without setting up
a build system, a CI pipeline, or committing node_modules to a repo.

After looking at Hugo, Jekyll, and MkDocs, I found **Docsify**. The pitch:
drop one `index.html` in your repo, and it reads your Markdown files on the
fly in the browser. No build. No generated files. No framework to learn.
Just push and it's live.

I tried it. It worked. Then I pushed to GitHub Pages, and the sidebar disappeared.

Here's the full story.

---

## How Docsify works

Docsify isn't a static site generator — it's a client-side runtime. The
browser fetches your `.md` files directly, and a small JavaScript library
renders them as HTML in real time.

```
Traditional static site generator:
  source .md files
       │  build step (Hugo/Jekyll/MkDocs)
       ▼
  generated HTML ──► CI/CD ──► host

Docsify:
  source .md files
       │  git push (that's it)
       ▼
  raw files on GitHub Pages
       │  browser fetches index.html
       │  Docsify JS fetches *.md on demand
       ▼
  rendered in browser ✅
```

The entire "build system" is one `<script>` tag pointing at the Docsify CDN.

---

## Setting it up — the three files

You need exactly three things in your repo root: `index.html`, `_sidebar.md`,
and `.nojekyll`. Let me walk through each.

### `index.html` — the Docsify bootstrap

This is the only non-Markdown file. It loads the Docsify runtime from CDN
and configures the site — theme, search, sidebar, dark/light toggle.

> **Template available:** [`templates/github-pages-index.html`](https://github.com/kpihx/tutos_live/blob/master/templates/github-pages-index.html)
> Copy it to your repo root, rename to `index.html`, replace the 4 placeholders
> (`SITE_TITLE`, `SITE_NAME`, `GITHUB_REPO_URL`, `SEARCH_PLACEHOLDER`).
> The full annotated version is below for reference.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Your Site Title</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">

  <!-- Theme CSS — the title attribute is required for theme switching to work -->
  <link rel="stylesheet"
        href="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/style.min.css"
        title="docsify-darklight-theme"
        type="text/css" />

  <style>
    /* VS Code sidebar layout */
    .sidebar      { display: flex !important; flex-direction: column !important; }
    .app-name     { order: 1 !important; margin-top: 20px !important; font-weight: bold !important; }
    .search       { order: 2 !important; margin-bottom: 20px !important; }
    .sidebar-nav  { order: 3 !important; flex: 1 !important; }

    /* Theme toggle button */
    #docsify-darklight-theme {
      top: 15px !important; right: 15px !important;
      width: 28px !important; height: 28px !important;
    }

    /* Inline code readability — override the default blue codeTypeColor */
    p code, li code, td code, blockquote code {
      background-color: var(--codeBackgroundColor) !important;
      color: var(--codeTextColor) !important;
      padding: 2px 6px !important;
      border-radius: 3px !important;
    }
  </style>
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
      alias: { '/.*/_sidebar.md': '/_sidebar.md' },
      search: {
        maxAge: 86400000,
        paths: 'auto',
        placeholder: '🔍 Search...',
        noData: '❌ No results.',
        depth: 6
      },
      darklightTheme: {
        defaultTheme: 'dark',
        dark: {
          background: '#1e1e1e',
          sidebarBackground: '#252526',
          textColor: '#d4d4d4',
          codeTextColor: '#ce9178',       /* readable orange on dark */
          codeBackgroundColor: '#2d2d2d',
          codeTypeColor: '#ce9178'        /* match codeTextColor — avoid blue override */
        },
        light: {
          background: '#ffffff',
          sidebarBackground: '#f3f3f3',
          textColor: '#333333',
          codeTextColor: '#c7254e',       /* readable dark red on light */
          codeBackgroundColor: '#f0f0f0', /* visible contrast on white page */
          codeTypeColor: '#c7254e'        /* match codeTextColor — avoid blue override */
        }
      }
    }
  </script>
  <!-- Load order matters: docsify core, then plugins -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/index.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-bash.min.js"></script>
</body>
</html>
```

Two things worth noting here.

The `title` attribute on the `<link>` tag is not optional. The darklight theme
plugin locates the stylesheet by that exact string to swap it at runtime. Remove
it, and the theme toggle button renders as an empty square and does nothing.

And `codeTypeColor` — the darklight theme uses this for inline `<code>` elements.
The default is `#007acc` (VS Code blue), which looks fine in a code editor but
is hard to read at body-text size, especially on a white background. Setting it
to the same value as `codeTextColor` keeps things consistent and readable.

### `_sidebar.md` — the navigation

Docsify reads this file to build the left panel. The format is simple Markdown:

```markdown
- **My Site**

- **Tutorials**
  - [🏠 Home](README.md)
  - [🔒 Tutorial 1](tutorial-1.md)
  - [🌐 Tutorial 2](tutorial-2.md)

- **Links**
  - [GitHub](https://github.com/yourname/yourrepo)
```

Each `- [Label](file.md)` becomes a clickable entry. Indented items are
sub-entries. Update this file every time you add a tutorial.

### `.nojekyll` — the most important file you didn't know you needed

I pushed everything and visited my GitHub Pages URL. The content loaded.
The dark theme worked. But the sidebar was completely gone.

After some digging I found the cause: **GitHub Pages runs Jekyll by default**,
and Jekyll silently drops any file or directory whose name starts with `_`.

```
GitHub Pages default behavior (Jekyll enabled):
  ├── index.html     → served ✅
  ├── README.md      → served ✅
  ├── tailscale.md   → served ✅
  └── _sidebar.md    → IGNORED ❌  (Jekyll convention: _ prefix = private)

  Docsify tries to fetch _sidebar.md → 404 → no navigation panel.
```

The fix is a single empty file at the repo root:

```bash
touch .nojekyll
```

```
With .nojekyll present:
  GitHub Pages sees it → disables Jekyll entirely → serves all files as-is
  ├── index.html     → served ✅
  ├── _sidebar.md    → served ✅  (Jekyll is gone, can't touch it)
  └── *.md           → served ✅
```

`.nojekyll` has no content — its presence is the signal. Commit it, push it,
and the sidebar appears.

---

## Previewing locally before pushing

Before pushing, I always verify the site locally. No server needed — just `npx`:

```bash
cd ~/Work/tutos_live

npx docsify-cli serve .
# → Serving at http://localhost:3000
```

`npx` downloads `docsify-cli` on first run, caches it, and never touches
your repo. Open `http://localhost:3000`, navigate the sidebar, test search,
toggle the theme.

> One gotcha: don't open `index.html` directly in the browser via `file://`.
> Browser security blocks the XHR requests Docsify uses to fetch `.md` files.
> Always go through the local server.

---

## Creating the GitHub repo — two ways

Once everything works locally, time to put it on GitHub.

```
  ┌──────────────────────────────────────────────────────┐
  │  Method A — git + github.com                         │
  │  You open a browser, create the repo through the UI, │
  │  then add the remote and push from the terminal.     │
  ├──────────────────────────────────────────────────────┤
  │  Method B — gh CLI                                   │
  │  Everything from the terminal. No browser needed.    │
  └──────────────────────────────────────────────────────┘
```

### Method A — git + github.com

```bash
# Stage and commit — including .nojekyll
git add .
git commit -m "feat: initial Docsify setup"
```

Go to **https://github.com/new**. Fill in the name, choose Public, and —
importantly — **don't** check "Initialize this repository". You already have
files. GitHub will show you a remote URL; grab the SSH one.

```bash
git remote add github git@github.com:yourname/tutos_live.git
git push -u github master
```

### Method B — gh CLI (terminal only)

```bash
# Stage and commit — including .nojekyll
git add .
git commit -m "feat: initial Docsify setup"

# Create the repo on GitHub
gh repo create yourname/tutos_live \
  --public \
  --description "My Ubuntu live tutorials"

# Add the remote (gh create doesn't add it automatically)
git remote add github git@github.com:yourname/tutos_live.git

# Push
git push github HEAD
```

> For a full `gh` CLI guide — repos, PRs, issues, releases — see [gh.md](gh.md).

---

## Enabling GitHub Pages — two ways

After pushing, Pages needs to be activated once. Again, two options:

### Method A — web UI

Go to your repo → **Settings** → **Pages**.
Under *Build and deployment*: Source = `Deploy from a branch`,
Branch = `master`, Folder = `/ (root)`. Save.

### Method B — gh api (instant, no browser)

```bash
gh api repos/yourname/tutos_live/pages \
  --method POST \
  -f "source[branch]=master" \
  -f "source[path]=/"
```

The response includes `html_url` — your site's URL. After about a minute,
it's live.

```
git push
  │
  ▼
GitHub Pages (Jekyll disabled by .nojekyll)
  ├── index.html   → entry point
  ├── _sidebar.md  → served ✅ (not dropped)
  └── *.md         → fetched on demand by Docsify
  │
  ▼
https://yourname.github.io/tutos_live/
  → sidebar ✅ · dark toggle ✅ · search ✅
```

---

## 🐛 Debugging

### Sidebar not showing

Almost certainly `.nojekyll` is missing. Check:

```bash
ls -la .nojekyll
```

If absent:
```bash
touch .nojekyll
git add .nojekyll && git commit -m "fix: add .nojekyll" && git push github HEAD
```

Wait ~1 min, then hard-refresh (`Ctrl+Shift+R`).

If `.nojekyll` exists, check that `loadSidebar: true` is in `window.$docsify`
and that `_sidebar.md` is at repo root.

### Theme toggle is an empty square

The `title="docsify-darklight-theme"` attribute is missing from the `<link>` tag.
The plugin locates the stylesheet by that exact string.

### Search not working

Only works through the local server. Open `http://localhost:3000`, not `file://`.

### GitHub Pages shows 404

Wait 1-2 minutes after enabling — the first deploy takes time. If it persists,
verify the branch and root folder in Settings → Pages.

---

## ✅ Verification

```bash
# Local
npx docsify-cli serve .
# → sidebar visible, dark toggle works, search works

# Remote — site is up
curl -s -o /dev/null -w "%{http_code}" https://yourname.github.io/tutos_live/
# → 200

# Remote — _sidebar.md is served (Jekyll is disabled)
curl -s -o /dev/null -w "%{http_code}" https://yourname.github.io/tutos_live/_sidebar.md
# → 200  (would be 404 without .nojekyll)
```

---

## 📚 References

- [Docsify documentation](https://docsify.js.org)
- [docsify-darklight-theme](https://github.com/boopathikumar018/docsify-darklight-theme)
- [GitHub Pages docs](https://docs.github.com/en/pages)
- [GitHub CLI (`gh`) full guide](gh.md)
