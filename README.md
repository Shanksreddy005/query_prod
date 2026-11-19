# QueryMind — Production Deployment

Production build of **QueryMind v1.2** — browser-native natural-language analytics (BYOK, zero backend).

This HEAD (`v1.2`) is a **release candidate that has not been pushed or deployed**. Cloudflare Pages deploys from the pushed branch, so the live site still serves whatever the last pushed commit contained; nothing here should be read as evidence about `https://querymind.in` until a push and a live verification pass have happened.

Hosted on **Cloudflare Pages** (this repository) with the two oversized DuckDB-WASM binaries served from **Cloudflare R2** (`querymind-static` bucket, custom domain `assets.querymind.in`).

| Field | Value |
|---|---|
| Application | QueryMind |
| Release version | v1.2 (executive-summary ranked-metric fix) |
| Source repository | https://github.com/Shanksreddy005/query_main |
| Source commit | `b474d69de01d3bee0df48a7a9fb169469c169869` — head of "Derive the ranked metric in the executive summary from the executed query", on top of `56c5ac8` (SVG export + keyless demo fixes), `238a392` and `0e3b33f` (documentation and repository hygiene). Artifacts were regenerated from a clean clone of this commit and matched byte-for-byte against the working-tree build. |
| Build date | 2026-11-19 |
| Build command | `npm ci && npx tsc --noEmit && npm test && npm run build` |
| Test status at build time | 349 tests / 91 suites / 349 pass / 0 fail |

> **The version label identifies this deployment, not a built-in marker.** The application ships no in-app version string (`package.json` stays at `0.0.0` by design), so the only authoritative identity of what is deployed is the source commit and the artifact hashes below.

> **Not byte-identical to the previous release (`b1a31ab`).** Two files change: `index.html` (asset reference) and the main JS bundle (the ranked measure is now read from the executed query instead of inferred, so a profit-ranked result is no longer narrated as a sales ranking). `assets/index-BWBgMqJ9.js` is removed by this rotation. CSS, worker scripts, fonts, images and both `.wasm` binaries are byte-identical to the previously verified release, so no R2 re-upload is required.

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
| `index.html` | `e4570160d63375c40750477b91e9d06c2c1e902c5024e1eb9afed0c081c5cf3c` |
| `assets/index-BrYHrD16.js` | `c18f9d601530bc8317c612818b0f4e406919b5af5a2c23984b19bb3d77dabc74` |
| `assets/index-B5urW9Yt.css` | `0578ef8eb7d42c66df4c20e7139d108060ae648a43ca9328a6f5e25ccf28db6f` |
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

1. Open the deployed URL — no console errors; network tab shows first-party resources only: this Pages origin, `assets.querymind.in` (the WASM fetch), and one provider endpoint when a query runs. **Known exception until the operator disables it:** the Cloudflare edge injects its own analytics beacon (`static.cloudflareinsights.com/beacon.min.js` and `POST /cdn-cgi/rum`) into the served HTML. That is a dashboard setting, not application code; see `docs/verification/production-quality-remediation-report.md` in the source repository for the disablement steps, and re-measure after they are done.
2. Run a bundled verified sample query — result shows "QUERY MATCHES REQUEST (n CHECKS PASSED)" and "In-Memory DuckDB"; this proves the cross-origin WASM load worked.
3. Confirm the R2 response headers: HTTP 200, `Content-Type: application/wasm`, `Access-Control-Allow-Origin` for the deployed origin.
4. Enter your own provider key in Settings → Test Key succeeds; invalid keys produce a clear provider error.
5. Open the Telemetry Auditor — request bodies contain schema metadata only, never dataset rows or results.
6. Click "Download Chart as SVG" on a suggested chart and open the saved file: it must be the chart (880×520, labelled), not a toolbar glyph, and must render standalone without the site's stylesheet.
7. With **no** API key configured, run all five E-Commerce demo questions: each must answer locally with a checks-passed badge instead of asking for a key.
8. Ask a profit-ranked question on the E-Commerce dataset (for example "Top products by total profit") and read the executive summary: the metric it names must be the one the result was ordered on — "ranked by total profit" beside profit figures, never "ranked by total sales". This is the v1.2 fix and it has only been verified against the summary engine in tests so far, so the live pass is what closes it.

## Build provenance

The artifacts above were produced from a clean clone of the source commit, not from a developer working tree:

1. `git clone` of `query_main`, checked out detached at `b474d69de01d3bee0df48a7a9fb169469c169869`, working tree clean.
2. `npm ci` → `npx tsc --noEmit` (clean) → `npm test` (349 tests / 91 suites / 0 fail) → `npm run build`.
3. Every file in the clone's `dist/` hashed with SHA-256 and compared to the working-tree build: all 10 files byte-identical.
4. The eight Pages files listed above then copied from the clean clone into this repository, and each re-hashed in place — every value above is the hash of the file actually committed here. The two `.wasm` binaries in the same `dist/` re-hashed to the R2 values above, which is what makes "no R2 re-upload" a measured statement rather than an assumption.
5. Files not copied: `dist/duckdb/duckdb-eh.wasm` and `dist/duckdb/duckdb-mvp.wasm` (35,913,747 and 41,325,187 bytes — both above the Cloudflare Pages per-file limit, both served from R2).

## Maintenance rules

- `.gitattributes` in this repository is configuration only (it disables end-of-line conversion so clones match the deployed artifacts byte-for-byte); it is not a deployed asset.
- Never edit the generated JS/CSS or rename files — every artifact is content-addressed and verified by checksum above.
- To update: build from a verified commit in `query_main`, copy the new artifacts here (remove superseded hashed files), update both inventory tables and the source commit SHA, then deploy. If the DuckDB-WASM package version changes, update `DUCKDB_WASM_VERSION` in `src/services/sqlEngine.ts` and upload the new binaries under the new versioned R2 key.
- This repository must never contain source code, tests, `node_modules`, `.env` files, keys or logs.
