# Repo Issues

Found while running the site locally (`python3 -m http.server`) and auditing
the repo. Nothing here fails `scripts/validate-catalog.sh` (catalog itself
is internally consistent: 16 entries, all `source` paths exist, exactly 5
featured, no duplicate names, no dangling `composes` references) — these are
gaps found by reading the code and docs directly.

## 1. Catalog on the site never reflects local edits (local-dev gotcha)

**File:** `assets/site.js:14, 177`

```js
var RAW_BASE = "https://raw.githubusercontent.com/cameri/schematics/main/";
...
fetchText(RAW_BASE + ".agent-schematics/marketplace.json")
```

The catalog section is populated by fetching `marketplace.json` from
`raw.githubusercontent.com/cameri/schematics/main/...` — always the live
`main` branch on GitHub — instead of the same-origin file next to
`index.html`.

**Effect:** running the site locally (as just done, on
`http://localhost:8787`) or from a feature branch/fork, editing
`.agent-schematics/marketplace.json` and reloading shows **no change** —
the page keeps rendering whatever is currently on GitHub's `main`. A
contributor testing a new catalog entry before opening a PR will see it
"not show up" and may think their edit is broken, when actually the page
never reads the local file at all.

This is a deliberate choice (comment explains GH Pages historically didn't
serve dotfile directories, and it lets the custom domain `schemaformat.ai`
work without being GH-Pages-specific) — but it has no local/dev override
(e.g. `location.hostname === "localhost"` falling back to the relative
path), so there's currently no way to preview catalog changes locally
before pushing.

## 2. "Rejected automatically" PR rule has no automation in the repo

**File:** `CONTRIBUTING.md:11-12, 44`

> PRs opened by outside contributors without a prior issue are rejected
> automatically, and the rejection will cite this rule.

The only workflow in `.github/workflows/` is `deploy-pages.yml` (builds +
deploys the site on push to `main`). There is no GitHub Action, bot config,
`CODEOWNERS`, or PR template in the repo that checks a PR for a linked
issue. If this enforcement exists, it must live outside the repo (a GitHub
App/bot configured in repo settings, not tracked in version control) —
nothing in the committed code backs the claim, so it can't be verified or
reproduced from the repo alone.

## 3. Minor: dead sort in catalog rendering (fixed)

**File:** `assets/site.js:94-96`

```js
var schematics = (plugins || [])
  .filter(function (p) { return p.category !== "authoring" && p.featured; })
  .sort(function (a, b) { return (b.featured ? 1 : 0) - (a.featured ? 1 : 0); });
```

The list is filtered to `p.featured === true` first, so every remaining
item has the same `featured` value — the subsequent `.sort()` by
`featured` is a no-op (comparator always returns `0`). Harmless, but dead
code that suggests an intended secondary sort (e.g. by name) was never
written.

## Not an issue, for reference

- `scripts/validate-catalog.sh` passes clean (16 entries, featured =
  `update-images-on-push`, `run-a-book-library`, `improve-docker-security`,
  `run-a-movies-and-series-library`, `run-a-music-library`).
- All `<name>.<ext>` files with uncommon formats (`agent.rego`,
  `openssl-server.conf`, `Containerfile.sops`) have their required
  `.schema` companion per the README's own convention.
- No broken relative links found in `README.md` / `CONTRIBUTING.md`, no
  missing `SCHEMATIC.md` in any `schematics/*/` package, all nav anchors
  (`#what`, `#how`, `#differences`, `#faq`, `#catalog`, `#spec`) resolve.
