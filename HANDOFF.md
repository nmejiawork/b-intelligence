# B Intelligence — Handoff

## What This Is

`b-intelligence.vercel.app` is a single-page React app (no build step) that shows YouTube channel analytics for the B Content suite. It lives as a static `index.html` deployed directly to Vercel.

The app uses Babel Standalone (`<script type="text/babel">`) to transpile JSX in-browser at runtime — there is no build pipeline.

---

## Bug Fixed (2026-03-11)

**Symptom:** The page rendered completely black / blank.

**Root cause:** A raw newline byte (`0x0a`) was embedded inside a JavaScript string literal in the inline Babel script:

```js
// BROKEN — literal newline inside single-quoted string → SyntaxError
.join('
')

// FIXED — escape sequence
.join('\n')
```

This caused `SyntaxError: Unterminated string constant` during Babel transpilation, which crashed the entire React app before it could mount. Nothing rendered.

**Fix:** 1-byte change at file offset ~53436. The `index.html` in this repo already has the fix applied.

---

## Deployment

This project deploys as a **static file** — no framework, no build command.

```
Vercel project: b-intelligence
Production URL: https://b-intelligence.vercel.app
```

`vercel.json` config:

```json
{
  "builds": [{ "src": "index.html", "use": "@vercel/static" }],
  "routes": [{ "src": "/(.*)", "dest": "/index.html" }]
}
```

To redeploy manually, use the Vercel CLI:

```bash
cd /path/to/b-intelligence
npx vercel --prod
```

Or via the Vercel dashboard: push to `main` and it will auto-deploy if the GitHub integration is connected (currently it is not — deploys have been done via API).

---

## Architecture

```
index.html          ← entire app (HTML + inline CSS + inline JSX via Babel)
vercel.json         ← static hosting config
```

No `package.json`, no node_modules, no build step. Everything is self-contained in `index.html`.

Key dependencies loaded from CDN at runtime:
- React 18 (UMD)
- Babel Standalone (for JSX transpilation)
- Chart.js (for analytics charts)

---

## Known Caveats

- **No Vercel GitHub integration** — deploys are done manually (via CLI or API). To wire up auto-deploy: go to Vercel dashboard → b-intelligence project → Settings → Git → connect `nmejiawork/b-intelligence`.
- **Local file was out of sync** — the `index.html` in this repo was the broken version. It has been updated to match production as of this commit.
- **Inline Babel scripts have strict string literal rules** — any future edits to JSX strings must use escape sequences (`\n`, `\t`, etc.), never raw special characters.
