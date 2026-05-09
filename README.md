# Bitcoin epoch new-buyer cohort survival — Stratified Kaplan-Meier

Single-file, dependency-light interactive analysis page that compares the
"new buyer" cohort survival of every Bitcoin halving epoch using a
stratified Kaplan-Meier survival function. The chart auto-extends by one
day each calendar day so Epoch 5's curve is always plotted up to today,
with prior epochs truncated to the same days-since-halving point for fair
comparison.

**Live demo:** [your-username.github.io/your-repo/](https://your-username.github.io/your-repo/)

## What it shows

For each halving epoch:

- **Cohort** = every UTXO created during the first *W* days post-halving
  (the "onboarding window") AND still un-spent at the close of that
  window.
- **Survival curve S(t)** = fraction of that cohort still un-spent at
  follow-up day *t* post-onboarding.
- **Curves are read at the right edge**: the higher the curve at the
  current days-since-halving point, the stickier that epoch's new
  buyers.

The onboarding window *W* is configurable via the dropdown (30, 60, 90,
or 180 days). All four options correspond exactly to BRK age-band edges
so the cohort denominator at *t*=0 has zero interpolation error.

## Method

For each epoch *N* (halving date *H<sub>N</sub>*) and onboarding window
*W*, evaluated through follow-up day *t* in [0, *T* − *W*] where *T* =
days(today − Halving 4):

```
n(0) = count of UTXOs aged 0..W at observation H_N+W       (cohort)
n(t) = count of UTXOs aged t..t+W at observation H_N+W+t   (survivors)
d(t) = n(t-1) − n(t)                                       (events)
S(t) = ∏_{τ ≤ t} (1 − d(τ)/n(τ-1))                          (K-M)
```

Algebraically *S(t) = n(t) / n(0)* given no within-window censoring
(which holds: all cohort UTXOs are observed throughout the horizon).

The cohort identification step uses the fact that at observation
*d = H<sub>N</sub> + W + t*, original-cohort UTXOs have age in
[*t*, *t*+*W*] days. Both endpoints algebraically resolve to the same
birth window [*H<sub>N</sub>*, *H<sub>N</sub>+W*], so the slice tracks
exactly the original cohort.

## Data source

All inputs come from a Bitcoin Research Kit (BRK) deployment at
[bitview.space](https://bitview.space/api), specifically from the
`utxos_under_<BAND>_old_utxo_count` endpoint family at daily resolution
(`/api/series/<name>/day1`).

Bands fetched:

```
1h  1w  1m  2m  3m  4m  5m  6m  1y  2y  3y  4y  5y  6y  7y  8y  10y  12y  15y
```

For follow-up days falling between band edges, supply is linearly
interpolated. Reliable below 6 months (1-month band spacing); coarser
beyond, where bands widen to 6 and 12 months.

## Caveats

1. **"New buyer" is a UTXO proxy, not a wallet/person proxy.** Includes
   coinbase, change outputs, and exchange-internal moves. Cross-epoch
   comparison stays valid because the same noise distribution applies to
   each epoch; the absolute survival rates are not literally "fraction
   of new buyers who held."
2. **Linear interpolation between band edges** is reliable below 6
   months; past 6 months the gaps widen to 6 and 12 months and
   interpolation noise increases.
3. **Independence assumption violated.** Large entities (exchanges,
   ETFs, treasury holders) spend many UTXOs simultaneously. This file
   omits confidence bands and significance tests deliberately for
   readability; if you need them, see [Greenwood variance and the
   log-rank test](https://en.wikipedia.org/wiki/Logrank_test).
4. **Epoch 1 excluded from chart.** 2009-2010 supply is so small that
   the cohort denominator is unstable. Math runs but the curve is not
   plotted.

## Stack

- Pure HTML / CSS / vanilla JS — no build step
- One CDN dependency: [TradingView Lightweight Charts v5](https://github.com/tradingview/lightweight-charts)
  (Apache 2.0 license)
- Live data from BRK at `bitview.space`

## Local development

No build needed. Open `index.html` in a browser.

For local file fetches against the BRK API:
- The page makes cross-origin fetches to `https://bitview.space/api/...`
- Modern browsers may block file:// → https:// XHR; serve over HTTP if
  needed:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/
```

## License

CC0 1.0 Universal — public domain. Free to use, modify, redistribute,
attribution appreciated but not required. See [LICENSE](LICENSE).

## Credits

- **Bitcoin Research Kit (BRK)**: open-source Bitcoin indexer by Antoine
  Le Calvez et al. — [github.com/bitcoinresearchkit/brk](https://github.com/bitcoinresearchkit/brk)
- **TradingView Lightweight Charts**: open-source charting library
- **Kaplan-Meier**: Edward L. Kaplan & Paul Meier (1958), *Nonparametric
  estimation from incomplete observations*, JASA 53(282).

---

Built as part of [bitcointerminal.net](https://bitcointerminal.net).
