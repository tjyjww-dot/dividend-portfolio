# CLAUDE.md

Guidance for AI assistants (Claude Code) working in this repository.

## TL;DR

- This repo is the **published dashboard** for a Korean dividend-stock auto-trading
  system (배당주 자동매매 포트폴리오). It is served as a static site via **GitHub Pages
  from the `docs/` folder**.
- The repo contains essentially **one meaningful file**: `docs/index.html` — a single,
  self-contained dashboard (HTML + CSS + vanilla JS + inline data).
- **The trading engine itself (Python + Korea Investment Securities "KIS" API) is NOT in
  this repository.** Only its generated HTML output is committed here.
- `docs/index.html` is **auto-generated and committed daily** by that external program
  (commit messages: `Update portfolio: YYYY-MM-DD HH:MM`).
- ⚠️ **The committed `docs/index.html` is currently broken**: it contains hundreds of
  unresolved Git merge-conflict markers. See [Known issue](#known-issue-committed-merge-conflicts).

## Repository layout

```
.
├── .gitignore          # ignores secrets (.env, data/kis_token.json) + Python/IDE/OS junk
├── docs/
│   └── index.html      # the entire dashboard — generated artifact, served by GitHub Pages
└── CLAUDE.md           # this file
```

There is no build system, package manager, test suite, CI workflow, or source code for the
trading logic in this repo. Do not expect `package.json`, a Python project, or `.github/`
workflows — they live wherever the trading engine runs (a private machine), not here.

## What this project is

A dashboard that visualizes an automated US-dividend-stock investment strategy. The UI is in
**Korean**. The strategy (documented in the in-page "투자 로직" modal) is:

- **Asset split:** Core 55% (SCHD ETF) / Satellite 35% (individual dividend stocks) /
  Cash buffer 10% (SGOV ETF).
- **Daily DCA:** invests ~9% of remaining cash each day (dollar-cost averaging).
- **AI scoring:** screens S&P 500 + NASDAQ 100 (~550 tickers, scraped from Wikipedia,
  refreshed every 7 days). Score weights — Safety 40%, Growth 30%, Value 20%, Timing 10%.
  Only tickers scoring **≥ 80** become buy candidates.
- **Fear & Greed weighting:** the CNN Fear & Greed index scales the daily buy size
  (extreme fear → buy 2–3×; extreme greed → buy 0×).
- **Execution:** runs nightly ~23:59 KST, places limit orders via the KIS API, max ~$500
  per order, US market hours only.

This context is descriptive (it explains the data you'll see). The repo only renders it; it
does not execute any of it.

## How `docs/index.html` is structured

Single file, dark theme, no external assets except **Plotly.js 2.27.0 via CDN**. Top-level
regions, in order:

1. **`<style>`** — all CSS inline.
2. **Header** — Fear & Greed badge, buy-weight multiplier, "투자 로직" (investment logic)
   button, "열람 전용" (read-only) badge.
3. **`#logicModal`** — the strategy explainer modal (mostly static copy).
4. **Stat cards** (`.stat-card`) — 총 자산가치 (total value), 예수금 (cash), 누적 배당금
   (cumulative dividends), 총 투입 원금 (total invested), etc.
5. **Charts** (Plotly): `#allocationChart` (asset-allocation pie) and `#historyChart`
   (value-over-time line).
6. **Tables**: 보유 종목 현황 (holdings), 매수 후보 종목 (buy candidates ≥80),
   거래 내역 (recent trades, last 20).
7. **`<script>`** — `Plotly.newPlot(...)` calls plus inline data such as
   `var history = [ {date, total_value, invested, fg_index, note?}, ... ];`.

Data is **embedded directly in the HTML** (rendered table rows + inline JS arrays). There is
no runtime data fetch; each daily regeneration rewrites these values.

## Known issue: committed merge conflicts

`docs/index.html` currently contains **hundreds of unresolved Git conflict markers**
(`<<<<<<< HEAD`, `=======`, `>>>>>>> <hash>`), nested many layers deep. They are *committed*
into the file (the working tree is clean), so the live page is corrupted.

Root cause: the automated daily "Update portfolio" process appears to merge/commit without
resolving conflicts, accreting a new conflict layer each day.

If asked to **fix/clean** the file:

- In each conflict, the **topmost `HEAD` value is the newest/correct one** (it corresponds to
  the most recent daily run). Verify against the inline `history` array — its last element
  has the latest `date` and matching `fg_index` / `total_value`.
- Resolution = keep the first (HEAD) side of every conflict and delete every
  `<<<<<<<`, `=======`, `>>>>>>>` marker plus the alternative sides.
- Because conflicts are deeply nested, a careful scripted pass (or regenerating the file from
  the source engine) is more reliable than hand-editing 360+ blocks. After any automated
  cleanup, sanity-check that the HTML still parses and that
  `grep -c '<<<<<<<\|>>>>>>>\|^=======' docs/index.html` returns 0.
- Do **not** invent new portfolio numbers — only ever keep values that already exist in the
  file. This is a real financial dashboard; fabricating data is worse than leaving a conflict.

The real long-term fix belongs in the external generator (commit a clean file, not a merge),
which is outside this repo.

## Conventions & workflow for assistants

- **Language:** UI text, labels, and commit messages are Korean. Match existing tone when
  editing user-facing strings; keep ticker symbols and code in English/ASCII.
- **Single file:** treat `docs/index.html` as the deliverable. Keep it self-contained
  (inline CSS/JS, Plotly via CDN) — don't split it into separate assets unless asked.
- **No secrets:** never add API keys, tokens, or account data. `.env` and
  `data/kis_token.json` are gitignored and must stay out of the repo. If you ever see real
  credentials in a diff, stop and flag it.
- **Generated artifact:** remember the file is normally machine-generated. Manual edits will
  be overwritten by the next daily run unless the generator is also changed (which you can't
  do from here). Call this out when relevant.
- **Verifying changes:** open `docs/index.html` in a browser (it needs internet for the
  Plotly CDN). There are no tests or linters to run.

## Git / PR workflow

- Develop on the branch you were assigned (e.g. `claude/...`); create it locally if missing.
- Commit with clear messages. Note that the repo's automated history uses
  `Update portfolio: <timestamp>` — use a descriptive message for human/AI changes instead so
  they're distinguishable.
- Push with `git push -u origin <branch>`; after pushing, open a **draft PR** if none exists.
- Never push to `main` or another branch without explicit permission.
