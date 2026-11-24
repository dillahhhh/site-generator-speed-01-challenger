# site-generator-speed-01 — challenger

[![make.fun bounty](https://img.shields.io/badge/make.fun-bounty-blue)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Node](https://img.shields.io/badge/node-%3E%3D22-green)](https://nodejs.org)

A faster drop-in for [makefun-site 1.0.0](https://make.fun/bounty/site-generator-speed-01-run-20261122),
make.fun's own static site generator — built for the
[site-generator-speed-01-run-20261122](https://make.fun/bounty/site-generator-speed-01-run-20261122)
bounty ("Build a static site faster", $100, entries close 2026-11-24T20:00:00Z).

`solve.js` exports `run(input)`, which returns exactly the answer makefun-site
1.0.0 returns for the same input — the output tree as a JSON object of every
path it writes and that file's text, byte for byte. `main.ts` is the driver the
harness runs: it reads every case from `/input/cases.json`, answers each with
`run`, and writes `/output/answers.json`.

## How it is faster

The reference pipeline is kept step by step (sources → front matter →
permalinks → Markdown → collections → tags → layouts → report); only the hot
work changed:

- **Markdown is lexed and rendered in one fused pass** instead of building a
  token tree and then walking it.
- **Inline rendering escapes plain-text runs in bulk** instead of one regex
  escape call per character.
- **Slugify takes an ASCII fast path** — NFKD normalisation and combining-mark
  removal are the identity on ASCII — and repeated slugify results are
  memoised per run (tag names and titles repeat across a site).
- **Each page's card is built once** and reused by its layout instead of
  being rebuilt per render.

Nothing is cached from one case to the next: every per-run cache is created
inside `run()` and dropped with it. No dependencies, no WebAssembly, no native
add-ons, no prebuilt binaries — JavaScript source only, Node 22.

## Verify locally

```sh
# answer the practice set with this entry
MAKEFUN_INPUT_DIR=../site-generator-speed-01-kit/site-generator-speed-01-run-20261122-kit/practice/input \
MAKEFUN_OUTPUT_DIR=/tmp/sgs-out \
node main.ts

# compare against the kit's expected answers
node -e "const a=require('/tmp/sgs-out/answers.json');const e=require('../site-generator-speed-01-kit/site-generator-speed-01-run-20261122-kit/practice/reference/expected.json');console.log(JSON.stringify(a)===JSON.stringify(e)?'IDENTICAL':'DIFFER')"
```

## Layout

| File | What it is |
| --- | --- |
| `solve.js` | `run(input)` — the fast drop-in |
| `main.ts` | Harness driver: cases in, answers out |
| `package.json` | `{"type": "module"}` — ESM |
| `LICENSE` | MIT |

## License

MIT — see [LICENSE](LICENSE).
