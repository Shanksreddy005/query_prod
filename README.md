# QueryMind — Production Deployment

Deployed production build of **QueryMind v1.0** — browser-native natural-language analytics (BYOK, zero backend).

| Field | Value |
|---|---|
| Application | QueryMind |
| Release version | v1.0 |
| Source repository | https://github.com/Shanksreddy005/query_main |
| Source commit | `15ad92e8552b5582cdc9c79506519ae54a02e9fa` |
| Verified release commit (build content) | `8e6483ff66dc4962dd2748666af6518c3f09bedb` — artifacts byte-identical to the build verified at this commit |
| Build date | 2026-11-17 |
| Build command | `npm ci && npx tsc --noEmit && npm test && npm run build` |
| Test status at build time | 301 tests / 83 suites / 301 pass / 0 fail |

The `15ad92e` commit above `8e6483f` is documentation-only (adds `README.md` and `docs/`); a rebuild comparison confirmed the generated artifacts are byte-identical to the verified build at `8e6483f`.

## File inventory (SHA-256)

| File | SHA-256 |
|---|---|
| `index.html` | `e1f2af141631be5be58bfd23bfceee0f863050443664f0dc6a464ec197148ff7` |
| `assets/index-sn9S7bCm.js` | `8c03ef43ed796a1d09b8dc41f03a957aa81614d195cca5d5fe3d4d69826f41b5` |
| `assets/index-B5urW9Yt.css` | `0578ef8eb7d42c66df4c20e7139d108060ae648a43ca9328a6f5e25ccf28db6f` |
| `assets/aistudio/image_ff69bad7.png` | `ff69bad7d2d91321292459a2b54392093e123d0575276a5ee8d7f69bc0cac6dc` |
| `duckdb/duckdb-eh.wasm` | `3abdec74989dcc54d2f2ea5621f611f3c45db1e7dff2f408476014d82beb2029` |
| `duckdb/duckdb-mvp.wasm` | `ee5560145a3d3e0ffa6dce697be802c08842f139a594698eecc7c754f7ad5f05` |
| `duckdb/duckdb-browser-eh.worker.js` | `fa889e6068c40426dea67c08cf16ce0cad7404eae94f6a2522adcabb5898eb93` |
| `duckdb/duckdb-browser-mvp.worker.js` | `964f678d3bfa5a23deb154e0a7950634a4d1415e571040ab2d47815dc3f13582` |
| `fonts/inter-latin-variable.woff2` | `3100e775e8616cd2611beecfa23a4263d7037586789b43f035236a2e6fbd4c62` |
| `fonts/jetbrains-mono-latin-variable.woff2` | `83c005d49d8a6a50474c73a5a36ac0468076e9c4a29da7bdb14995d80560a5be` |

## Deployment instructions

This repository contains only generated static assets — upload the **contents** (not a nested folder) to any static host:

1. **Netlify** — drag-and-drop the folder, or `npx netlify-cli deploy --dir .`
2. **Vercel** — deploy as "Other" with output directory `.`
3. **GitHub Pages** — serve from the repo root (custom domain or user site; the app uses root-relative asset paths)
4. **nginx / object storage** — serve `.wasm` as `application/wasm`; `index.html` with `no-cache`; hashed `assets/*` immutable

Notes: no rewrite rules needed (single page, no client-side routing); Cloudflare Pages is unsuitable (25 MiB per-file limit vs the 39.4 MB DuckDB-WASM binary); DuckDB-WASM's multi-threaded variant requires `Cross-Origin-Opener-Policy: same-origin` + `Cross-Origin-Embedder-Policy: require-corp` (safe here — all assets are same-origin); the single-threaded variant works without special headers.

## Post-deployment verification

1. Open the deployed URL — no console errors; network tab shows only same-origin resources (plus one provider endpoint when a query runs).
2. Run a bundled verified sample query — result shows "QUERY MATCHES REQUEST (n CHECKS PASSED)".
3. Enter your own provider key in Settings → Test Key succeeds; invalid keys produce a clear provider error.
4. Open the Telemetry Auditor — request bodies contain schema metadata only, never dataset rows or results.

## Maintenance rules

- Never edit the generated JS/CSS or rename files — every artifact is content-addressed and verified by checksum above.
- To update: build from a verified commit in `query_main`, replace all files here atomically, update the inventory table and commit SHA, then deploy.
- This repository must never contain source code, tests, `node_modules`, `.env` files, keys or logs.
