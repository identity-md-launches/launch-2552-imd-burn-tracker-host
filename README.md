# IMD tokens burned

A static dashboard of IMD burned on Base, Ethereum and Robinhood Chain, with browser-wallet
cards for requesting a page refresh and bridging the BurnExecutor's IMD to Base to burn.
The requested publication name for this update is **burntracker**. The publishing system takes
the finished `dist/` export after submission; this repository does not publish itself.

- `dist/` is the site, served as it is: `index.html`, `fonts.css` (IBM Plex Mono, SIL OFL 1.1),
  `d3.v7.9.0.min.js` (ISC) and `data/*.json`. Its ten dashboard data sources and
  `data/imd-contracts.json` load from the same origin using relative URLs.
- `npm run build` refreshes `dist/data/` from public sources (`node refresh/refresh.mjs dist/data`). It replaces
  the files only when every sanity check passes, and otherwise keeps them and records why in
  `dist/data/imd-refresh-status.json`.
- The build never changes `dist/index.html`, the bundled assets or `dist/data/imd-contracts.json`.
- `refresh/refresh_imd.py` is the original Python refresh (needs `pycryptodome`).

Burn definitions: Base burns = 10,000,000 minted - Base totalSupply + Base burn wallets; Ethereum and
Robinhood Chain = tokens in 0x...dEaD, 0x...0000 and 0x...0001. BurnExecutor share = IMD bridged by the
Ethereum BurnExecutors to the Base burn receiver and burned there. Bridging itself is not burning.

## Install, preview and rebuild

Use Node.js 22 or newer. The application and its production build have no npm dependencies, so
there is no package installation step or required dependency lockfile. The existing manifest
and dependency configuration are preserved.

Preview the committed export through HTTP, rather than opening the HTML as a file:

```sh
python3 -m http.server 8000 --directory dist
```

Open `http://localhost:8000/`. Stop the preview with Ctrl+C. Any static server can serve the same
directory. Relative script, font and data paths also support gateway subpaths; there is no router
or server-side application.

Refresh the snapshot using the repository's production command, with network access:

```sh
npm run build
```

This runs `node refresh/refresh.mjs dist/data`. A source failure retains the prior snapshot and
writes the reason to `dist/data/imd-refresh-status.json`; an exit status of zero alone does not
mean fresh data was obtained. Inspect `last_result`, `message` and `data_as_of` there. The refresh
implementation, source endpoints and optional overrides are documented in `refresh/PORT.md`.

## Exact supplied page

`dist/index.html` and `dist/data/imd-contracts.json` are unchanged copies from
[banse/imd-burn-tracker at 354e7a2](https://github.com/banse/imd-burn-tracker/tree/354e7a2b22692fd2068ab055c0673308d24390ab/dist).
The HTML is also the page's source: styles, charts and wallet logic are inline. No bundler or
generated JavaScript package is needed. The four route cards replace the former aggregate
All chains card; both transaction cards and the BurnExecutor explorer link come from that file.

The dashboard uses local snapshots. Only the transaction cards make external runtime reads:
public Ethereum RPCs and, when a completed version is available, an IPFS gateway. They use the
visitor's injected browser wallet for signing. No private credentials, relay, analytics or
application backend is part of the export. Refresh requires 0.5 IMD plus Ethereum gas; the
executor bridge uses its own IMD and the caller's ETH for LayerZero fees and gas. Availability
depends on current contract state and RPC/gateway responses.

## Validation of this update

- `npm run build` completed successfully and refreshed the public-source data with all sanity
  checks passing. The final snapshot is `2026-10-25T20:23:47Z`; status and summary record success.
- All three inline JavaScript programs and both build modules passed `node --check`.
- `node refresh/check.mjs` passed its existing offline regression suite, including rejection,
  rollback and preservation of prior data.
- Chromium inspected the production export at a static subpath. All 25 interaction checks passed,
  including search, time ranges, keyboard navigation, wallet absence/rejection, Ethereum network
  switching, exact approval, proof-verified refresh, hook sweep, bridge fees and failure states.
  Transaction checks used disposable test fixtures; no real wallet transaction was sent.
- Both themes were checked at 320, 390, 620, 640, 768 and 1440px without document overflow.
- The pinned taste probe passed 8/16 checks. Its remaining findings, five axe rule groups,
  wallet-connector differences and missing favicon/Open Graph metadata are documented in
  [artifacts/validation.md](artifacts/validation.md). The supplied HTML remains unchanged as required.
- A real JavaScript typecheck using TypeScript `--allowJs --checkJs --noEmit` ran and failed with
  118 diagnostics in the supplied untyped page. Syntax and interaction passes do not substitute
  for a passing typecheck. WebKit could not launch because required runtime libraries were
  unavailable; native zoom, physical-device and screen-reader checks were not performed.

The complete build result, integrity checks and review coverage are recorded in
[artifacts/validation.md](artifacts/validation.md). [DESIGN.md](DESIGN.md) records the implemented
tokens, typography, components and responsive behavior. The screenshot artifacts record the
checked export; fixtures, browser packages and caches are excluded from delivery.
