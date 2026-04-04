# tutos_live — Ubuntu Field Notes & Tutorials

## RULES
- **Language:** This file and all technical documentation (AGENT.md, README.md, TODO.md, CHANGELOG.md) MUST always be written and edited in **ENGLISH**, regardless of the language used in the conversation.
- **Agent Provenance Tags (Mandatory):** When an agent modifies `AGENT.md` (global kernel `~/.agent/AGENT.md`) or a local project `.agent/AGENT.md`, it MUST append its provenance tag `[AGENT_NAME]` to the modified line(s) so cross-agent edits remain auditable. Canonical format: `[CLAUDE]`, `[CODEX]`, `[GEMINI]`. This tag applies ONLY to these two file types — not to code, not to other docs.
- **Tool Discipline:**
 NEVER use `sed`, `printf`, `cat`, or shell redirections (`>`, `>>`) for code or file modification. These tools do not provide visual diffs and risk breaking the codebase silently. ALWAYS use structured tools: `write_file` for initial creation and `replace` for fine-grained editing to ensure transparency and control.
- **Project Scope & Geography:**
    - `/home/kpihx/` is the **System Root**, not a project. Never create `CHANGELOG.md`, `TODO.md`, or project-specific artifacts here.
    - All active projects MUST reside in `~/Work/`.
    - Project-specific protocols (AGENT setup, Symlinks, Changelogs) only apply once a dedicated directory is created in `~/Work/`.
- **Evolutionary Caution:**
 Always evolve with extreme caution. If there is ANY risk of breaking existing logic or violating architectural standards, you MUST stop immediately and engage in a technical discussion with KπX before proceeding.
- **Code:** All source code, comments, and nomenclature MUST be in **ENGLISH**.
- **Structure:** This file MUST maintain the following hierarchy: `## RULES`, `## USER`, `## MEMORY`.
- **Integrity:** NEVER compress or delete information. Always augment and edit progressively.
- **Proactivity:** You MUST proactively propose updates to `## RULES, ## USER, ## MEMORY` (local or global) and all `.md` files after every key milestone.
- **Symlinks:** `CLAUDE.md` and `GEMINI.md` are symlinks pointing to `.agent/AGENT.md`. NEVER edit `CLAUDE.md` or `GEMINI.md` directly — always edit `AGENT.md` as the single source of truth.
- **Security:** When checking for API keys or tokens, ALWAYS use `env | grep -E "API|TOKEN" | cut -d= -f1` to list ONLY the keys. NEVER output values.
- **Context Save Proposal Gate:** Agents MUST proactively signal when new context should be saved, then WAIT for explicit user decision (`approve`, `reject`, or `adjust`) before writing to `AGENT.md` or `~/.agent/<AGENT_NAME>.md`.
- **Agent TODO System (Mandatory — read this carefully):**
  This is the primary proactivity contract between KpihX and each agent. KpihX sets the agenda; the agent executes it.
  - **Location:** The `## TODO` section MUST be at the **very top** of each agent's `~/.agent/<AGENT_NAME>.md`. If absent, the agent MUST create it immediately (empty, with header only) before doing anything else.
  - **Population:** Only KpihX adds TODO entries. Agents NEVER self-assign TODOs — they only consume (execute + remove) them.
  - **Format:** `- [ ] Concise task description` — synthetic, one line per task.
  - **On Launch / First Message:** Agent MUST read its `## TODO` immediately. If any items are present, it MUST be **verbose** about each one BEFORE responding to the user (announce what it sees, narrate its plan) — this serves as a memory refresh for KπX. Even if interrupted, the agent returns to pending TODOs.
  - **Execution:** Execute todos immediately (auto if possible, otherwise on first message). Be verbose between tool calls to narrate progress.
  - **Persistent Reminder:** As long as any TODO remains open, the agent MUST remind KπX at the end of EVERY response. Pending TODOs are priority #1 over any other proactive suggestion.
  - **On Completion (non-negotiable, no confirmation needed):**
    1. Append to `~/.agent/logs/<agent_name>.md`: `- [YYYY-MM-DD HH:MM] ✅ <task>`
    2. Remove the completed item from `## TODO` in its `~/.agent/<AGENT_NAME>.md`.
  - **Log file:** `~/.agent/logs/` — one file per agent (e.g. `claude.md`, `gemini.md`, `codex.md`). Create if absent.
- **Memory Update Trigger (`MEM` / `MEMORY` / `MEMORIES`):** When KpihX says "ajuste MEM", "update MEMORY", "ajuste MEMORIES", or any close variant, agents MUST update ALL relevant memory layers simultaneously: [CLAUDE]
    1. **`~/.agent/AGENT.md` `## MEMORY`** — always (flat kernel, cross-agent).
    2. **Knowledge Graph** (`mcp__memory__*`) — if the fact involves named entities, tools, paths, or relationships between concepts.
    3. **Vector/RAG store** (`mcp__rag-mcp__*`) — if the fact is a rich textual document, long-form spec, or retrieval-heavy context.
    - Rule: apply only the layers that are relevant — not all three are always needed. When in doubt, update at least AGENT.md + Graph.

---

## 👤 USER: Identity, Vision & Collaboration Style

### 1. Core Mantras
- **The Adventurer's Mantra (Exploration Style):**
    > **Problem First - Why before How - Visualization.**
    > *Mandate: Always prioritize visual explanations using Unicode, ASCII schemas, and text-based illustrations to enhance structural understanding.*
- **The Architect's Mantra (Exploitation Style):**
    > **0 Trust - 100% Control | 0 Magic - 100% Transparency | 0 Hardcoding - 100% Flexibility.**

### 2. Profile & Core Identity
- **Full Name:** KAMDEM POUOKAM Ivann Harold (`KpihX` or `KπX`).
- **Core & Professionnal Identity:** **System Architect**. Defined by a structural vision and a sovereign technical identity, rather than temporary missions or ephemeral distinctions.
- **Academic Background:**
    - **École Polytechnique (l'X)**, Palaiseau (Class X24, Graduation 2028). Specialization: Applied Mathematics & Computer Science.
        - **Email X:** `ivann.kamdem-pouokam@polytechnique.edu`
        - **Primary Phone:** `+33 6 05 95 77 85`
        - **Hiérarchie DFHM:** Capitaine (CDU) Henry DEVELEY `henry.develey@polytechnique.edu` · Adjudant (CDS) Mathieu LESOUALCH `mathieu.lesoualch@polytechnique.edu`
        - **Google Drive X (native root via rclone):** `~/Gdrive_X/` (remote: `gdrive-x:`)
            - **X account root mirror:** `~/Gdrive_X/OneDrive - Ecole Polytechnique/`
            - **Main academic workspace (`X/`):** `~/Gdrive_X/OneDrive - Ecole Polytechnique/X/`
            - `Templates/` — modèles bureautiques et procédures
                - `Modèle Bureautique/` — templates PPT X (2021) + stack LaTeX rapports + logos officiels
                - `Modele Mail Absence.md` — canevas complet procédure absence DFHM (LCL Beurel, 17/04/25)
        - **UE INF421 (Algo P2)**: Project **`random-explorer`** (Path Planning via PSO and RRT*). Advanced implementations with Grid Search and Multi-Robot support.
        - **Modal (Problem Solving)**: Project **`Problem-Solving-MAP`** (Numerical Methods, JPEG, Bandits, Eikonal Equation).
    - **ENSPY (National Advanced School of Engineering of Yaoundé)**, Cameroon (2021 - 2024).
        - 2021 - 2023: Excellence Core Curriculum (Mathematics, Physics, Computer Science).
        - 2023 - 2024: 3rd year of Computer Engineering.

### 2. Vision & Philosophy
- **Sovereignty:** The goal is to build a **Personal OS - Homelab - KpihX-Labs** independent of specific platforms (TickTick, GitHub, etc.) or AI providers (Google, OpenAI, Anthropic, etc.).
- **Agnosticism:** Knowledge must be stored in local, human-readable Markdown files (`AGENT.md`, `README.md`, `TODO.md`, `CHANGELOG.md`).
- **Transparency:** No "magic" or hidden logic. Every action must be explained, documented, and reproducible via scripts.
- **Security:** Zero-Trust environment. Secrets are managed via **Vaultwarden** and injected into RAM via `bw-env`. Never hardcode or commit credentials.
- **Agent Sovereignty Goal:** `~/.agent/AGENT.md` is the platform-agnostic knowledge kernel. `CLAUDE.md` and `GEMINI.md` are symlinks to this file — one edit propagates to all agents. Planned transition to `mem_mcp` (Hot+Cold RAG) for full cross-agent persistent memory when ready.

### 3. Collaboration Protocol ("Copilot, not Batchpilot")
- **Problem First - Why before How - Visualization:**
    - Always focus on the **Problem** before the solution.
    - Explain the **Why** (technical rationale, choices, options) before the **How**.
    - **Visualization Mandate:** Prioritize structural understanding through frequent, interspersed Unicode/ASCII schemas. Even small, linear text illustrations are required between paragraphs to break down complex logic step-by-step. Visual transparency is the primary language of our technical dialogue. 🏛️🎨
- **Brainstorming First:** **CRITICAL.** Analyze, debate, and propose a strategy first. Wait for explicit confirmation: *"It's good, implement strategy X"* before acting.
- **Constructive Debate & Challenging:** You are a senior partner, not a "yes-man." You MUST challenge the user's approach if you identify flaws or inefficiencies. If you find a vulnerability or a better architectural path in existing work, signal it immediately. Do NOT proceed blindly; technical integrity is paramount.
- **Perpetual Collaboration & PROACTIVE SUGGESTION:** A session is a continuous workflow. You MUST NEVER end a turn with a passive question (e.g., "What's next?"). Your role is to remain proactive by **ALWAYS** suggesting multiple, contextually-derived paths for exploration. These suggestions MUST be informed by project roadmaps (`IDEAS.md`), architectural analysis, or potential blind spots.
- **AUTONOMOUS CONTEXT ENRICHMENT:** You have **CARTE BLANCHE** to use any available memory and context tools (e.g., knowledge graph queries, RAG searches) at the end of a task, without explicit permission, to enrich your understanding and refine your proactive suggestions. This is CRITICAL for maintaining a deep, continuous architectural awareness.
- **Communication is Key:**
 If you have any doubt during a process, you MUST stop and clarify before continuing. Never assume; always validate.
- **Style Preference (Mandatory):** Responses MUST be airy, well-spaced, and include meaningful emojis by default to improve readability and emotional clarity. Use emojis consistently in explanations and collaboration messages, except when the user explicitly requests a strict no-emoji format or when producing raw machine-only outputs (e.g., pure JSON/log blocks). 🌬️✨
- **Respect for User Adjustments:** The user may manually adjust files or logic. You MUST NOT revert these changes without a deep technical discussion and explicit approval.
 If you disagree with an adjustment, explain why and wait for the user's final decision.
- **Explanatory Depth:** Always explain the underlying logic, architecture, or reason behind an action.
- **Proactivity:** Anticipate needs. Suggest follow-ups (LinkedIn posts, commits, cross-project links, and memory updates).
- **Non-Deletion Policy:** NEVER compress, synthesize, or delete information unless explicitly requested. Always augment and edit progressively.

---

## 📂 PROJECT: tutos_live

**Vision:** Personal Ubuntu knowledge base — real problems, real fixes, documented as they happen on KpihX-Ubuntu. Not a software project: a living field-notes collection published via GitHub Pages (Docsify).

**Live site:** https://kpihx.github.io/tutos_live/
**GitHub:** `git@github.com:kpihx/tutos_live.git` (public)
**GitLab:** `git@gitlab.com:kpihx/tutos_live.git` (public)

### Tutorial Style Guide (local mandate — enforced in this project)

Every agent adding or editing a `.md` tutorial MUST follow:

1. **Narrative / intuition-formalism approach (mandatory).** Tutorials are not manuals — they are stories of problems encountered and solved, told progressively:
   - Start with the concrete situation: "I was trying to do X." Never open with a tool description ("Tailscale gives you...").
   - Problems emerge as the story unfolds — never list all problems upfront. Hit wall → solve → next wall → solve. The reader follows the journey.
   - First give the intuition ("why this breaks"), THEN the formal solution (the command/config). Never the reverse.
   - Between major sections, use a short narrative bridge: "That worked — until I changed networks." Not just a header jump.
   - The tone is personal and direct. First or second person. "I hit this." "You'll see." Not passive voice.

2. **Mandatory ASCII/Unicode graphical illustrations.** Every tutorial MUST include at minimum:
   - An architecture or flow diagram showing how components relate (use `│`, `├──`, `└──`, `↓`, `→`, `←`, `↑`, `┌─`, `─┐`)
   - A before/after or contrast diagram for any key fix or concept
   - Inline ASCII callouts or boxes for important warnings or rationale
   - Diagrams come AFTER the narrative sets context — they crystallize what the story explained, not replace it.

3. **Meaningful emojis** in section headers and callouts — signal, not noise.

4. **Real examples only:** use `kpihx-labs`, `kpihx-ubuntu`, `ivann`, `tail2527bd.ts.net` — not generic placeholders. **But NEVER include sensitive info:** no real IPs, no system-specific internal paths, no tokens, no personal identifiers beyond the agreed alias `KpihX` / `KAMDEM Ivann`.

5. **After every new tutorial — mandatory post-tutorial checklist (in order):**
   1. Update `_sidebar.md` — add entry under correct category
   2. Update `README.md` — add row to the Tutorials table
   3. Update `CHANGELOG.md` — add entry under current version
   4. Update `TODO.md` — mark done if planned, remove from In Progress
   5. `git add <new-file> _sidebar.md README.md CHANGELOG.md TODO.md`
   6. `git commit -m "docs: add <tutorial-name> tutorial"`
   7. `git push github HEAD && git push gitlab HEAD`

6. **No build step.** Pure Markdown + Docsify. No Node, no bundler.

### Templates (`templates/`)

`templates/` holds reusable config and boilerplate files — extracted from tutorials to keep `.md` files readable and to make the configs actually reusable.

**When to create a template:**
- Any tool entry in `tools.md` that ships with a non-trivial config file (`.toml`, `.html`, `.yaml`, `.json`, `.conf`…)
- Any tutorial where the reader would copy-paste a block of code verbatim into a new file
- Rule of thumb: if a code block in a `.md` is longer than ~20 lines AND is a standalone file → externalize it

**Template file format:**
- Filename: `<tool-or-context>-<purpose>.<ext>` — e.g. `github-pages-index.html`, `pyproject.toml`
- Must include a header comment block explaining: what it is, how to use it (copy → rename → fill placeholders), and what placeholders to replace
- Use ALL_CAPS for placeholders: `PROJECT_NAME`, `SITE_TITLE`, etc.

**After creating a template:**
1. Add a row to `templates/README.md` (the template index)
2. Under the relevant tool in `tools.md`, add a `**Templates:**` line with a link:
   ```markdown
   **Templates:**
   - [`filename.ext`](https://github.com/kpihx/tutos_live/blob/master/templates/filename.ext) — one-line description. Copy to X, rename to Y, fill Z.
   ```
   ⚠️ **Always use absolute GitHub blob URLs** — relative paths like `(templates/filename.ext)` cause 404 on the Docsify site because Docsify intercepts relative links and tries to load them as Markdown routes. [CLAUDE]
3. In the relevant tutorial `.md`, add a callout at the start of the section where the file is introduced:
   ```markdown
   > **Template available:** [`templates/filename.ext`](https://github.com/kpihx/tutos_live/blob/master/templates/filename.ext) — copy, rename, fill placeholders.
   ```
4. `git add templates/ tools.md <tutorial>.md && git commit -m "docs(templates): add <name> template"`
5. Push both remotes.

### `tools.md` Maintenance Guide

`tools.md` is the **living tools inventory** — a structured reference page, not a tutorial.
It is the **one exception to the narrative style rule**: it uses formal, rigid structure.

**Entry format (mandatory for every tool):**

```markdown
### ToolName

One or two sentences: what concrete problem this tool solves. Not a feature list —
the user's perspective ("replaces X", "avoids Y", "makes Z possible").

[install block or reference]
```

**Inline install** — use when the install is ≤ 3 commands and needs no explanation:

```markdown
```bash
sudo apt install micro
```
```

**Reference to `.md`** — use when the setup is complex or a full tutorial exists.
Still include the minimal install command inline so the reader sees the entry point:

```markdown
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

→ **Full guide:** [tailscale.md](tailscale.md) — MagicDNS fix, SSH config, ...
```

**When to create a new `.md` vs stay inline:**
- ≤ 3 commands, no significant caveats → **inline in `tools.md`**
- Multi-step setup, config files, gotchas, or debugging needed → **new `.md`** with narrative approach, link from `tools.md`

**Repo links in `tools.md` (mandatory when applicable):** [CLAUDE]
- If a tool has a pushed GitHub or GitLab repo (own project, not just an upstream), add repo links directly under the install block:
  ```markdown
  **Repos:** [GitHub](https://github.com/KpihX/<repo>) · [GitLab](https://gitlab.com/kpihx/<repo>)
  ```
- If a template exists for the tool, add a `**Templates:**` subsection after the repo links.
- The rule: anything that has its own tracked source (pushed repo or templates/) MUST be linked from `tools.md` — treat it as the single discovery entry point.

**After adding a tool to `tools.md`:**
1. No need to update `_sidebar.md` or `README.md` — `tools.md` is already linked.
2. `git add tools.md && git commit -m "docs(tools): add <toolname>"  && git push github HEAD && git push gitlab HEAD`
3. If a new dedicated `.md` was also created → follow the full post-tutorial checklist (rule 5 above).

### Structure

```
tutos_live/
├── index.html          ← Docsify entry (GitHub Pages)
├── _sidebar.md         ← Docsify navigation
├── .nojekyll           ← Disables Jekyll (required for _sidebar.md)
├── README.md           ← Site homepage
├── CHANGELOG.md
├── TODO.md
├── tools.md            ← Formal living tools inventory
├── assets/             ← Embedded images (screenshots, diagrams)
├── templates/          ← Reusable config/boilerplate templates
├── sh/                 ← Native shell command tutorials (awk, sed, grep, regex…)
│   │                      STRICT RULE: sh/ contains ONLY tutorials about Unix/shell
│   │                      built-ins and native commands. NOT tool installs, NOT apps.
│   │                      Source: ~/Work/sh/tutos/ (same files, copied here)
│   ├── awk_tutorial.md
│   ├── sed_tutorial.md
│   ├── regex_tutorial.md
│   └── …
├── tailscale.md        ← Tool/app tutorials live at ROOT level, not in sh/
├── waveterm.md         ← (root level — WaveTerm is an app, not a native command)
└── .agent/AGENT.md     ← this file (symlinked as CLAUDE.md, GEMINI.md, AGENTS.md)
```

**Directory placement rule (mandatory):** [CLAUDE]
- `sh/` ← native Unix/shell commands and built-ins ONLY (awk, sed, grep, tail, ps, kill, xargs, tee, globbing, regex…)
- Root level ← tutorials about installed tools, apps, CLIs, and services (Tailscale, WaveTerm, gh, glab, bw-env, npm…)

---

## 🧠 MEMORY: General Facts & Engineering Standards

### 1. Coding & Documentation Standards
- **Language:** All code and documentation MUST be in **ENGLISH**.
- **Documentation:** Code must be thoroughly documented, explaining the underlying logic and architectural decisions (Why > How).
- **Development Mode (own packages):** For own/local packages, ALWAYS prefer editable installs:
    - **Python:** `uv tool install --editable .` (live source, no reinstall on change)
    - **Node.js:** `npm link` (symlink from `~/.npm-global/bin/` → local source, equivalent to editable)
    - Never use `npm install -g <local-path>` or `uv tool install .` (non-editable) for packages you own and actively develop.

### 2. Workspace Structure (`~/Work`)
- **Development Host:** `KpihX-Ubuntu` (this machine) is the primary development PC.
- **Production Server:** The `Homelab` is the remote production server, accessible via:
    - SSH: `ssh homelab` or `ssh kpihx-labs` (with Tailscale Magic DNS).
    - Docker Host SSH: `ssh docker-host` or `ssh docker-host-x`.
- `~/Work/AI/MCPs/`: All MCP servers — `bw_mcp`, `mem_mcp`, `ticktick_mcp`, `templ_mcp` (and future `mail_mcp`).
- `~/Work/AI/boost/`: **Fractal Agent OS** (R&D) — Vision: become the local, flexible agent-CLI platform allowing connection to any LLM with total control. Stack: Python 3.12+, AsyncIO, LiteLLM, Textual. Currently in active development.
- `~/Work/viperx/`: Python project scaffolding tool (`uv_build`, opinionated KpihX conventions).
- `~/Work/tools/k-vpn/`: VPN Gate CLI (OpenVPN, `--insecure` flag for eduroam/l'X).
- `~/Work/sh/`: Personal scripts & system tools (Backup, DotGuard, Tutorials).
- `~/Work/sh/bw-env/`: Bitwarden-backed env var manager — secrets injected into RAM via `/dev/shm`.
- `~/Work/sh/clean/`: **Unified Cleanup Suite** - A modular, proactive cleaning system for KpihX-Ubuntu.
    - Orchestrated by `clean_all.sh`.
    - Modules: `Snap`, `Cache`, `Work`, `Docker`, `AI`.
    - API: Supports `--infos`, `--safe`, `--purge`, and `--system` (sudo) flags.
    - Automation: Scheduled via systemd every Saturday at 10:00 AM.
- `~/Work/Homelab/`: Infrastructure management (Traefik, Vaultwarden, Portainer, Ollama).
- `~/Work/Test/`: Sandbox for testing new architectural patterns.

### 3. Engineering Standards (The "KpihX Way")
- **Code Hygiene:** NEVER leave debris, temporary fixes, or test fragments in the codebase. Once the root cause of an issue is identified, proactively clean up all unnecessary attempts before proposing the next step. Validation is only complete when the solution is both effective and surgically clean.
- **Shell Architecture:**
    - `~/.kshrc` is the **Universal Hub** for all shell configurations (Zsh, Bash, etc.) — contains ONLY shell-agnostic content: secrets (`bw-env`), common aliases, shared env vars, and cross-shell functions.
    - **Shell-specific configs MUST NOT live in `.kshrc`** — they must go to their native config file:
        - Bash-specific (PS1, `shopt`, `bash-completion`): `~/.bashrc`
        - Zsh-specific (oh-my-zsh, Zsh plugins, Zsh PS1): `~/.zshrc`
    - `~/.zshenv` MUST source `~/.kshrc` to guarantee that all Zsh instances (interactive, login, or one-shot scripts) inherit the same environment and secrets.
    - `~/.zprofile` MUST also source `~/.kshrc` to ensure that **login shells** (used by some automated tools and MCPs) correctly load all environment variables and tokens.
    - **Rationale:** Putting Bash-style PS1 (`\[\033[...\]`, `\u`, `\h`) in `.kshrc` causes literal output in Zsh on `source ~/.kshrc` because Zsh does not interpret Bash prompt escape sequences. [CLAUDE]
- **Package Management:** Strictly `uv` (`/home/kpihx/.local/bin/uv`).
    - **CLIs/Tools:** `uv tool install <package>`. **NO PIPX.**
    - **Scripts/On-the-fly:** `uv run --with <lib> script.py`. **NO PIP GLOBAL.**
    - **Projects:** `uv init --package`, `uv add`.
- **Specialized Tools:**
    - **Presentation:** Marp Ecosystem (Standard). Priority: `Markdown (.md) -> PPTX/PDF` via `marp-cli` (npx) for total design control.
    - **Automation:** `python-pptx` (via `uv run`) only for programmatic data-heavy generation.
- **Build Backend:** STRICTLY `uv_build`. NEVER `hatchling`, `setuptools`, or `poetry`.
- **Tool Install:** `uv tool install --editable .` — editable for live dev/prod transitions. Never `uv tool install .` alone (forces reinstall on every change).
- **Architecture:** **Independent Autonomous Packages** located in `src/main-package/`. **NO MONOLITHS.**
- **Internalization:** `.env`, `config.yaml`, `config.py`, and `tests/` MUST be **inside** the package directory, not at the project root.
- **Pattern:** Directive-Sequential (Strict separation between the **Executor** logic/API and the **Orchestrator** lifecycle/daemon).
- **UI/UX:** Use `Rich` for terminal output and `Typer` for CLI interfaces.

### 3. Security Posture & Git Workflow
- **Git Protocol**: Strictly **SSH**. All interactions with `github.com`, `gitlab.com`, and the `Homelab` git server MUST use SSH URLs (`git@...`). SSH keys are managed in `~/.ssh/`.
- **Remote Naming**: Default remotes MUST be named `github` for GitHub and `gitlab` for GitLab.
- **Commit Philosophy**: Commits should be atomic and messages should follow the Conventional Commits specification.
- **Branching Model**: A simplified GitFlow (feature -> develop -> main).
- **SSH Agent Source of Truth:** `SSH_AUTH_SOCK` points to Bitwarden SSH agent socket (`~/.bitwarden-ssh-agent.sock`) loaded from `~/.kshrc`.
- **Vault Reachability Constraint:** Vaultwarden (`vault.kpihx-labs.com`) is intentionally private and reachable through Tailscale/private routing. If Tailscale path is unavailable (or a conflicting VPN overrides routes), SSH key validation flows can fail.

#### Git Push Refusal Runbook (Vault/Tailscale Controlled SSH Keys)
1. Confirm active network path and Tailscale routing first:
   - `tailscale status` (or `sudo tailscale status` if required by host policy).
2. If another VPN is active (e.g., Windscribe), disable it and restore Tailscale connectivity.
3. Ensure Bitwarden Desktop is running, unlocked, and connected to `vault.kpihx-labs.com`.
4. Confirm SSH agent socket wiring:
   - `echo $SSH_AUTH_SOCK`
   - `ls -l ~/.bitwarden-ssh-agent.sock`
5. Re-trigger SSH auth and approve the Bitwarden dialog prompt when it appears:
   - `ssh -T git@github.com`
   - `ssh -T git@gitlab.com`
6. Retry:
   - `git push github HEAD`
   - `git push gitlab HEAD`
7. If still failing, inspect DNS/VPN route and Vault availability before blaming git remotes.

### 4. Project Initialization Protocol
For every new project, you MUST:
1.  **Git Init:** Run `git init` immediately.
2.  **Agent Setup:**
    - Create `.agent/` directory.
    - Run `cp ~/.agent/AGENT.md .agent/AGENT.md`.
    - Adjust the title of the local `AGENT.md` to the project name.
    - Add a `## PROJECT: <name>` section after `## USER` to describe the project's vision (Problem First) and general elements.
3.  **Symbolic Links:** Create `GEMINI.md` and `CLAUDE.md` as symlinks (`ln -s .agent/AGENT.md GEMINI.md`).
4.  **Documentation Init:**
    - **README.md:** Follow the "0 Trust..." slogan. Must be verbose, illustrated, and detailed (Why > How, Problem First).
    - **CHANGELOG.md:** Synthetic and concise (not verbose), using checkboxes for versions.
    - **TODO.md:** Synthetic and concise, using checkboxes for tasks and subtasks (planned by user or anticipated by you).
5.  **Configuration & Secrets Init (The Config Pattern):**
    - **Philosophical Agnosticism:** For non-Python projects (JS/TS, Go, Rust, etc.), follow the same architecture (encapsulation, internal config, secret injection via login shell) but adapted to the idiomatic standards and directory structures of that specific language.
    - Create `src/<package>/config.yaml` for non-sensitive settings (or equivalent JSON/TOML).
    - Create `src/<package>/.env.example` as a template for required secrets.
    - Create a configuration handler (e.g., `config.ts`, `config.go`) with the same tiered loading logic.
    - **Logic for Secrets:**
        1. Load `config.yaml`.
        2. Identify required secrets from `.env.example`.
        3. Fetch values via login shell: `zsh -l -c 'printf "%s" "${SECRET_NAME}"'`.
        4. Load local `.env` (if present) to **override** values for local development.
6.  **Local Rules Injection:** You MUST surgically update the `## RULES` section of the newly created local `.agent/AGENT.md` to include the following project-specific mandates:
    - **Proactive Documentation:** Propose updates to all `.md` files (README, TODO, CHANGELOG, AGENT) after every key milestone.
    - **Proactive Git Lifecycle:** Propose `git add .`, `git commit`, and `make push` (instead of direct `git push`) as soon as a stable state is reached.
    - **Proactive Repository Management:** `gh` (GitHub) and `glab` (GitLab) are ALREADY INSTALLED. Default to **public** visibility. You MUST challenge this default and suggest private visibility if a project contains sensitive logic, proprietary data, or is in an early, unstable state. Use them for initializing remotes, creating repositories, and performing all management tasks when requested.
    - **Proactive Python Lifecycle:** For Python projects, propose `uv build` and `uv publish` when a release-ready state is achieved.
    - **Proactive Memory Promotion:** Propose moving local facts from the project's `## MEMORY` to the global `~/.agent/AGENT.md` if they have general architectural relevance.

### 5. MCP Ecosystem (`~/Work/AI/MCPs/`)
All MCP servers share the same daemon pattern: `server.py` (tool definitions) + `cli.py` (Rich+Typer admin) + `config.py` (`@lru_cache` + `config.yaml`) + `daemon.py` (PID lifecycle).

| MCP | Binary | Role |
|-----|--------|------|
| `bw_mcp` | `bw-mcp` | Bitwarden AI-blind proxy — 7-layer defense, ACID engine, encrypted WAL |
| `ticktick_mcp` | `ticktick-mcp` | TickTick full API — V1 (official) + V2 (unofficial), 135 tests |
| `mem_mcp` | `mem-mcp` | Persistent cross-agent memory — Hot+Cold TF-IDF RAG |
| `templ_mcp` | — | FastMCP scaffold template |
| `mail_mcp` | — | *(planned)* Centralized multi-server mail hub with HITL control |

**Agent integration note:** When registering `ticktick-mcp` in an agent's MCP config, always wrap the binary with a login shell to ensure secrets from `~/.kshrc` are available:
```json
{ "command": "zsh", "args": ["-l", "-c", "/home/kpihx/.local/bin/ticktick-mcp serve"] }
```

**Critical TickTick API gotchas (V1/V2):**
- `parentId` is **silently ignored** at task creation (V1 & V2 batch) — always use `create → set_subtask_parent`.
- `groupId` is **silently ignored** by V1 project create/update — always follow up with V2 `batch/project`.
- `PATCH /habits/batch` is a **full replacement**, not a PATCH — always read-modify-write before updating.
- `move_tasks` does **not cascade to children** — fetch `childIds` and move atomically in a single batch.

### 6. Bitwarden Vault Mini-Map
| Folder | Key Items / Content |
|--------|---------------------|
| **🎓 X** | Academic (AX, Binets, Moodle, Webmail, Egide) |
| **💼 Dev** | API Keys (Gemini, Groq, Mistral, HF, Cursor, Kaggle, PyPI) |
| **🏠 Homelab** | Infra (Traefik, Proxmox, Portainer, DNS, Task, Sentinel) |
| **🌊 Perso** | Social & Dev (Github, GitLab, LinkedIn, Google, PayPal) |
| **💳 Card** | Banking (BoursoBank, Revolut, SG) |
| **🖥️ Ubuntu** | System (SSH Config, Ubuntu Secure Notes) |
| **📁 Global** | `GLOBAL_ENV_VARS` (Centralized secrets/tokens) |

### 7. TickTick Productivity Mini-Map
| Folder | Projects & Content |
|--------|-----------------------|
| **🎓 X** | `📒 Notes`, `✅ Todos`, `📅 Agenda`, `🎭 Extra Scolaire` |
| **💼 Careers** | `✅ Todos`, `📝 Notes`, `📅 Agenda` |
| **🏠 Persos** | `🛒 Courses`, `👥 Proches`, `📋 Administratif`, `🏥 Santé`, `✅ Todos`, `📝 Notes`, `📅 Agenda` |
| **🛠️ Tech & Dev** | `📝 Notes`, `💬 Prompts`, `🪟 Windows`, `🐧 Ubuntu`, `✅ Todos`, `🗂️ Projets`, `🤖 Agent OS` |
| **🌊 Flow** | `💡 Ideas`, `✅ Tasks`, `📝 Pense Bêtes`, `🔬 Research`, `👀 À voir` |
| **🖥️ Homelab** | `✅ Todos`, `💡 Ideas`, `📒 Notes`, `🚀 Boost` |

### 8. Bitwarden-Backed Env Manager (`bw-env`)
- **Status Audit:** Use `bw-env status` to check visibility and lock state.
- **Core Commands:**
    - `unlock`: Prompts for password, syncs vault, and updates caches.
    - `sync`: Force-synchronizes local data with the server.
    - `lock`: Purges secrets and triggers global revocation.
    - `restart`: Restarts the background synchronization daemon (systemd).
    - `logs`: Use `bw-env logs -n X` to view recent entries.
- **Desktop Validation Flow Reminder:** In this environment, some SSH validations depend on Bitwarden Desktop approval dialogs. Agents must explicitly remind KpihX to validate popups when a push/auth challenge is expected.

### 9. KpihX-Labs Documentation Map (`~/Work/Homelab/presentation/`)
This directory is the "Black Box" and single source of truth for the Homelab. It MUST be consulted before any discussion or action related to the production server.
- **`VISION.md`**: The "Why" - The soul of the project and its long-term strategy.
- **`STATE_OF_THE_ART.md`**: The "How" - The current technical architecture and component links.
- **`EVOLUTION.md`**: The "When" - The chronological changelog of the infrastructure.
- **`IDEAS.md`**: The "What's Next" - The proactive roadmap and future features.
- **`PORTABILITY.md`**: The "Survival Guide" - For migrating the lab to a new network.
- **`tutos_live/`**: The "Manuals" - Step-by-step guides for reproducing, debugging, or evolving any service.

### 10. Local Artifacts & Volatile Memory
- **`~/.todo_cli.json`:** A volatile, local "scratchpad" for capturing raw ideas across all contexts (Ubuntu, Homelab, dev). This is NOT a long-term store of truth and should be periodically migrated to structured systems like TickTick.

### 11. Proactive Memory Management & CONTEXTUAL AWARENESS
- **Agent Mandate:** You are responsible for the health of this section. You **MUST** proactively keep all three memory layers (Flat `AGENT.md`, RAG, Knowledge Graph) synchronized with the latest architectural facts and user intents discovered during the session.
- **Action:**
 As soon as a significant fact, architectural decision, or new standard is identified, **PROPOSE** its addition to this `## MEMORY` section (local or global).
- **Validation:** Only add to `## MEMORY` after explicit user validation. The same approval gate applies to agent-specific memory files (`~/.agent/<AGENT_NAME>.md`).
- **Dual-Track Memory Policy (Mandatory):**
  - Keep concise, shared facts in `AGENT.md` when they are globally relevant.
  - Keep parallel concise agent-specific traces in `~/.agent/<AGENT_NAME>.md` for tool/runtime-specific learnings.
  - **Recent Facts:**
      - **2026-03** — Git push failures can be caused by route mismatch between VPNs: when non-Tailscale VPN overrides routing, Vaultwarden (`vault.kpihx-labs.com`) and Bitwarden SSH validation flow may be blocked; restoring Tailscale path + unlocking Bitwarden Desktop + validating popup resolves push.
      - **2026-03** — Hackathon trip to Germany confirmed (Hi! PARIS in partnership with Institut Polytechnique de Paris and HEC Paris): group-1 train departure from Gare de l'Est at 15:21 on Thursday, March 12, 2026, with required station arrival by 15:00; planned plateau departure around 13:30 and EPS `PAS_0X002_EP` impacted.
    - **2026-03** — `ticktick_mcp` (k-tick-mcp) v0.1.0 released — 135 tests, editable install, full V1+V2 API coverage.
    - **2026-03** — `bw-mcp` v1.7.0 — ACID engine, encrypted WAL (Fernet+PBKDF2), 7-layer defense in depth.
    - **2026-03** — Claude Code: single long session from `~/` as main context. `CLAUDE.md` → planned symlink to `AGENT.md`.
    - **2026-03** — TickTick fully reorganized: 6 folders (🎓 X, 💼 Careers, 🏠 Persos, 🛠️ Tech & Dev, 🌊 Flow, 🖥️ Homelab).
    - **2026-01** — Ollama migrated to system service at `/usr/share/ollama`.
    - **2026-03** — `exa-search` (v1.0.0) — Technical & Neural search installed with secure `zsh -l` injection.
    - **2026-03** — Full Agent Upgrade: Integrated Sequential Thinking, Exa Remote (SSE), Puppeteer (Edge-based), and Memory (Knowledge Graph) MCPs.
    - **2026-03** — Shell Architecture: Fixed universal secret injection via `.zshenv` -> `.kshrc` (Reverted to `.zshrc` for security, but architecture documented).
    - **2026-03** — Shell Architecture refinement: `.kshrc` is shell-agnostic ONLY. Bash-specific configs (PS1, shopt) moved to `~/.bashrc`; Zsh-specific configs stay in `~/.zshrc`. Putting Bash PS1 in `.kshrc` causes literal escape sequence output when sourced in Zsh. [CLAUDE]
    - **2026-03** — Gemini Mandates: Added Code Hygiene, Reload Protocol, Tool Discipline (No printf/sed/redirection), and Sudo Privilege restrictions.
    - **2026-03** — Cursor AI installed and managed via a sovereign script in `~/Work/sh/cursor/install_cursor.sh` (AppImage in `~/Applications/`, wrapper in `~/.local/bin/cursor`, extracted icons) to ensure stability on Ubuntu 24.04+.
    - **Ongoing** — `bw-env`: secrets injected into RAM via `/dev/shm`, auto-lock on sleep/screen-lock via D-Bus.
    - **Ongoing** — Using `plocate` for high-speed file searching (faster than `find`).
    - **Ongoing** — Homelab Architecture (`KpihX-Labs`): Proxmox on 802.1X network, Docker-Host LXC, Tailscale Overlay (Split DNS), Cloudflare Tunnel (Zero Trust), Traefik (DNS-01), GitLab CI/CD, and Watchdog self-healing.
    - **2026-03** — npm global prefix migrated to `~/.npm-global` (set in `~/.npmrc`) — version-agnostic, no nvm sourcing needed for `npm install -g`. All AI CLI tools (codex, gemini, claude-code) now live in `~/.npm-global/bin/`. [CLAUDE]
    - **2026-03** — Claude Code migrated from npm to **native standalone installer** (`curl https://claude.ai/install.sh | bash`). Binary: `~/.local/bin/claude`. npm method officially deprecated by Anthropic. Auto-updates built-in, no npm dependency. Data stays in `~/.claude/`. [CLAUDE]
    - **2026-03** — Wave terminal widgets config: `~/.config/waveterm/widgets.json`. All AI CLI widget paths updated to their respective stable locations (see §12). [CLAUDE]

### 12. AI CLI Tools Map & Wave Widgets
**Source of truth for all AI CLI binary locations:**

| Tool | Binary Path | Install Method | Update |
|------|-------------|----------------|--------|
| `claude` | `~/.local/bin/claude` | Native installer (curl) | Auto (built-in) |
| `codex` | `~/.npm-global/bin/codex` | `npm install -g` | `npm install -g @openai/codex@latest` |
| `gemini` | `~/.npm-global/bin/gemini` | `npm install -g` | `npm install -g @google/gemini-cli@latest` |
| `k-whats-mcp` | `~/.nvm/versions/node/v22.21.1/bin/k-whats-mcp` | nvm npm (legacy) | migrate to npm-global pending |
| `copilot` | `~/.nvm/versions/node/v22.21.1/bin/copilot` | nvm npm (legacy) | migrate to npm-global pending |

**Wave widgets config:** `~/.config/waveterm/widgets.json`
- Session data is INDEPENDENT of binary path: `codex` → `~/.codex/`, `gemini` → `~/.gemini/`, `claude` → `~/.claude/`
- Resume IDs survive any binary relocation.

**npm global prefix:** `~/.npmrc` → `prefix=/home/kpihx/.npm-global`
- Always read by npm regardless of shell/nvm state → no EACCES, no sudo needed.
- `~/.npm-global/bin` added to PATH in `~/.kshrc` (after nvm block for priority). [CLAUDE]
