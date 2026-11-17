# QueryMind — Production Deployment

Deployed production build of **QueryMind v1.0** — browser-native natural-language analytics (BYOK, zero backend).

Hosted on **Cloudflare Pages** (this repository) with the two oversized DuckDB-WASM binaries served from **Cloudflare R2** (`querymind-static` bucket, custom domain `assets.querymind.in`).

| Field | Value |
|---|---|
| Application | QueryMind |
| Release version | v1.0 (R2 WASM remediation) |
| Source repository | https://github.com/Shanksreddy005/query_main |
| Source commit | `c5253a7028e97a21e1ea565b1c2d282ead5f2801` — "Serve DuckDB-WASM binaries from first-party R2 (Cloudflare Pages 25 MiB limit)" |
| Build date | 2026-11-17 |
| Build command | `npm ci && npx tsc --noEmit && npm test && npm run build` |
| Test status at build time | 301 tests / 83 suites / 301 pass / 0 fail |

> **Not byte-identical to the previous release (`daffa85`).** The application bundle changed by design: the WASM URL now points at the R2 domain. `index.html` and the main JS changed; CSS, worker scripts, fonts, images and both `.wasm` binaries are byte-identical to the previously verified release. Superseded artifact `assets/index-sn9S7bCm.js` was removed by rotation.

## Why two hosts

Cloudflare Pages rejects uploads of individual files larger than 25 MiB. Both DuckDB-WASM binaries exceed it, so they are served from the first-party R2 bucket:

| File | Bytes | Served from | R2 object key |
|---|---|---|---|
| `duckdb/duckdb-eh.wasm` | 35,913,747 | `https://assets.querymind.in/duckdb/1.33.1-dev57.0/duckdb-eh.wasm` | `duckdb/1.33.1-dev57.0/duckdb-eh.wasm` |
| `duckdb/duckdb-mvp.wasm` | 41,325,187 | `https://assets.querymind.in/duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` | `duckdb/1.33.1-dev57.0/duckdb-mvp.wasm` |

These two files are **not in this repository**. Upload sources (verify SHA-256 after upload): `query_main/public/duckdb/` (tracked in the source repo) or `query_main/dist/duckdb/`. The R2 objects must be uploaded with:

- **Content-Type:** `application/wasm`
- **Cache-Control:** `public, max-age=31536000, immutable`
- **CORS:** `GET` from `https://querymind.ai`, `https://www.querymind.ai` and the Cloudflare Pages origins; no credentials
- Bucket otherwise private (public access only via the custom domain, static objects only)

## File inventory (SHA-256) — Cloudflare Pages (this repository, 8 files)

| File | SHA-256 |
|---|---|
| `index.html` | `e4cd0a0f5d514d044c643d10c868deb37ca34781da81547755619370a22218fa` |
| `assets/index-aKeiN4x3.js` | `7e94d706eea1ab336fc3221a944e9b204be0a896ec82e367113c2aeb05731b5b` |
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

1. Open the deployed URL — no console errors; network tab shows only first-party resources: this Pages origin, `assets.querymind.in` (the WASM fetch), and one provider endpoint when a query runs.
2. Run a bundled verified sample query — result shows "QUERY MATCHES REQUEST (n CHECKS PASSED)" and "In-Memory DuckDB"; this proves the cross-origin WASM load worked.
3. Confirm the R2 response headers: HTTP 200, `Content-Type: application/wasm`, `Access-Control-Allow-Origin` for the deployed origin.
4. Enter your own provider key in Settings → Test Key succeeds; invalid keys produce a clear provider error.
5. Open the Telemetry Auditor — request bodies contain schema metadata only, never dataset rows or results.

## Maintenance rules

- `.gitattributes` in this repository is configuration only (it disables end-of-line conversion so clones match the deployed artifacts byte-for-byte); it is not a deployed asset.
- Never edit the generated JS/CSS or rename files — every artifact is content-addressed and verified by checksum above.
- To update: build from a verified commit in `query_main`, copy the new artifacts here (remove superseded hashed files), update both inventory tables and the source commit SHA, then deploy. If the DuckDB-WASM package version changes, update `DUCKDB_WASM_VERSION` in `src/services/sqlEngine.ts` and upload the new binaries under the new versioned R2 key.
- This repository must never contain source code, tests, `node_modules`, `.env` files, keys or logs.
