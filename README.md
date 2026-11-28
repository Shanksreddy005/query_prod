# QueryMind — Production Deployment

Production build of **QueryMind v1.2** — browser-native natural-language analytics (BYOK, zero
backend) — with a pre-rendered document for every public route: landing page, privacy policy,
contact page, analytics workspace and custom not-found page.

**Release status: prepared locally for review on 2026-11-28. This worktree holds an uncommitted
artifact transfer built from source commit `4ab9c8e`; it has not been committed, pushed or
deployed, and the live origin has not been re-measured for this preparation.**

Cloudflare Pages deploys this repository automatically on any push to `main`, so the artifacts below
describe a prepared candidate, not a live state. The 2026-11-25 deployment and live measurements
later in this document are historical records of an earlier release; they do not describe these
files. See [Deployment identity](#deployment-identity-and-live-verification).

| Field | Value |
|---|---|
| Application | QueryMind |
| Release version | v1.2 + SEO route documents + privacy hardening (prepared) |
| Source repository | https://github.com/Shanksreddy005/query_main |
| Source commit | `4ab9c8e9e3f63b249349fa1029e6971d6421a509` — "Harden AI metadata disclosure and column preflight" |
| Source branch / remote | `main`, `origin/main` at the same commit; clean working tree |
| Production repository | https://github.com/Shanksreddy005/query_prod |
| Production release commit | Not created — this transfer is uncommitted |
| Production parent commit | `953eb3d473ff3f8b44f22977f3991adb1c9cb756` — "Deploy SEO Release 1 production artifacts" |
| Production branch / remote | `main`, `origin/main` at the parent commit; prepared files uncommitted |
| Build date | 2026-11-28 |
| Build command | `npm ci && npm test && npm run lint && npm run build` |
| Type check | `npm run lint` (`tsc --noEmit`) — clean |
| Test status at build time | 519 tests / 129 suites / 519 pass / 0 fail |
| Deployment | None — no commit, push or Cloudflare deployment was made |
| Deployment check | Not applicable |
| Live verification | Not performed for this preparation |
| Previous production release commit | `953eb3d473ff3f8b44f22977f3991adb1c9cb756` |

> **The version label identifies a release, not a built-in marker.** The application ships no in-app
> version string (`package.json` stays at `0.0.0` by design), so the only authoritative identity of
> what is deployed is the production commit and the artifact hashes below.

## Release identity and commit provenance

### Source

- Repository `query_main`, branch `main`, HEAD and `origin/main` at
  `4ab9c8e9e3f63b249349fa1029e6971d6421a509`, working tree clean.
- Commit message: "Harden AI metadata disclosure and column preflight". Its parent is
  `75fe0bb30bbdadf8c68c07ce5200dc9ef9bfaddf` ("Implement SEO Release 1").
- The commit changes 11 files (+803/−45): the numeric range threshold and outward rounding in
  `src/services/schemaProfiler.ts`; the bounded local column preflight and provider-side scope
  allowances in `src/services/aiService.ts`; the privacy, landing, API-key and telemetry disclosure
  copy; `docs/ARCHITECTURE.md`; and the associated regression tests.

### Production

- Repository `query_prod`, branch `main`, HEAD and `origin/main` at
  `953eb3d473ff3f8b44f22977f3991adb1c9cb756` before this preparation.
- This preparation modifies five documents (`index.html`, `privacy.html`, `contact.html`,
  `app.html`, `404.html`), adds `assets/index-D2cQV5H0.js`, removes
  `assets/index-BbFiEFSP.js`, and regenerates this README. It is not committed and no release
  commit exists for it.

### Build provenance

1. The build ran in `query_main` at
   `4ab9c8e9e3f63b249349fa1029e6971d6421a509` with branch `main`, `origin/main` at the same commit
   and `git status` empty. `npm ci` reinstalled 208 packages from the lockfile; `npm audit`
   reported one moderate advisory and no dependency was changed. No `.env` file exists in the
   checkout.
2. `npm test` reported 519 tests / 129 suites / 0 fail; `npm run lint` (`tsc --noEmit`) was clean;
   `npm run build` succeeded and pre-rendered all five documents. The only build warning is Vite's
   chunk-size advisory for the 837.85 kB (228.30 kB gzip) main bundle.
3. The build emitted exactly 18 files: the 16 deployable files listed in the inventory below plus
   `duckdb/duckdb-eh.wasm` and `duckdb/duckdb-mvp.wasm`. No source maps and no environment files
   were emitted.
4. All 16 deployable files were copied from that `dist/`; each was compared byte-for-byte with the
   build and re-hashed in place. The two `.wasm` files were not copied. Their source hashes
   (`3abdec74…` and `ee556014…`) match the R2 inventory recorded below.
5. The only non-`dist` files in this repository are this README and `.gitattributes`.

### What changed relative to the previous production artifacts

The previous production repository state (`953eb3d`) carried the SEO Release 1 documents and
`assets/index-BbFiEFSP.js`. This preparation replaces the main JS bundle with
`assets/index-D2cQV5H0.js` and updates all five HTML documents to reference it. The privacy policy
and landing FAQ now state the 25-row range-disclosure threshold, outward two-significant-figure
widening and the minimisation caveat instead of an absolute; the privacy policy additionally
records that an identifier-shaped column the table does not store is refused in the browser before
any provider request. The stylesheet, `_headers`, `robots.txt`, `sitemap.xml`, worker scripts,
fonts, the AI Studio image and the social card are byte-identical to the previous production
artifacts.

The 2026-11-25 live measurement recorded `/privacy` and `/contact` responses 398 and 253 bytes
larger than the then-committed artifacts because Cloudflare injects its email-obfuscation script at
the edge. That is an edge transformation, not an artifact mismatch; no live measurement was taken
for this preparation.

## Deployment identity and live verification

> **Historical record — 2026-11-25 deployment.** The values in this section describe the
> `550c43b` release and its live measurements. They do not identify, and were not re-run for, the
> prepared artifacts above.

Pushing `query_prod` `main` to `origin/main` triggered the existing Git-connected Cloudflare Pages
project `query-prod` automatically. No manual `wrangler pages deploy`, deployment approval or
redeploy was performed.

| Evidence | Value |
|---|---|
| Cloudflare check run | `Cloudflare Pages`, app `cloudflare-workers-and-pages`, status `completed`, conclusion `success` |
| Started / completed | 2026-11-25T16:47:21Z (both timestamps) |
| Deployment ID | `01f6fd35-62e5-4303-a349-309aa8abd0b8` |
| Project / account | `query-prod` / `564cd063e78f8ab9d3b7b1442f037db7` |
| Deployment preview | `https://01f6fd35.query-prod.pages.dev` |

Cloudflare preview documents were compared byte-for-byte with the committed artifacts, and the
production origin was compared with both:

| Document | Preview SHA-256 | Production SHA-256 | Commit artifact SHA-256 |
|---|---|---|---|
| `/` | `a943c9f849030725f5406b893d19c86e4d7f2afc2d5b7caf58bfc2d82044bf31` | same | same |
| `/app` | `bf1f6a309e2ce8603710ba16eb374fbe5a50645c79e513ed1591c3f70b9a289c` | same | same |
| unknown route (`404.html`) | `4471c7e66b3869f71c3a80318af1cec61a9ee56c6ddba95c1869c1bc04d55e41` | same | same |

The GitHub Deployments API returned `[]` and the GitHub commit-status API returned `pending` with
zero contexts; the Cloudflare integration reports deployments through its check run rather than
those APIs.

**Identity caveat:** the Cloudflare dashboard's provider-side commit SHA field, environment and
branch were not independently inspected because authenticated dashboard access was unavailable. Do
not treat that field as verified. Identity is established by the Cloudflare-authored check run
linking deployment `01f6fd35…` to the release commit, plus byte-identical preview and production
documents.

Live verification was read-only and ran on 2026-11-25 (initial route matrix 16:53 UTC, re-measured
17:00–17:02 UTC). No deployment, cache purge, Search Console request, DNS, R2, Cloudflare or Git
change was performed during verification.

## Route architecture — five documents

`scripts/prerender.tsx` in the source repository renders `<PublicSite route=… />` with
`react-dom/server` after `vite build`, swaps the head block for the route's metadata, and writes
five documents. The workspace document carries only its head and an empty root; the client mounts
the workspace in the browser.

| Route | Document | Last measured HTTP (2026-11-25) | Prepared bytes | `<title>` | Canonical / `og:url` | Robots | Social card |
|---|---|---|---|---|---|---|---|
| `/` | `index.html` | 200 | 34,259 | QueryMind \| Browser-Based Natural Language to SQL | `https://querymind.in/` | `index,follow` | yes |
| `/privacy` | `privacy.html` | 200 | 25,102 | Privacy Policy — QueryMind | `https://querymind.in/privacy` | `index,follow` | yes |
| `/contact` | `contact.html` | 200 | 16,324 | Contact — QueryMind | `https://querymind.in/contact` | `index,follow` | yes |
| `/app` | `app.html` | 200 | 2,160 | Workspace — QueryMind | none | `noindex,nofollow` | no |
| unknown | `404.html` | 404 | 10,241 | Page Not Found — QueryMind | none | `noindex,nofollow` | no |

`app.html` and `404.html` deliberately carry no homepage canonical, `og:url`, `og:image` or
`twitter:image`. Every indexable metadata URL is absolute HTTPS on the apex domain. After mount,
`/app` uses a client-rendered `h2` and has no `h1`; this is recorded as a minor heading/accessibility
observation, not an indexing failure, because the route is `noindex,nofollow`.

### Cloudflare Pages serving behaviour (measured 2026-11-25)

This repository connects to Cloudflare Pages (output = repository root, no build command). The
following was measured on the live origin:

- `/`, `/privacy`, `/contact` and `/app` return their matching documents with HTTP 200.
- `/index.html`, `/privacy.html`, `/contact.html` and `/app.html` return HTTP 308 to `/`, `/privacy`,
  `/contact` and `/app`. The trailing-slash forms `/privacy/`, `/contact/` and `/app/` return the
  same 308 redirects.
- A top-level `404.html` disables Pages' automatic SPA fallback. Unknown paths return `404.html`
  with HTTP 404 and `Cache-Control: no-store`.
- No route fell back to the homepage: `/app` returned the 2,160-byte workspace document and the
  unknown path returned the 10,241-byte not-found document.
- There is no `_redirects` or `_routes.json` file, and neither should be added. A catch-all rewrite
  to `index.html` would serve the landing document for `/privacy` and `/app`. A top-level `_headers`
  file (Phase 1, below) carries response headers only and does not participate in routing.

## Phase 1 security headers (prepared 2026-11-26)

`_headers` at the repository root adds five non-CSP response headers to every static asset response.
The file is emitted from `query_main/public/_headers` by the production build and copied here
byte-for-byte; Cloudflare Pages parses it at deploy time and never serves it as an asset.

```text
/*
  Strict-Transport-Security: max-age=86400
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: accelerometer=(), camera=(), display-capture=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), publickey-credentials-get=(), usb=(), clipboard-write=(self)
```

- `Strict-Transport-Security: max-age=86400` — one day, deliberately short for the first release. No
  `includeSubDomains` and no `preload`: `www.querymind.in` is unverified and HTTPS enforcement for
  subdomains has not been decided. HSTS is sticky in clients, so the value is raised only after a
  clean verification window.
- `X-Content-Type-Options: nosniff` and `Referrer-Policy: strict-origin-when-cross-origin` match the
  values Pages already returned on 2026-11-25; declaring them keeps the guarantee if platform
  defaults change.
- `X-Frame-Options: DENY` — neither the public pages nor the workspace are designed for embedding.
- `Permissions-Policy` disables sensor, capture, geolocation and payment APIs the application never
  uses. `clipboard-write=(self)` is kept because three copy controls call
  `navigator.clipboard.writeText`.

Deliberately excluded from this phase: `Content-Security-Policy` (enforcing or report-only), `COEP`,
`CORP`, `Cache-Control` and `Access-Control-Allow-Origin`. CSP needs its own browser-verified pass
because the DuckDB worker is created from a blob URL and inherits the document policy, including the
Emscripten `eval` requirement; existing cache and CORS values are unchanged by this file.

Phase 1 security headers were prepared and locally committed on 2026-11-26. Live response-header
verification remains pending. The last live measurement, on 2026-11-25, predates the file and
carried none of these five headers. Record post-deployment values only after measuring them on the
live origin.

After deployment, verify with:

```bash
curl -sSI https://querymind.in/ | grep -iE 'strict-transport-security|x-content-type-options|x-frame-options|referrer-policy|permissions-policy'
curl -sSI https://querymind.in/app | grep -iE 'strict-transport-security|x-content-type-options|x-frame-options|referrer-policy|permissions-policy'
curl -sSI https://querymind.in/privacy | grep -iE 'strict-transport-security|x-frame-options|content-security-policy'
curl -sSI https://querymind.in/assets/index-CHAEi_d6.js | grep -iE 'strict-transport-security|x-content-type-options'
curl -sS -o /dev/null -w '%{http_code}\n' https://querymind.in/_headers
curl -sS -o /dev/null -w '%{http_code}\n' https://querymind.in/definitely-not-a-route
curl -sSI https://querymind.in/privacy.html
```

Expect five headers on 200 responses, `404` for `/_headers` itself and for unknown routes, and an
unchanged `308` for the `.html` form. Pages does not document whether `_headers` applies to 404 or
redirect responses, so record the measured behaviour rather than assuming it.

## Live SEO verification

> **Historical record — measured on 2026-11-25** against the then-deployed release. The prepared
> documents carry the same route architecture and metadata contract, but they were not re-measured
> on the live origin.

Measured from raw HTML (no JavaScript) on 2026-11-25:

- Homepage title `QueryMind | Browser-Based Natural Language to SQL` and description
  `Ask questions about your data in plain English. QueryMind turns natural language into SQL and runs the analysis locally in your browser with DuckDB-WASM.`
  matched byte-for-byte.
- Privacy and Contact returned their unique titles, descriptions, canonicals and `og:url` values.
- All three indexable routes reference `https://querymind.in/og/querymind-social-preview.png` via
  `og:image` and `twitter:image`.
- Landing, privacy, contact and 404 documents contain pre-rendered `<main>` markup; `app.html`
  contains an empty root for the client mount.
- Each indexable public page and the 404 page contain exactly one `h1`. `/app` has none.
- `/app` and the unknown route carry `noindex,nofollow`, no canonical and no social metadata.
- No stale bundle references or broken local paths were found.

## Social preview, robots and sitemap

- Social card: [`og/querymind-social-preview.png`](og/querymind-social-preview.png), 1200 × 630
  PNG, 64,834 bytes, SHA-256
  `9a15f6798382dda1fd3ff9b9003d05880aeb3b7938f98c4a8c7c38b68ec9672c`, unchanged in this
  preparation. It is referenced as `https://querymind.in/og/querymind-social-preview.png` by all
  three indexable routes; `app` and `404` do not reference it.
- `robots.txt`: `text/plain`, 270 bytes. Allows crawling, disallows `/app`, and declares
  `Sitemap: https://querymind.in/sitemap.xml`.
- `sitemap.xml`: `application/xml`, 388 bytes, SHA-256
  `f16c3ae0dd23ebd3e91442068319628e7c41a17b84fbfa35cb3e0eb52885d34b`, unchanged in this
  preparation. Contains exactly `https://querymind.in/`, `https://querymind.in/privacy` and
  `https://querymind.in/contact`; no `/app` entry. It carries `lastmod` values (`/` 2026-11-28,
  `/privacy` and `/contact` 2026-11-25).

## Why two hosts — DuckDB-WASM on R2

Cloudflare Pages rejects uploads of individual files larger than 25 MiB. Both DuckDB-WASM binaries
exceed it and are served from the first-party R2 bucket. **This repository does not contain either
binary, and no R2 object was downloaded in full, replaced, uploaded, deleted or modified during the
release.**

| File | Bytes | Served from | R2 object key |
|---|---|---|---|
| `duckdb/duckdb-eh.wasm` | 35,913,747 | `https://assets.querymind.in/duckdb/1.33.1-dev57.0/duckdb-eh.wasm` | `duckdb/1.33.1-dev57.0/duckdb-eh.wasm` |
| `duckdb/duckdb-mvp.wasm` | 41,325,187 | `https://assets.querymind.in/duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` | `duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` |

Bucket `querymind-static`, custom domain `assets.querymind.in`, existing version
`1.33.1-dev57.0` — all unchanged. The generated bundle resolves the engine base URL as
`https://assets.querymind.in/duckdb/${version}` with `version = "1.33.1-dev57.0"` and references
`duckdb-browser-eh.worker.js` and `duckdb-browser-mvp.worker.js`, which remain in this repository.

Live R2 verification on 2026-11-25:

- Both objects answered ranged GET with HTTP 206 and `Content-Type: application/wasm`; sizes
  `35,913,747` and `41,325,187` bytes; `Cache-Control: public, max-age=31536000, immutable`.
- Requests with `Origin: https://querymind.in` received
  `Access-Control-Allow-Origin: https://querymind.in`; a request with `Origin: https://example.com`
  received no ACAO header.
- `query_prod` tracks zero `.wasm` files; both worker scripts returned HTTP 200 from Pages.

R2 requirements remain as configured: `Content-Type: application/wasm`, immutable long cache,
`GET` CORS for `https://querymind.in` (and the other approved origins), bucket otherwise private.

## Production artifact inventory

Sixteen deployable Pages files, compared byte-for-byte with the `dist/` output built from source
`4ab9c8e` on 2026-11-28. Status is relative to the previous production commit `953eb3d`; no live
origin measurement exists for this preparation. The two `.wasm` binaries are excluded and remain on
R2.

| Path | Bytes | SHA-256 | Status |
|---|---|---|---|
| `index.html` | 34,259 | `ae16d04760a63d7b0c06b8f95f0ea9df4ac7de77b409a306c15fe279e03cae9a` | modified |
| `privacy.html` | 25,102 | `9f7d3d07bc5a99f35abcf5b2793d29c38f2369912f46fe35fe0773b5de8efe0d` | modified |
| `contact.html` | 16,324 | `35ed51080abcf076cd36c3dc51fa55713fcec386a5f8f617aafa048b536e6ce6` | modified |
| `app.html` | 2,160 | `9565c12dec41b1c5f8c921dd0721c3f382f5e5f53855a8b4ba3fbe1170c17ac9` | modified |
| `404.html` | 10,241 | `3b7ab8d5ee61e94d23fc3e8e646ca39b3a4f9a28b6a8eea5a238157173691fac` | modified |
| `robots.txt` | 270 | `1d3b2168d85434b1abff9032935358b47c6664a7a2c13d84bb0860a6ce9b9b75` | unchanged |
| `sitemap.xml` | 388 | `f16c3ae0dd23ebd3e91442068319628e7c41a17b84fbfa35cb3e0eb52885d34b` | unchanged |
| `og/querymind-social-preview.png` | 64,834 | `9a15f6798382dda1fd3ff9b9003d05880aeb3b7938f98c4a8c7c38b68ec9672c` | unchanged |
| `assets/index-D2cQV5H0.js` | 837,859 | `1b019445e6f49be8dd613f1290c79d5ca2abd7e2ef2775d4d6d085cfdaf3130e` | new |
| `assets/index-BoCwoH1v.css` | 58,279 | `ed26d8c6c2ca757cd38fa6446867e7a6c87781c6ff3e3abd3c2a8c4d35a705df` | unchanged |
| `assets/aistudio/image_ff69bad7.png` | 88,620 | `ff69bad7d2d91321292459a2b54392093e123d0575276a5ee8d7f69bc0cac6dc` | unchanged |
| `duckdb/duckdb-browser-eh.worker.js` | 773,223 | `fa889e6068c40426dea67c08cf16ce0cad7404eae94f6a2522adcabb5898eb93` | unchanged |
| `duckdb/duckdb-browser-mvp.worker.js` | 839,642 | `964f678d3bfa5a23deb154e0a7950634a4d1415e571040ab2d47815dc3f13582` | unchanged |
| `fonts/inter-latin-variable.woff2` | 48,256 | `3100e775e8616cd2611beecfa23a4263d7037586789b43f035236a2e6fbd4c62` | unchanged |
| `fonts/jetbrains-mono-latin-variable.woff2` | 31,432 | `83c005d49d8a6a50474c73a5a36ac0468076e9c4a29da7bdb14995d80560a5be` | unchanged |
| `_headers` | 361 | `4d0c3210169d3e1d8c1b612e4aa581fe315f9ca79be68b7ab8a4e1e6bc7d3f1c` | unchanged |

The 2026-11-25 live verification matched the then-committed artifacts and is recorded above as
history. No live request was made for this preparation, so none of these hashes has been compared
with `https://querymind.in` yet.

Removed in this preparation: `assets/index-BbFiEFSP.js`. No other file was removed. No file was
added except `assets/index-D2cQV5H0.js`.

Repository-level, not referenced by any application route: `.gitattributes` (disables end-of-line
conversion so clones match the artifacts byte-for-byte) and this `README.md`.

## Live browser smoke tests

Performed against `https://querymind.in` on 2026-11-25 with existing browser automation; no package
installation, file change, provider key or private dataset was used. The session viewport was
500 × 772.

- Landing, Privacy, Contact and the 404 page rendered with their expected titles and one `h1`
  each; `/app` mounted without an `h1`. Fonts loaded and no image failed; no horizontal overflow
  was observed at the tested viewport.
- The workspace mounted after skipping the provider dialog and reported the runtime state
  `In-Memory DuckDB`.
- A curated query returned 3 rows × 4 columns in 94 ms, 7 integrity checks, `VERIFIED`.
- A simple custom count returned 25 customers, 1 row, 24 ms, 4 integrity checks.
- An arbitrary free-text question without a configured key was refused with the provider-key
  notice and made no provider request. No real API key was entered.
- Dark theme activation and persistence through reload were verified.
- No uncaught application exceptions were observed. One curated query produced a dataset-capability
  clarification and a console warning (not an error).
- No non-QueryMind third-party host was requested in the observed session. No Cloudflare beacon was
  observed, but hosting-level configuration was not independently inspected.
- Synthetic automation clicks on the theme and dialog-skip controls reported success without
  changing state on the first attempts; DOM-level activation worked and persisted. This is recorded
  as a test-harness interaction anomaly, not an observed application defect.

DuckDB-WASM is fetched inside a worker and does not appear in the page-level network log. R2
serving was verified by direct HTTP requests, and local execution success is the runtime evidence
that the engine loaded.

## Remaining limitations and unverified checks

The following are not verified and must not be treated as completed:

1. The Cloudflare dashboard's own deployment commit SHA field, environment and branch were not
   independently inspected.
2. Search Console ownership, sitemap submission, Google-selected canonicals and indexing status
   remain unverified; no Search Console access was used.
3. Google and LinkedIn social-preview caches were not tested and remain external.
4. A real provider-key query was not executed; only the keyless guard and local demo paths were
   tested.
5. Cloudflare edge analytics-beacon configuration remains not fully verified.
6. `www.querymind.in` DNS was not tested or changed.
7. Desktop-width visual layout was not measured; browser automation used a 500 × 772 viewport.
8. `/app` has a client-mounted `h2` without an `h1`.
9. HSTS, CSP and Permissions-Policy were not observed in the inspected responses. This is a
   hosting observation, not proof of a broken release; no Cloudflare or security configuration was
   changed.
10. The Phase 1 `_headers` file was not confirmed on the live origin: whether Pages applies it to
    404 responses, whether the 308 redirects carry the headers, and the final header values on the
    origin all remain to be measured. CSP is deliberately absent (Phase 2).
11. The artifacts prepared in this transfer are uncommitted and undeployed; every live measurement
    in this README predates them.

Other hosting observations from the live responses: `x-content-type-options: nosniff`,
`referrer-policy: strict-origin-when-cross-origin`, `Access-Control-Allow-Origin: *` on HTML,
NEL/`Report-To` headers and `Server: cloudflare`. HTML is served with
`Cache-Control: public, max-age=0, must-revalidate`; the 404 is `no-store`; hashed assets are
`max-age=14400, must-revalidate`; R2 WASM is one-year immutable.

## Maintenance and future release procedure

- Never edit the generated JS/CSS or rename files — every artifact is content-addressed and
  verified by checksum above.
- To release: build from a verified commit in `query_main`, copy every deployable `dist/` file here,
  remove superseded hashed files, regenerate this README and re-hash, then commit and push. The push
  to `query_prod` `main` is what triggers the Git-connected Cloudflare Pages deployment; no manual
  `wrangler pages deploy` is used.
- After each deployment, verify the live origin against the committed hashes and record only
  measured results in this README. Distinguish the Cloudflare integration check and content
  verification from the provider-side dashboard fields.
- If the DuckDB-WASM package version changes, update `DUCKDB_WASM_VERSION` in
  `src/services/sqlEngine.ts` and upload the new binaries under the new versioned R2 key.
- Keep `404.html` and `app.html` together: with a top-level `404.html`, Pages no longer falls back
  to `index.html` for `/app`.
- Release status must be explicit: a working tree that has not been pushed is a prepared candidate,
  not a deployment. Update the status table only with results measured on the live origin.
- This repository must never contain source code, tests, `node_modules`, `.env` files, keys or logs.