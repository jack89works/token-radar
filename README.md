# Token Launch Radar (token-radar-01)

A dashboard that ranks pre-token protocols by what their token launch would be
worth, built for make.fun's "Build the best token launch radar" bounty.

## What it does

`main.ts` (Node, standard library only) reads the sealed snapshot at
`/input/snapshot.json` and writes:

- `/output/board.json` — 220 ranked rows: every protocol on the snapshot's
  watchlist (40) plus 180 early pre-token protocols below the $50M floor.
  Each row carries `implied_value_usd` and a seven-step `farm_steps` plan.
- `/output/data.js` plus a copy of `dashboard/` — a static page with a
  sortable, filterable board, a per-protocol detail view, a live-token view
  set apart from the ranking, and the model stated in full.
- `/output/screenshots/` — board, detail and filtered views of the page.
- `/output/dossier.json` — written with `buildDossier` from
  `vendor/makefun-board.ts`, byte-identical to what the scorer rebuilds.

## The model (radar-2)

No protocol name is special-cased and no multiple is hand-written:

    implied = round( tvl_usd x category_multiple
                     x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
                     x (0.6 + min(age_days, 1095) / 1095 x 0.4) )

`category_multiple` is read from the snapshot itself: the median
market-cap / TVL of the live-token protocols in the same category, clamped
to [0.05, 1.50] and shrunk toward the global median (x0.169) by the peer
count, weight n / (n + 5). A category with no live peers uses the global
median. Every row records its multiple, its category's raw median and its
peer count, so any number on the board can be audited back to the snapshot.

A protocol counts as pre-token when its record carries no token (no market
cap, no CoinGecko id, no symbol). The board excludes CEX and versioned-pool
names from the early rows; the watchlist itself is always covered in full.

## Farm plans

Each plan is derived from the protocol's own record (category, chains, TVL,
age): a category-specific main action and second action, a weekly qualifying
action, a hold of 30–60 days by age, a Sunday points check, and a closing
review step that states the plan's whole gas budget (about $26–$34) next to
its working capital ($50–$250 by TVL tier).

## Run it locally

    mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input input/snapshot.json

The program is deterministic: two runs on the same snapshot write the same
bytes. The dashboard renders with no network; open `out/dashboard/index.html`.

## Layout

- `main.ts` — entrypoint: snapshot in, board + dashboard + dossier out.
- `lib/model.ts` — the scoring model, the farm-plan derivation, validation.
- `dashboard/` — `index.html`, `app.css`, `app.js` (the page recomputes each
  row's implied value from the same formula and checks it against the board).
- `vendor/makefun-board.ts` — the kit's dossier library, vendored unchanged.
- `screenshots/` — the three views the judge sees first.
