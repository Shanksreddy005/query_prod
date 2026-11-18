# QueryMind — Production Deployment

Deployed production build of **QueryMind v1.1** — browser-native natural-language analytics (BYOK, zero backend).

Hosted on **Cloudflare Pages** (this repository) with the two oversized DuckDB-WASM binaries served from **Cloudflare R2** (`querymind-static` bucket, custom domain `assets.querymind.in`).

| Field | Value |
|---|---|
| Application | QueryMind |
| Release version | v1.1 (production quality remediation) |
| Source repository | https://github.com/Shanksreddy005/query_main |
| Source commit | `0e3b33ffd94069a7b93a4d89130d22fc2d0027e0` — head of "Ignore the wrangler local emulation cache directory"; the shipped code fix is `56c5ac897894c866762907c0d4de1bf4784a20a1` and the two commits after it are documentation-only. Artifacts were regenerated from a clean working tree at this commit and matched byte-for-byte. |
| Build date | 2026-11-18 |
| Build command | `npm ci && npx tsc --noEmit && npm test && npm run build` |
| Test status at build time | 330 tests / 86 suites / 330 pass / 0 fail |

> **Not byte-identical to the previous release (`e1d4c59`).** Two files changed: `index.html` (asset references) and the main JS bundle (standalone SVG chart export, the ranking-direction validation fix, three added keyless demo blueprints and corrected keyless copy). `assets/index-aKeiN4x3.js` was removed by rotation. CSS, worker scripts, fonts, images and both `.wasm` binaries are byte-identical to the previously verified release, so no R2 re-upload is required.

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
| `index.html` | `aff3b3a5ceb83951501dafec5e9df746158b5268f6dbd69a2fd0512fa5142945` |
| `assets/index-BWBgMqJ9.js` | `7eb989b5d82933dd5b60c147df80ca2c6280b25e10836a64206387e074ed20d3` |
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

## Maintenance rules

- `.gitattributes` in this repository is configuration only (it disables end-of-line conversion so clones match the deployed artifacts byte-for-byte); it is not a deployed asset.
- Never edit the generated JS/CSS or rename files — every artifact is content-addressed and verified by checksum above.
- To update: build from a verified commit in `query_main`, copy the new artifacts here (remove superseded hashed files), update both inventory tables and the source commit SHA, then deploy. If the DuckDB-WASM package version changes, update `DUCKDB_WASM_VERSION` in `src/services/sqlEngine.ts` and upload the new binaries under the new versioned R2 key.
- This repository must never contain source code, tests, `node_modules`, `.env` files, keys or logs.
