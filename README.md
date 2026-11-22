# QueryMind — Production Deployment

Production build of **QueryMind v1.2** — browser-native natural-language analytics (BYOK, zero backend).

**Release status: prepared and locally committed — not pushed, not deployed.** The live site still serves `7dcdb25` (built from source `b474d69`), which was pushed, deployed and verified on 2026-11-19; that pass and its limits are recorded in the post-deployment section below and in the source repository: `docs/verification/v1.2-live-production-verification.md` (at `query_main` `a49532a` or later). This working tree carries the next release candidate, built from source `2cc8305`; it becomes live only after an authorized push of this repository, which is also what triggers Cloudflare Pages.

Hosted on **Cloudflare Pages** (this repository) with the two oversized DuckDB-WASM binaries served from **Cloudflare R2** (`querymind-static` bucket, custom domain `assets.querymind.in`).

| Field | Value |
|---|---|
| Application | QueryMind |
| Release version | v1.2 (workspace and interaction refinement) |
| Source repository | https://github.com/Shanksreddy005/query_main |
| Source commit | `2cc8305dfa894712b8b0136a38c8bde4b09a8b7b` — "Refine the workspace interface and align documentation with v1.2", on top of `a49532a` and `31bf2cc` (v1.2 verification records) and the previously deployed `b474d69`. Artifacts were generated from a clean clone of this commit, and every file hash below is taken from that build. |
| Build date | 2026-11-22 |
| Build command | `npm ci && npx tsc --noEmit && npm test && npm run build` |
| Test status at build time | 435 tests / 111 suites / 435 pass / 0 fail |
| Deployment status | **Prepared and locally committed — not pushed, not deployed** |
| Currently deployed release | `7dcdb25` (source `b474d69`), deployed and verified 2026-11-19 |

> **The version label identifies a release, not a built-in marker.** The application ships no in-app version string (`package.json` stays at `0.0.0` by design), so the only authoritative identity of what is deployed is the source commit and the artifact hashes below.

> **Not byte-identical to the deployed release (`7dcdb25`, source `b474d69`).** Three files change: `index.html` (asset references and the pre-paint theme reader), the main JS bundle, and the stylesheet. The bundle carries the workspace and interaction refinement (theme support, dialog focus management, query-lifecycle feedback, sortable results table, single-metric layout, chart export contrast); the stylesheet gains the theme tokens, focus-ring and duration utilities, typography scale and targeted reduced-motion rules those changes depend on. `assets/index-BrYHrD16.js` and `assets/index-B5urW9Yt.css` are removed by this rotation. Worker scripts, fonts, the PNG and both `.wasm` binaries are byte-identical to the deployed release, so no R2 re-upload is required.

## Why two hosts

Cloudflare Pages rejects uploads of individual files larger than 25 MiB. Both DuckDB-WASM binaries exceed it, so they are served from the first-party R2 bucket:

| File | Bytes | Served from | R2 object key |
|---|---|---|---|
| `duckdb/duckdb-eh.wasm` | 35,913,747 | `https://assets.querymind.in/duckdb/1.33.1-dev57.0/duckdb-eh.wasm` | `duckdb/1.33.1-dev57.0/duckdb-eh.wasm` |
| `duckdb/duckdb-mvp.wasm` | 41,325,187 | `https://assets.querymind.in/duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` | `duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` |

These two files are **not in this repository**. Upload sources (verify SHA-256 after upload): `query_main/public/duckdb/` (tracked in the source repo) or `query_main/dist/duckdb/`. The R2 objects must be uploaded with:

- **Content-Type:** `application/wasm`
- **Cache-Control:** `public, max-age=31536000, immutable`
- **CORS:** `GET` from `https://querymind.in`, `https://www.querymind.in` and the Cloudflare Pages origin (placeholder `https://<pages-project>.pages.dev` — replace with the actual URL once known); no credentials
- Bucket otherwise private (public access only via the custom domain, static objects only)

## File inventory (SHA-256) — Cloudflare Pages (this repository, 8 files)

| File | SHA-256 |
|---|---|
| `index.html` | `34839cc4cb3646d3248583b250d41dda32ff36aeb92b1356a51f989505dad09b` |
| `assets/index-DKse2nZt.js` | `0c85a6256527f34afa4b7510174d19c2c712abe31ff8d1e9352bc6093d1326d0` |
| `assets/index-CFGGKU9c.css` | `95fed23ce513a0908febdde3a394f1c6b131c70c58dc333d3d7693c2b50d5a01` |
| `assets/aistudio/image_ff69bad7.png` | `ff69bad7d2d91321292459a2b54392093e123d0575276a5ee8d7f69bc0cac6dc` |
| `duckdb/duckdb-browser-eh.worker.js` | `fa889e6068c40426dea67c08cf16ce0cad7404eae94f6a2522adcabb5898eb93` |
| `duckdb/duckdb-browser-mvp.worker.js` | `964f678d3bfa5a23deb154e0a7950634a4d1415e571040ab2d47815dc3f13582` |
| `fonts/inter-latin-variable.woff2` | `3100e775e8616cd2611beecfa23a4263d7037586789b43f035236a2e6fbd4c62` |
| `fonts/jetbrains-mono-latin-variable.woff2` | `83c005d49d8a6a50474c73a5a36ac0468076e9c4a29da7bdb14995d80560a5be` |

## File inventory — Cloudflare R2 (`querymind-static`, 2 objects)

| Object key | SHA-256 | Bytes |
|---|---|---|
| `duckdb/1.33.1-dev57.0/duckdb-eh.wasm` | `3abdec74989dcc54d2f2ea5621f611f3c45db1e7dff2f408476014d82beb2029` | 35,913,747 |
| `duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` | `ee5560145a3d3e0ffa6dce697be802c08842f139a594698eecc7c754f7ad5f05` | 41,325,187 |

## Deployment instructions

1. This repository connects to Cloudflare Pages for Git-based deployment (output = repo root, no build command — files are pre-built).
2. Configure R2 as described above (bucket, two object uploads, `application/wasm`, immutable cache, custom domain `assets.querymind.in`, CORS for the Pages origins) **before** deploying this commit — the app fetches the WASM from R2 on first query.
3. Deploy; the deployment succeeds only because every file here is ≤ 25 MiB.

## Post-deployment verification

Each item below is the standing instruction for a release. The recorded results are the 2026-11-19 pass on the deployed release `7dcdb25` (source `b474d69`); they are kept as historical evidence and **do not verify the locally committed candidate built from `2cc8305`**. Every item must be re-run after an authorized push and deployment, and the new results recorded then.

1. Open the deployed URL — no console errors from the application's own code; the resources the application requests are first-party only: this Pages origin, `assets.querymind.in` (the WASM fetch), and one provider endpoint when a query runs.
   **Result: pass for the application's own requests, with the analytics state left open.** In the cache-bypassing cold loads the page frame requested only the Pages origin for HTML, JS, CSS and both fonts (plus one same-origin `blob:` entry), the served document was byte-identical to `index.html` in this repository, and the live DOM held exactly one script element: the app bundle. No Cloudflare dashboard was opened, so nothing here is a claim about a setting. **What is not clean:** an earlier capture of the same pass showed the edge injecting its own analytics script — a `static.cloudflareinsights.com` script element, a defined `window.__cfBeacon`, and a same-origin `POST /cdn-cgi/rum` from the page — and the pre-release audit had recorded the same chain. That injection is a hosting behaviour, not repository content: none of those identifiers appears in any file of this repository or of the source repository, nor in the shipped bundle text. Because a beacon was seen during the pass and the dashboard was never inspected, the finding stays open; the disablement and re-measurement steps are in `docs/verification/production-quality-remediation-report.md` in the source repository. Separately, a `window.RumSDK` global (with an `ArmsRum` key) was present in the automated browser's page context even in loads where no third-party script was fetched, and could not be attributed to QueryMind: no markup, no bundle string and no request accompanied it. On the console side: zero errors and zero warnings across the cold load and the verified-sample runs; the missing-key guard on an unsupported free-text question with no key writes one expected `console.error` line per rejection (two were logged for two such questions), which is correct refusal behaviour and not a failed query. Cloudflare also answers with `Nel` / `Report-To` headers (`success_fraction: 0.0`), so a browser may report failed connections on this host to Cloudflare; and `http://` answers `301` to `https://`, with no `Strict-Transport-Security`, Content-Security-Policy or `Permissions-Policy` on the measured responses.
2. Run a bundled verified sample query — result shows "QUERY MATCHES REQUEST (n CHECKS PASSED)" and "In-Memory DuckDB"; this proves the cross-origin WASM load worked.
   **Result: pass.** The verified-sample questions run in the pass each carried a checks-passed label (7 or 8 checks), "In-Memory DuckDB", and engine times from 19 ms to 290 ms; the six whose narration was recorded are tabulated in `docs/verification/v1.2-live-production-verification.md` section 5 in the source repository.
3. Confirm the R2 response headers: HTTP 200, `Content-Type: application/wasm`, `Access-Control-Allow-Origin` for the deployed origin.
   **Result: pass.** In-browser `GET` of both objects from the live origin returned 200 with `application/wasm`, `Cache-Control: public, max-age=31536000, immutable`, and the pinned sizes (35,913,747 and 41,325,187 bytes) with the `\0asm` magic; `Access-Control-Allow-Origin` was echoed for the deployed origin and withheld for an unapproved one; a `OPTIONS` preflight declaring only `Access-Control-Request-Method: GET` answered 204 with `Access-Control-Allow-Methods: GET` and `Access-Control-Max-Age: 86400`; a bucket listing request returns 404. Detail worth keeping: the objects do serve ranges to a non-browser client (`Accept-Ranges: bytes`, `206` with a correct `Content-Range`), but a preflight that also declares the `Range` request header is answered 403 with no `Access-Control-*` headers — so a cross-origin range request from a browser would fail, and the application never issues one (the engine uses a plain `GET`). Not a functional defect in this release; recorded so an operator who later wants resumable or partial fetches knows the CORS rule has to allow the `Range` header first.
4. Enter your own provider key in Settings → Test Key succeeds; invalid keys produce a clear provider error.
   **Result: not performed.** No API key was entered during this pass, so neither the valid-key nor the invalid-key provider path was exercised live. This item stays open.
5. Open the Telemetry Auditor — request bodies contain schema metadata only, never dataset rows or results.
   **Result: pass for what the pass could cover.** In keyless mode the auditor held "No Outbound Traffic Yet" with zero recorded dispatches, and no provider endpoint appeared in the network log — so no dataset row and no result left the browser. Because no provider request was made, this does not re-measure provider request bodies; that claim continues to rest on the earlier Gemini and Groq audits and on the automated contract tests.
6. Click "Download Chart as SVG" on a suggested chart and open the saved file: it must be the chart (880×520, labelled), not a toolbar glyph, and must render standalone without the site's stylesheet.
   **Result: pass.** 4,925-byte `image/svg+xml` file: `width="880" height="520"` with a matching `viewBox`, its own `xmlns`, `<title>`, `<desc>` and `aria-label`, 17 text elements (both axis captions plus per-bar value labels), 22 inline `fill` attributes, and no `class`, `<link>`, `<style>`, `@import` or external `url()`. Loaded as a standalone image it reported intrinsic 880 × 520 and rasterized fully painted. The CSV and JSON exports were also checked and produced the correct rows and the question plus executed SQL.
7. With **no** API key configured, run all five E-Commerce demo questions: each must answer locally with a checks-passed badge instead of asking for a key.
   **Result: pass.** All five chips answered locally with checks-passed labels and no key prompt; `localStorage` held no key at any point.
8. Ask a profit-ranked question on the E-Commerce dataset (for example "Top products by total profit") and read the executive summary: the metric it names must be the one the result was ordered on — "ranked by total profit" beside profit figures, never "ranked by total sales". This is the v1.2 fix and it has only been verified against the summary engine in tests so far, so the live pass is what closes it.
   **Result: pass, and one wording correction.** The live check was run with the curated question "What are the top 5 most profitable products in Technology?" (`ORDER BY total_profit DESC`), which narrated "5 orders were ranked by total profit. UltraBook Pro 15 leads with $880…"; the string "ranked by total sales" appeared nowhere, including after a full reload with a cold engine. The quantity-ranked and per-group cases named their own measures too ("ranked by total units sold"). Note that the example question in this item, "Top products by total profit", is **not** one of the curated set: with no key it is correctly refused by the contract guard rather than answered locally, so use one of the five chips (or configure a key) when repeating this check.

## Build provenance

The artifacts above were produced from a clean clone of the source commit, not from a developer working tree:

1. `git clone` of `query_main`, checked out detached at `2cc8305dfa894712b8b0136a38c8bde4b09a8b7b`, working tree clean, no `.env`.
2. `npm ci` → `npx tsc --noEmit` (clean) → `npm test` (435 tests / 111 suites / 0 fail) → `npm run build`.
3. Every file in the clone's `dist/` hashed with SHA-256. The build emitted exactly 10 files: the 8 Pages files above plus the two `.wasm` binaries. No source maps and no env files were emitted.
4. Only the three changed Pages files were copied from the clean clone into this repository — `index.html`, `assets/index-DKse2nZt.js` and `assets/index-CFGGKU9c.css` — and each was re-hashed in place; every value above is the hash of the file actually committed here. The five unchanged Pages files were verified byte-identical to the previously deployed copies and left untouched.
5. Files not copied: `dist/duckdb/duckdb-eh.wasm` and `dist/duckdb/duckdb-mvp.wasm` (35,913,747 and 41,325,187 bytes — both above the Cloudflare Pages per-file limit, both served from R2, both unchanged: they hash to the R2 values above).

## Maintenance rules

- `.gitattributes` in this repository is configuration only (it disables end-of-line conversion so clones match the deployed artifacts byte-for-byte); it is not a deployed asset.
- Never edit the generated JS/CSS or rename files — every artifact is content-addressed and verified by checksum above.
- To update: build from a verified commit in `query_main`, copy the new artifacts here (remove superseded hashed files), update both inventory tables and the source commit SHA, then deploy. If the DuckDB-WASM package version changes, update `DUCKDB_WASM_VERSION` in `src/services/sqlEngine.ts` and upload the new binaries under the new versioned R2 key.
- Release status must be explicit: a commit that has not been pushed is a prepared candidate, not a deployment. Update the status table and the post-deployment section only with results actually measured against the live origin.
- This repository must never contain source code, tests, `node_modules`, `.env` files, keys or logs.
