# Bun — Sovereign JS/TS Runtime & Package Manager

> **Machine:** KpihX-Ubuntu (Ubuntu 25.10)
> **Lived on:** 2026-03 · **Status:** Production-stable

---

Node.js + npm get the job done, but they are slow, verbose, and fragmented —
`npm install`, `npx`, `tsc`, `jest`, `esbuild` are four separate tools to install
and configure before you write a single line of code.

**Bun** collapses the entire JS/TS toolchain into one binary:
runtime, package manager, test runner, bundler, and publisher — all built-in,
all fast.

The mental model that makes this click:

```
Python world        ↔  JS/TS world
─────────────────────────────────────
uv                  ↔  bun
pyenv               ↔  nvm  (kept as fallback for third-party tools)
PyPI                ↔  npmjs.com  (bun publishes there too)
jsr.io              ↔  future TS-native package registry
```

Bun does NOT replace Node.js on this machine — `nvm` and Node stay, because
Gemini CLI, Codex, Copilot, and Claude Code are Node-dependent. Bun is the
**sovereign choice for KpihX-owned projects**.

---

## Installation

```bash
curl -fsSL https://bun.sh/install | bash
```

The installer drops the binary at `~/.bun/bin/bun` and prints a PATH line.
Add it to `~/.kshrc` (the universal shell hub) **before** the nvm block so Bun
takes priority for own projects:

```bash
# ~/.kshrc — near the top, before nvm block
export PATH="$HOME/.bun/bin:$HOME/.npm-global/bin:$PATH"
```

PATH resolution order: `~/.bun/bin` → `~/.npm-global/bin` → rest.
This makes `bun` the default for own projects while npm-global serves third-party
CLI tools (gemini, codex, copilot, claude).

Reload and verify:

```bash
source ~/.kshrc
bun --version
# → 1.x.x
```

---

## What Bun replaces (and what it does not)

```
Command         npm/Node equivalent     Notes
──────────────────────────────────────────────────────────────────
bun install     npm install             Reads package.json, writes bun.lockb
bun add <pkg>   npm install <pkg>       Adds dep + updates lockfile
bun add -d      npm install --save-dev  Dev dependency
bun remove      npm uninstall
bun run <cmd>   npm run <cmd>           Runs scripts from package.json
bun test        jest / vitest           Built-in, jest-compatible API
bun build       esbuild / webpack       Built-in bundler
bun publish     npm publish             Publishes to npmjs.com
bunx <pkg>      npx <pkg>              Runs a package without installing
bun init        npm init                Scaffolds a new project
```

`bun run` also executes TypeScript files directly — no `ts-node`, no compile
step:

```bash
bun run src/index.ts   # runs TS natively
```

---

## Starting a new project

```bash
bun init          # interactive scaffold
# or
bun init -y       # non-interactive with defaults
```

This creates `package.json`, `tsconfig.json`, `index.ts`, and `bun.lockb`.

Declare the engine in `package.json`:

```json
{
  "name": "my-project",
  "version": "0.1.0",
  "engines": {
    "bun": ">=1.0.0"
  }
}
```

---

## TypeScript — zero config

Bun runs TypeScript natively. For strict mode (mandatory for new KpihX projects):

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler"
  }
}
```

No need to install `typescript` as a dependency — Bun uses its own TS parser.
Install `typescript` only if you need `tsc` for type-checking in CI:

```bash
bun add -d typescript
bunx tsc --noEmit   # type-check without emitting files
```

---

## Linting & formatting — Biome

Biome replaces ESLint + Prettier in one tool.

```bash
bunx biome init          # creates biome.json at project root
bunx biome check .       # lint + format check
bunx biome check --write .  # auto-fix
```

`biome.json` minimal config:

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.4/schema.json",
  "organizeImports": { "enabled": true },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2
  }
}
```

---

## Testing — bun test

`bun test` uses a jest-compatible API — no migration needed if you have existing
jest tests.

```typescript
// src/utils.test.ts
import { describe, test, expect } from "bun:test";
import { add } from "./utils";

describe("add", () => {
  test("sums two numbers", () => {
    expect(add(1, 2)).toBe(3);
  });
});
```

```bash
bun test              # run all *.test.ts files
bun test --watch      # re-run on file change
bun test src/utils    # target specific file/pattern
```

Coverage:

```bash
bun test --coverage
```

---

## Bundling — bun build

```bash
bun build src/index.ts --outdir dist --target node
bun build src/index.ts --outdir dist --target browser --minify
```

For a CLI tool that should run on any machine with Node installed:

```bash
bun build src/index.ts --compile --outfile dist/my-tool
# → single self-contained binary, no Bun runtime needed
```

---

## Publishing to npmjs.com

`bun publish` pushes to npmjs.com — same registry as npm.

```bash
bun build                  # build first if needed
bun publish                # dry-run by default in some versions
bun publish --access public  # for scoped packages
```

Lock file `bun.lockb` is binary — commit it:

```bash
# .gitignore — do NOT ignore bun.lockb
# node_modules/ ← yes, ignore
# bun.lockb    ← NO, commit this
```

---

## Docker base image

For KpihX-owned JS/TS services:

```dockerfile
FROM oven/bun:1-slim

WORKDIR /app
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile

COPY src ./src
CMD ["bun", "run", "src/index.ts"]
```

Use `node:22-bookworm-slim` only when a project explicitly depends on Node
runtime behavior (e.g. native addons, Node-only APIs).

---

## Coexistence with Node + nvm

Bun and Node coexist without conflict.

```
~/.bun/bin/bun          ← Bun binary (own projects)
~/.nvm/versions/node/   ← Node managed by nvm (third-party CLIs)
~/.local/bin/node       ← symlink → active nvm node version (PATH fallback)
~/.npm-global/bin/      ← Gemini, Codex, Copilot, Claude (npm-installed)
```

When in a KpihX project directory, `bun` is used for everything.
When running `gemini`, `codex`, or `copilot`, they invoke Node transparently.

**After every nvm upgrade**, refresh the fallback symlink:

```bash
ln -sf ~/.nvm/versions/node/vX.Y.Z/bin/node ~/.local/bin/node
```

---

## Quick reference

```bash
# Project lifecycle
bun init -y                        # scaffold
bun add <pkg>                      # add dependency
bun add -d <pkg>                   # add dev dependency
bun install                        # install from lockfile
bun run <script>                   # run package.json script
bun run src/index.ts               # run TS file directly

# Quality
bunx biome check --write .         # lint + format fix
bunx tsc --noEmit                  # type-check

# Test
bun test                           # all tests
bun test --watch                   # watch mode

# Build & publish
bun build src/index.ts --outdir dist --target node
bun publish --access public

# One-off (no install)
bunx <package>                     # like npx
```
