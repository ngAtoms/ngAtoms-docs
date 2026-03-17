# ngAtoms-docs — GitHub Pages Deployment Design

**Date:** 2026-03-17
**Status:** Approved

## Overview

Separate `ngAtoms-docs` from the `NgAtoms_lib` parent directory into its own GitHub repository (`thereisnoplacelike/ngatoms-docs`) and deploy it as a GitHub Pages static site at `https://thereisnoplacelike.github.io/ngatoms-docs`. Retire `apps/docs` from the ngatoms monorepo.

## Goals

- Public-facing documentation site served via GitHub Pages
- Automatic deployment on every push to `main`
- No orphan `gh-pages` branch — use the GitHub Pages artifact deploy pattern

## Non-Goals

- SSR / server-side rendering (static prerendering only)
- Hosting on Vercel, Netlify, or any other platform

---

## Section 1: AnalogJS Static Configuration

### Changes to `vite.config.ts`

- Pass `outputMode: 'static'` to the `analog()` plugin to switch from SSR to static prerendering
- Set `base: '/ngatoms-docs/'` so all asset paths resolve correctly under the GitHub Pages subpath
- Remove the `preview` npm script from `package.json` (it launches a Node SSR server, irrelevant for static output)

**Build output:** `dist/analog/public/` — a fully static directory ready for GitHub Pages

### Why static

GitHub Pages serves static files only. AnalogJS's static mode prerenders all routes to HTML at build time, producing output compatible with Pages without any Node.js server.

---

## Section 2: GitHub Actions Workflow

**File:** `.github/workflows/deploy.yml`

### Trigger
Push to `main`.

### Steps
1. Checkout repository
2. Setup Node.js 20
3. `npm ci`
4. `npm run build`
5. Upload `dist/analog/public/` as a GitHub Pages artifact (`actions/upload-pages-artifact`)
6. Deploy artifact to GitHub Pages (`actions/deploy-pages`)

### Permissions
```yaml
permissions:
  pages: write
  id-token: write
```

### Concurrency
```yaml
concurrency:
  group: pages
  cancel-in-progress: true
```
One deployment at a time; new pushes cancel in-progress runs.

### One-time manual step
After creating the GitHub repo, set **Settings → Pages → Source** to **"GitHub Actions"** (not the branch/folder method).

---

## Section 3: Retiring `apps/docs` from the Monorepo

The ngatoms monorepo (`ngatoms/`) has an `apps/docs` Angular playground that is superseded by this dedicated docs repo.

### Changes to `ngatoms/`
- Delete `apps/docs/` directory
- Remove the `docs` project entry from `angular.json`
- The `apps/*` workspace glob in root `package.json` remains (covers `apps/example`)
- Update `ngatoms/CLAUDE.md`: remove `apps/docs` from the packages table
- Update `NgAtoms_lib/CLAUDE.md`: note that `ngAtoms-docs` is now a separate GitHub repo (`thereisnoplacelike/ngatoms-docs`), not a local sibling directory

---

## File Summary

| File | Action |
|---|---|
| `ngAtoms-docs/vite.config.ts` | Add `outputMode: 'static'` and `base: '/ngatoms-docs/'` |
| `ngAtoms-docs/package.json` | Remove `preview` script |
| `ngAtoms-docs/.github/workflows/deploy.yml` | Create Pages deploy workflow |
| `ngatoms/apps/docs/` | Delete entire directory |
| `ngatoms/angular.json` | Remove `docs` project entry |
| `ngatoms/CLAUDE.md` | Remove `apps/docs` row from packages table |
| `NgAtoms_lib/CLAUDE.md` | Update `ngAtoms-docs` description to reflect separate repo |
