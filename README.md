# QueryMind — Production Deployment

Production build of **QueryMind v1.2** — browser-native natural-language analytics (BYOK, zero
backend) — with a pre-rendered document for every public route: landing page, privacy policy,
contact page, analytics workspace and custom not-found page.

**Release status: Prepared locally — not committed, not pushed, not deployed.**

| Field | Value |
|---|---|
| Application | QueryMind |
| Release version | v1.2 + SEO route documents |
| Source repository | https://github.com/Shanksreddy005/query_main |
| Source commit | `377e6cd9c3b2386dcdf9a853c9b743256de7576f` — "Give every public route its own document and metadata", on top of `8d06139` (theme buttons), `2b3f4fb` (the source of the live release), `2cc8305`, `a49532a` and `31bf2cc` |
| Build date | 2026-11-25 |
| Build command | `npm ci && npx tsc --noEmit && npm test && npm run build` |
| Type check | `npx tsc --noEmit` — clean |
| Test status at build time | 481 tests / 122 suites / 481 pass / 0 fail |
| Deployment status | **Prepared locally — not committed, not pushed, not deployed** |
| Live baseline | `aa0a7e6` (built from source `2b3f4fb`), deployed and verified 2026-11-25 |

> **The version label identifies a release, not a built-in marker.** The application ships no in-app
> version string (`package.json` stays at `0.0.0` by design), so the only authoritative identity of
> what is deployed is the source commit and the artifact hashes below.

> **Not byte-identical to the live baseline (`aa0a7e6`, source `2b3f4fb`).** Four documents are new
> (`privacy.html`, `contact.html`, `app.html`, `404.html`) and `index.html`, the main JS bundle and
> this README change. The live baseline contains only `index.html`, so `/privacy` and `/contact`
> were answered by client-side routing over the landing document and `/app` and unknown paths by
> the Cloudflare Pages SPA fallback. This release gives each route its own document, adds canonical
> and social metadata, and turns the SPA fallback off with a top-level `404.html`. Stylesheet,
> `robots.txt`, `sitemap.xml`, the PNG, worker scripts, fonts and both R2 `.wasm` objects are
> unchanged. `assets/index-DAlCOhl1.js` is removed by this rotation.

## Route architecture — five documents

`scripts/prerender.tsx` in the source repository renders `<PublicSite route=… />` with
`react-dom/server` after `vite build`, swaps the head block for the route's metadata, and writes
five documents. The workspace document carries only its head and an empty root; the client mounts
the workspace in the browser.

| Route | Document | `<title>` | Canonical / `og:url` | Robots | Social card |
|---|---|---|---|---|---|
| `/` | `index.html` | QueryMind \| Browser-Based Natural Language to SQL | `https://querymind.in/` | `index,follow` | yes |
| `/privacy` | `privacy.html` | Privacy Policy — QueryMind | `https://querymind.in/privacy` | `index,follow` | yes |
| `/contact` | `contact.html` | Contact — QueryMind | `https://querymind.in/contact` | `index,follow` | yes |
| `/app` | `app.html` | Workspace — QueryMind | none | `noindex,nofollow` | no |
| unknown | `404.html` | Page Not Found — QueryMind | none | `noindex,nofollow` | no |

`app.html` and `404.html` deliberately carry no homepage canonical, `og:url`, `og:image` or
`twitter:image`. Every indexable metadata URL is absolute HTTPS on the apex domain.

## Cloudflare Pages expectations

This repository connects to Cloudflare Pages (output = repository root, no build command). The
documents rely on Pages' documented serving rules:

- `/`, `/privacy`, `/contact` and `/app` are served from their matching `.html` files.
- Pages redirects file URLs to their extension-less form: `/privacy.html` → `/privacy`,
  `/contact.html` → `/contact`, `/app.html` → `/app`. On the live origin on 2026-11-25, both
  `/index.html` and `/index/` answered `308` to `/`, so trailing-slash forms are expected to
  converge the same way.
- A top-level `404.html` disables Pages' automatic SPA fallback. Unknown paths return
  `404.html` with HTTP 404, and `app.html` must ship with it so `/app` remains a real document.
- There is no `_redirects`, `_routes.json` or `_headers` file, and none should be added. A
  catch-all rewrite to `index.html` would serve the landing document for `/privacy` and `/app`.

None of this has been measured on the deployed origin for these five documents. It is documented
and simulated locally; the post-deployment checklist below is the live gate.

## Social preview and discovery files

- Social card: [`og/querymind-social-preview.png`](og/querymind-social-preview.png), 1200 × 630
  PNG, 64,834 bytes, referenced as
  `https://querymind.in/og/querymind-social-preview.png` by all three indexable routes.
- `robots.txt`: allows crawling, disallows `/app`, declares
  `Sitemap: https://querymind.in/sitemap.xml`.
- `sitemap.xml`: exactly `https://querymind.in/`, `https://querymind.in/privacy` and
  `https://querymind.in/contact`; no `/app`, no `lastmod`.

## Why two hosts — DuckDB-WASM on R2

Cloudflare Pages rejects uploads of individual files larger than 25 MiB. Both DuckDB-WASM binaries
exceed it and are served from the first-party R2 bucket. **This repository does not contain either
binary, and this preparation did not download, replace, upload, delete or modify any R2 object.**

| File | Bytes | Served from | R2 object key |
|---|---|---|---|
| `duckdb/duckdb-eh.wasm` | 35,913,747 | `https://assets.querymind.in/duckdb/1.33.1-dev57.0/duckdb-eh.wasm` | `duckdb/1.33.1-dev57.0/duckdb-eh.wasm` |
| `duckdb/duckdb-mvp.wasm` | 41,325,187 | `https://assets.querymind.in/duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` | `duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` |

Bucket `querymind-static`, custom domain `assets.querymind.in`, existing version
`1.33.1-dev57.0` — all unchanged. The generated bundle resolves the engine base URL as
`https://assets.querymind.in/duckdb/${version}` with `version = "1.33.1-dev57.0"` and references
`duckdb-browser-eh.worker.js` and `duckdb-browser-mvp.worker.js`, which remain in this repository.

R2 requirements remain as configured: `Content-Type: application/wasm`, immutable long cache,
`GET` CORS for `https://querymind.in` (and the other approved origins), bucket otherwise private.

## Production artifact inventory

Fifteen deployable Pages files. Sizes and SHA-256 are of the files in this working tree; each was
compared byte-for-byte with the verified `dist/` output. Status is relative to the live baseline
`aa0a7e6`.

| Path | Bytes | SHA-256 | Status |
|---|---|---|---|
| `index.html` | 32,934 | `a943c9f849030725f5406b893d19c86e4d7f2afc2d5b7caf58bfc2d82044bf31` | modified |
| `privacy.html` | 24,253 | `a049c5b1c7d605a821c201c367638b86c2a7f3472fada21269a346c4da0f2bfb` | new |
| `contact.html` | 16,324 | `93b50c2426ce8577c9ec2652e221f456ec58ac3181647f62db44b8f3e6919cda` | new |
| `app.html` | 2,160 | `bf1f6a309e2ce8603710ba16eb374fbe5a50645c79e513ed1591c3f70b9a289c` | new |
| `404.html` | 10,241 | `4471c7e66b3869f71c3a80318af1cec61a9ee56c6ddba95c1869c1bc04d55e41` | new |
| `robots.txt` | 270 | `1d3b2168d85434b1abff9032935358b47c6664a7a2c13d84bb0860a6ce9b9b75` | unchanged |
| `sitemap.xml` | 482 | `c5ab36bad48e5b5f6933ccb3b20713fc58b7395cd5bb646ba7edeae88d69a046` | unchanged |
| `og/querymind-social-preview.png` | 64,834 | `9a15f6798382dda1fd3ff9b9003d05880aeb3b7938f98c4a8c7c38b68ec9672c` | new |
| `assets/index-CHAEi_d6.js` | 832,068 | `ea68a6bb2516525e50db647b692c38e775e501c54c9fb8ebc6c6da6506468c02` | new (replaces `assets/index-DAlCOhl1.js`) |
| `assets/index-BoCwoH1v.css` | 58,279 | `ed26d8c6c2ca757cd38fa6446867e7a6c87781c6ff3e3abd3c2a8c4d35a705df` | unchanged |
| `assets/aistudio/image_ff69bad7.png` | 88,620 | `ff69bad7d2d91321292459a2b54392093e123d0575276a5ee8d7f69bc0cac6dc` | unchanged |
| `duckdb/duckdb-browser-eh.worker.js` | 773,223 | `fa889e6068c40426dea67c08cf16ce0cad7404eae94f6a2522adcabb5898eb93` | unchanged |
| `duckdb/duckdb-browser-mvp.worker.js` | 839,642 | `964f678d3bfa5a23deb154e0a7950634a4d1415e571040ab2d47815dc3f13582` | unchanged |
| `fonts/inter-latin-variable.woff2` | 48,256 | `3100e775e8616cd2611beecfa23a4263d7037586789b43f035236a2e6fbd4c62` | unchanged |
| `fonts/jetbrains-mono-latin-variable.woff2` | 31,432 | `83c005d49d8a6a50474c73a5a36ac0468076e9c4a29da7bdb14995d80560a5be` | unchanged |

Removed relative to the live baseline: `assets/index-DAlCOhl1.js`. Removed relative to the local
unpushed commit `e393b7e`: `assets/index-Ci7VMYy9.js`. No other file is removed.

Repository-only, not deployed: `.gitattributes` (disables end-of-line conversion so clones match
the artifacts byte-for-byte) and this `README.md`.

## Deployment procedure

1. A separate authorization is required to commit and push this working tree. The push is what
   triggers Cloudflare Pages; this preparation does not push or deploy.
2. Upload is the repository root as-is — no Pages build command, no generated files to exclude.
   Every file is below the 25 MiB per-file limit; the two `.wasm` binaries are not here.
3. Before deploying, confirm both R2 objects still answer HTTP 200 with `application/wasm` and the
   pinned sizes; this release changes neither object nor key.
4. After deployment, run the checklist below against the live origin.

## Post-deployment verification checklist

None of these has been measured on the deployed origin for this release. The previously recorded
results in the source repository's verification documents cover earlier releases only.

1. `/`, `/privacy`, `/contact` each return their own document, with the title, description,
   canonical, `og:url` and robots directive in the route table above; each has exactly one `h1`.
2. `/app` returns `app.html` with `Workspace — QueryMind`, `noindex,nofollow`, no canonical and no
   social metadata; the workspace mounts and DuckDB-WASM loads from R2; a curated keyless query
   passes integrity checks.
3. `/privacy.html`, `/contact.html`, `/app.html` and their trailing-slash forms redirect (`308`) to
   the extension-less routes, matching the measured `/index.html` behaviour.
4. An unknown path returns `404.html` with HTTP 404, `noindex,nofollow`, no canonical and one `h1`.
5. `/robots.txt` returns the real file as `text/plain`, disallowing `/app` and declaring
   `https://querymind.in/sitemap.xml`; `/sitemap.xml` returns the real file as `application/xml`
   with exactly the three indexable URLs and no `/app` entry.
6. `/og/querymind-social-preview.png` returns the 64,834-byte PNG with `image/png`; LinkedIn and
   Google preview caches are external and must be checked separately.
7. Both public theme buttons toggle and persist across reloads; no monitor/system control remains.
8. No console errors from the application's own code; application requests are first-party (Pages
   origin, `assets.querymind.in`, one provider endpoint only when a query runs). The earlier
   observation of a Cloudflare-injected analytics beacon remains an open hosting-level finding and
   must be checked in the Cloudflare dashboard, not inferred from repository content.
9. With no key, a bundled sample query answers locally; an uploaded dataset plus a free-text
   question is refused with the provider-key notice. A real provider key has not been tested in
   this preparation.
10. Search Console: ownership, sitemap submission, Google-selected canonicals and indexing status
    have not been verified; no account access was used.

## Build provenance

1. The build ran in `query_main` at
   `377e6cd9c3b2386dcdf9a853c9b743256de7576f` with branch `main`, `origin/main` at the same commit
   and `git status` empty. `package-lock.json` is unchanged (SHA-256
   `7ae1734fcbc8d3e79b5d2b91fe1896a3787b5d868e0f63365561dc77566179c0`); no `.env` file exists in
   the checkout (only the placeholder `.env.example`).
2. `npm ci` succeeded from the lockfile; `npx tsc --noEmit` was clean; `npm test` reported 481
   tests / 122 suites / 0 fail; `npm run build` succeeded and pre-rendered all five documents.
3. The build emitted exactly 17 files: the 15 deployable files above plus
   `duckdb/duckdb-eh.wasm` and `duckdb/duckdb-mvp.wasm`. No source maps and no environment files
   were emitted.
4. All 15 deployable files were copied from that `dist/`; each was compared byte-for-byte with the
   build and re-hashed in place. The two `.wasm` files were not copied. Their source hashes
   (`3abdec74…` and `ee556014…`) match the R2 inventory recorded above.
5. The only non-`dist` files here are this README and `.gitattributes`.

## Repository reconciliation

The local branch also contains `e393b7e` — "Rotate production assets for the light/dark theme
buttons", built from source `8d06139`. It changed only `README.md`, `index.html` and the JS bundle,
and it has not been pushed. The theme behaviour it introduced is already part of source `377e6cd`,
so this candidate supersedes its two generated files and removes `assets/index-Ci7VMYy9.js`.
`e393b7e` was not reset, amended, cherry-picked, merged, pushed or deleted during preparation.
Because the branch HEAD remains at that commit, any future push prepared without a separately
authorized history decision would publish it alongside the new release commit; this README makes
no claim that it is part of the live baseline.

## Maintenance rules

- Never edit the generated JS/CSS or rename files — every artifact is content-addressed and
  verified by checksum above.
- To update: build from a verified commit in `query_main`, copy every deployable `dist/` file here,
  remove superseded hashed files, regenerate this README and re-hash, then deploy. If the
  DuckDB-WASM package version changes, update `DUCKDB_WASM_VERSION` in
  `src/services/sqlEngine.ts` and upload the new binaries under the new versioned R2 key.
- Keep `404.html` and `app.html` together: with a top-level `404.html`, Pages no longer falls back
  to `index.html` for `/app`.
- Release status must be explicit: a working tree that has not been pushed is a prepared
  candidate, not a deployment. Update the status table only with results measured on the live
  origin.
- This repository must never contain source code, tests, `node_modules`, `.env` files, keys or
  logs.
