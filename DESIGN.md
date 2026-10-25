# IMD Burn tracker design

## Overview

This dashboard lets IMD holders inspect burn snapshots across Base, Ethereum and Robinhood Chain and use their browser wallet to request a refresh or bridge the executor's IMD to the Base burn receiver. The supplied page is the source and the static export: `dist/index.html`, from upstream commit `354e7a2b22692fd2068ab055c0673308d24390ab`. It is preserved byte for byte for this assignment.

The existing visual system is monochrome, with IBM Plex Mono, square panels, fine rules, compact data labels and an inverted summary panel. The hierarchy is summary, route totals, charts and tables, then the two transaction cards. These are implemented facts, including departures from the generic design guide; unresolved findings are in `artifacts/validation.md`.

## Colors

Global tokens are in `dist/index.html:12`; component aliases begin at `dist/index.html:374`.

| Token | Dark | Light | Role |
| --- | --- | --- | --- |
| `--color-bg` | `#000` | `#fff` | Page and panel paper |
| `--color-fg` | `#fff` | `#000` | Ink, borders and primary text |
| `--color-fg-muted` | `#9a9a9a` | `#555` | Notes, secondary labels and status text |
| `--color-bad` | `#ff6b62` | `#b3261e` | Data-load failure dot |
| `--ib-ink`, `--ib-paper`, `--ib-dim` | Aliases of foreground, background and muted foreground | Same aliases | Component colors |
| `--ib-c0` | Ink | Ink | Primary chart series |
| `--ib-c1` | 52% foreground mixed with background | Same formula | Secondary chart series |
| `--ib-c2` | 24% foreground mixed with background | Same formula | Third chart series |
| `--ib-soft` | 13% foreground mixed with background | Same formula | Gridlines, separators and unselected chart background |

The default is dark. `prefers-color-scheme: light` selects light unless `data-theme="dark"` is set on the document; explicit `data-theme="light"` and `data-theme="dark"` also work. There is no theme-switch button.

The summary `.ib-wall` inverts ink and paper. Its muted text uses `--ib-inv-dim`, mixing 62% background with foreground; remaining supply uses a 16% mix. Primary enabled `.ib-btn-primary` buttons also invert ink and paper. Chart categories additionally have text labels and numeric data; errors use text and underlining.

Measured from browser-computed RGB pairs: body ink against paper is **21.00:1** in both themes; muted text against paper is **7.46:1** in both themes. These measurements do not cover every SVG, opacity, focus or transient state.

## Typography

`dist/fonts.css` embeds WOFF2 IBM Plex Mono, normal styles at weights 400, 500 and 600, with `font-display: swap`. Browser inspection confirmed weights 400 and 600 loaded; 500 is declared but was unused in the checked page. The fallback stack is `ui-monospace, SFMono-Regular, Menlo, monospace`, via `--ib-mono`.

| Role/class | Size | Weight | Other rules |
| --- | --- | --- | --- |
| `.ib` body | 13px | 400 | Unitless line-height 1.55; tabular numerals |
| `.ib-title` | `clamp(22px, 3.4vw, 28px)` | 600 | Line-height 1.15; tracking -0.03em |
| `.ib-hero-num` | `clamp(34px, 5.2vw, 56px)` | 600 | Line-height 1.05; tracking -0.03em |
| `.ib-big` data totals | `clamp(22px, 2.6vw, 28px)` | 600 | Line-height 1.15; tracking -0.03em |
| `.ib-h` section heading | 12px | 600 | Uppercase; tracking 0.1em |
| `.ib-label` | 10.5px | 600 | Uppercase; tracking 0.1em inherited |
| `.ib-h3`, `.ib-foot`, `.ib-asof`, chart axes | 11px | Varies by role | Secondary heading tracking 0.1em; footer natural case |
| `.ib-note`, `.ib-rf-row`, `.ib-rf-note` | 11.5px | 400 | Wallet rows line-height 1.5; values wrap anywhere |
| `.ib-sub`, `.ib-search` | 12px | 400 | Search input remains 12px on mobile |
| `.ib-table`, `.ib-rf-lead` | 12.5px | 400 | Tables use tabular numerals |
| `.ib-btn` | 10.5px | 400 | Uppercase; tracking 0.06em |

The summary description has a 62ch measure and `text-wrap: pretty`; section descriptions use 80ch and transaction leads 70ch. Chart labels may shorten with an ellipsis and expose full text on pointer hover. The supplied source uses more than six sizes and small tracked capitals; this document records that existing system rather than claiming guide conformance.

## Layout

`#dash-root` is at most 1240px wide, centered, with `clamp(16px, 4vw, 34px)` padding and border-box sizing. `.ib` is a grid with 18px gaps. Panel interiors use 20px horizontal padding, and KPI/split grids use 14px gaps.

The summary uses `minmax(0, 1fr) auto` columns with 20px by 36px gaps and `clamp(20px, 4vw, 34px)` padding. At 620px and below it becomes one column; its logo and legend then sit alongside each other, with a 120px logo.

KPI columns auto-fit with a minimum of `min(100%, 12rem)`; outlook panels use 18rem and supply/split panels use 14rem. Transaction readouts use auto-fit columns of at least `min(100%, 18rem)`, with 6px by 24px gaps. Transaction buttons wrap with 8px gaps.

At 640px and below `.ib-hide` removes secondary table columns. Tables retain local horizontal scrolling where necessary. `.ib-scroll` tables have a maximum height of 460px, local scrolling and sticky headings. The search field is at most 22rem wide and 32px tall. Charts size themselves from their container width and redraw through `ResizeObserver`.

Chromium checked 320, 390, 620, 640, 768 and 1440px in both themes without document-level horizontal overflow. The 390×844 first screen shows the product, total and `#refresh` anchor. Wallet controls remain in the bottom cards. Native 200% zoom, physical devices and RTL mirroring were not verified.

## Elevation & Depth

The interface is flat. `--ib-rule` is `1.5px solid var(--ib-ink)` and defines panels, heading separators and controls. Internal table and split-panel dividers use 1px soft rules. There are no card shadows. Tooltips have explicit `box-shadow: none`, paper background, ink border and `z-index: 2`; sticky table headings use `z-index: 1`.

## Shapes

Panels, buttons, inputs, chart swatches and tooltips have square corners (`border-radius: 0`). Chart stacks are 14px tall and clipped to their frame. The data-error indicator is a 6px circle. The supply logo is an inline SVG, not an external image dependency.

## Components

These are HTML/CSS and inline JavaScript patterns, not framework exports.

| Pattern | Source | Behavior |
| --- | --- | --- |
| Summary `.ib-wall` | `dist/index.html:90` | Product name, total, snapshot, refresh anchor and supply logo; skeletons while data loads |
| KPI `.ib-card.ib-kpi` | `dist/index.html:111` | Route label, large amount, contextual note; no aggregate All chains card |
| History `.ib-seg` | `dist/index.html:152` | Native All / 12 months / 90 days buttons; active selection uses `aria-pressed` and inverted fill |
| Search `.ib-search` | `dist/index.html:228` | Native searchbox named “Search launched tokens”; filters name, symbol, address and type; zero matches produces an empty table body |
| Tables `.ib-tablewrap`, `.ib-scroll` | `dist/index.html:464` | Numeric columns align right; hover rows use a 5% ink mix; secondary columns hide on narrow screens |
| Refresh `#refresh` | `dist/index.html:334` | Public tracker state, hash-verified IPFS record and wallet state; connect, exact token approval, then refresh transaction |
| Executor burn `#burn` | `dist/index.html:350` | Public executor/hook state, reviewed destination, fee and wallet checks; optional sweep and bridge action |
| Status `.ib-rf-note` | `dist/index.html:329` | Inline wallet errors and transaction progress; not a live region in this source |
| Tooltip `.ib-tip` | `dist/index.html:462` | Pointer-following chart detail; pointer-events disabled; no keyboard equivalent implemented |

Native buttons and links support keyboard activation. The search field has an explicit 2px focus outline; other controls retain browser focus indicators. The checked refresh link showed a visible native indicator. Controls use native `disabled` during unavailable/pending actions; muted labels distinguish unavailable states. There is no connect sheet or disconnect control: EIP-6963 selects the first announced provider, falling back to `window.ethereum`. Wallet limitations are documented in validation.

The loading skeleton pulses only under `prefers-reduced-motion: no-preference`. The logo rise uses the same preference guard and a 1.4s animation. Stack hover changes opacity over 0.2s; refreshing data opacity uses `--cds-dur-slow: .3s`. There is no `transition: all`.

## Do's and Don'ts

- Reuse the existing semantic color aliases and relative asset paths. New content should use `.ib-card`, `.ib-h`, `.ib-sub` and the existing grid patterns when a future assignment permits edits.
- Use native buttons for actions and links for destinations. Keep fee, payer, chain and burn consequences beside wallet controls.
- Treat displayed snapshots separately from live card reads. The build refreshes snapshot JSON; the browser cards query the configured public RPCs.
- Preserve contract addresses, ABIs, token amounts and proof checks unless a future task explicitly changes them. Runtime contract configuration is `dist/data/imd-contracts.json`.
- To add another static page in an authorized follow-up, reuse `fonts.css`, theme tokens, `.ib` spacing and panel patterns, keep URLs relative, and recheck both themes and narrow widths. This task does not add another page or modify the pinned HTML.

Review method: Jakub Krehel's [Better Interface](https://github.com/jakubkrehel/skills/tree/267330e1adfc66a718fb65fa6918c1f06d0a689e/skills/better-interface) (MIT); documentation method: Paul Bakaus's [Impeccable](https://github.com/pbakaus/impeccable/blob/9d715cc4f5564a990ca8345abfdd5df6dc9b41c8/skill/reference/document.md) (Apache-2.0). This is an implementation-specific record, not a copy of either guide.
