# Roadmap — Voice Board + Pack Leftovers: Priority Matrix & Fanout TDD Spec
Date: 2026-09-12. Rule: every item ships behind a failing check first (red → green → refactor). All checks run local-light (python3/node/sips) — nothing here needs the build box.

## Priority matrix

| Pri | Item | Why this rank | Effort | Lane |
|---|---|---|---|---|
| P0 | Verify 4 lane-reported ban threads (2× r/GMail, 1× promo-revocation, 1× CLI+MCP suspension) | Report currently cites them as leads; upgrading to verified-or-dropped decides credibility | ~30 min | verify |
| P0 | Crop signup banner (~120px) off both issue screenshots | Posting-ready assets; blocks the whole social launch | ~10 min | assets |
| P0 | Log-zero proper fix (c-collapse): replace $0.06 floor hack with ECharts `markPoint`/annotation at axis + tooltip note | Only remaining chart that misrenders data; credibility bug | ~30 min | charts |
| P1 | §02 funding-timeline mini bar ($8M seed Oct-25 → $13M Series A Jul-26) | Closes "capital" narrative gap; reuses receipts | ~30 min | charts |
| P1 | §05 TTFB bar chart (6 vendors, ms, log axis) | Only section with zero visuals; matrix deserves a picture | ~30 min | charts |
| P1 | Print stylesheet + OG/meta tags + author line | Cheap craftsmanship; unblocks sharing/printing | ~20 min | chrome |
| P2 | Refresh-date logic + dark-mode snapshot re-verify | Hygiene; do on next content refresh, not now | ~20 min | chrome |
| WONTFIX (moot) | Gemini shot-1 retry | User-supplied comic beats diffusion output; quota spend unjustified | — | — |

## Module boundaries (one lane owns one region — no shared files except index.html)

- **M-charts**: ECharts option builders only (`c-collapse` fix, `c-funding` new, `c-ttfb` new). Pure data-in/option-out thinking; tests assert on option JSON.
- **M-chrome**: `<head>` meta/OG, print CSS, TOC/SVG already shipped — only additive tweaks. Tests assert on markup strings.
- **M-verify**: `account-risk-and-issue-precedents.md` only. Contract per claim: `{url, title, date, tier}` where tier ∈ verified/dropped. A claim that can't be opened is deleted, never kept as "reported".
- **M-assets**: `social-pack/*.png` only, via macOS `sips` (light, local-safe). Tests assert pixel dimensions.

## Merge protocol (kills the multi-editor conflict class)

1. Lanes never edit `index.html` concurrently. Each lane returns a **patch block** keyed by a unique anchor string + the exact `oldString`/`newString`.
2. Integrator applies blocks sequentially in P-order, one `edit` per block.
3. Gate after every block: `python3` HTML parse (zero unclosed/mismatched) + `node --check` on the inline script + anchor inventory (`s01–s08`, `c-*`, `f3/f4`). Any red gate stops the line — no batching past a failure.
4. Final gate: full-file byte count + visual spot-check of touched chart regions.

## TDD checks (write these BEFORE the fix)

```bash
# M-charts: Kokoro series must carry an annotation, not a fake floor value
node -e "const o=require('./c-collapse.option.json'); assert(o.series.find(s=>s.name.includes('Kokoro')).markPoint, 'needs markPoint not floor hack')"
# M-charts: funding chart anchors
node -e "const o=require('./c-funding.option.json'); assert.deepEqual(o.xAxis.data,['Seed Oct-25','Series A Jul-26']); assert.deepEqual(o.series[0].data,[8,13])"
# M-verify: every cited thread opens with matching title (script fetches + compares, tier flips to verified only on match)
python3 verify_threads.py  # exits non-zero on any mismatch; report keeps zero lane-reported rows
# M-assets: banner crop = exactly 120px off height, width untouched
sips -g pixelWidth -g pixelHeight issue-1001-confession.png  # before/after diff asserts Δh==120, Δw==0
```

## Fanout map (3 parallel lanes, then integrate)

```
main (integrator: applies + gates)
├── lane-charts  → patch blocks for c-collapse fix + c-funding + c-ttfb (P0+P1)
├── lane-verify  → updated precedents.md, zero lane-reported rows (P0)
└── lane-assets  → cropped PNGs + dim assertions (P0)
M-chrome (P1/P2) rides with whichever lane finishes first — head-only edits, no conflicts.
```

Suggested fire order: lane-verify + lane-assets first (unblock truth + launch), lane-charts second, M-chrome last. Say the word and I fan out.
