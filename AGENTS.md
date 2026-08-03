# AGENTS.md

## Cursor Cloud specific instructions

ZAP (ZCL Advanced Platform) is a Quasar/Vue + Electron desktop app with a Node.js
backend that doubles as a CLI and an HTTP/REST server, plus a code-generation engine.
Standard developer commands live in `package.json` scripts and
`docs/development-instructions.md` / `docs/faq.md`; the notes below only cover
non-obvious, environment-specific gotchas.

### Node version (important)

- ZAP only supports Node **v14/16/18/20** (see `versionsCheck()` in
  `src-electron/util/env.js`). Node 22 fails that check and makes
  `test/env.test.js` "Versions check" fail, so the project must run on **Node 20**.
- The VM's default `node` (`/exec-daemon/node`) is Node 22 and sits early on `PATH`.
  Node 20 (installed via nvm) is symlinked into `/usr/local/cargo/bin`
  (`node`/`npm`/`npx`), which is the first entry on `PATH`, so `node`/`npm` resolve
  to v20 in every shell. If `node --version` ever reports v22, run `hash -r`; if the
  symlinks are missing, recreate them pointing at `~/.nvm/versions/node/v20.20.2/bin`.
- Because native modules (`sqlite3`, etc.) are compiled against the Node 20 ABI, do
  not build/install under Node 22 and then run under Node 20 (or vice versa).

### Dependencies / build

- The update script runs `npm ci` (system libs such as cairo/pixman/pango/jpeg/gif,
  `xvfb`, and `libxml2-utils` are already installed in the environment).
- `canvas` is NOT an installed dependency in this version; the
  `npm rebuild canvas --update-binary` inside `postinstall` is a harmless no-op.
- Build the SPA before running the server or app: `npm run build-spa`
  (full build is `npm run build`).

### Running the app (headless VM)

- The Electron GUI (`npm run zap`) needs a display and is not the easiest to drive here.
- Prefer the web/server mode: `npm run zap-devserver` (Zigbee, serves the SPA + REST on
  http://localhost:9070/) or `npm run server` (Zigbee + Matter). Open
  `http://localhost:9070/` in a browser — the SPA connects to the REST API on the same
  origin, so no `?restPort=` query param is needed.

### Lint / test

- Lint: `npm run lint`.
- Unit tests: `npm run test:unit` (jest). Known flake: `test/server-bare.test.js` can
  fail at the _suite_ level during the full parallel run with a jest-worker
  `TypeError: Converting circular structure to JSON` IPC error (an Express `req/res`
  object leaking into the worker result). All individual tests pass; the suite passes
  reliably when run in isolation (`npx jest test/server-bare.test.js`) or with
  `--runInBand`. This is unrelated to environment setup.
- E2E (Cypress) tests exist (`npm run test:e2e-ci`, etc.) and require `xvfb` + a Chrome
  binary path argument, as in `.github/workflows/release.yml`.
