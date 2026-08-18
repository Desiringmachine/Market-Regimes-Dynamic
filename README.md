![screenshot](https://github.com/Desiringmachine/Market-Regimes-Dynamic/blob/main/Screenshot%202026-08-18%20145226.png)
# Market Regimes (Dynamic) — NQStats [Desiringmachine]
Volatility Regime Detection for Any Instrument

A TradingView Pine Script v6 overlay indicator that measures and classifies
volatility regime across six timeframes simultaneously: 

**Daily, Session,Hourly, Weekly, Monthly, and Yearly**.


> 
> **Author / Code:** [@Desiringmachine](https://x.com/Desiringmachine) · GitHub: [@Desiringmachine](https://github.com/Desiringmachine)
> **Concept:** [nqstats.com](https://nqstats.com)
> **Credit:** [@probablechris](https://x.com/probablechris)

---

## What it does

Each monitor panel answers one question:

> *Is volatility right now elevated, compressed, or normal relative to history?*

It does this by computing two statistics for every timeframe and comparing them:

| Statistic | What it is |
|---|---|
| **roll(N)** | Rolling stdev of the last N **completed** period returns — the blue line |
| **base(Xy)** | Rolling stdev of every period return within the last X **years** — the yellow line |

The ratio `roll / base` determines the regime colour of the square (■):

- **Red** → `roll > base × 1.05` — volatility elevated (expansion)
- **Green** → `roll < base × 0.95` — volatility compressed
- **Blue** → within band — normal regime

Thresholds are adjustable in settings.

---

## Live (in-progress) projection

Every monitor also displays what the current still-forming period is doing
**right now**, before it closes:

### Live diamond ◆
A dotted lead-line from the last completed blue-line point extends toward
where the **next** blue-line point will land. The diamond walks horizontally
as time elapses within the period (e.g. by August in a calendar year it sits
~63% of the way across). Its vertical position is the **projected roll(N)**:

proj = stdev( N−1 most recent completed returns + live return )

The live return is scaled by `sqrt(1 / elapsedFrac)` before entering this
formula — the standard volatility annualisation rule. Without scaling, a
partial-year return of +18% (7.5 months elapsed) would be numerically smaller
than full-year returns simply because less time has passed, pulling the
diamond artificially close to the blue line regardless of regime. With
scaling: `18% × √(1/0.63) ≈ 22.7%` — a proper annualised-volatility
equivalent that sits at the right position relative to history.

A 3% minimum elapsed-fraction guard prevents blow-up on the very first bars
of a new period.

### Header text fields (all independently toggleable)
| Field | Shows |
|---|---|
| `live ##%` | Raw signed return of the current period open-to-now |
| `roll' ##%` | Projected roll(N) — the diamond's actual plotted value |
| `roll ##%` | Last completed roll(N) — the final blue-line point |
| `base ##%` | Dynamic baseline stdev over the lookback window |
| `x##` | Ratio of roll / base |
| `n=###` | Sample count feeding the baseline (optional) |

---

## Monitors

| Monitor | Updates | Default on |
|---|---|---|
| **D** — Daily | Each day close | Yes |
| **S** — Session | Each session close (Asia / London / NY AM / NY PM) | Yes |
| **H** — Hourly | Each hour close | Yes |
| **W** — Weekly | Each week close | No |
| **M** — Monthly | Each month close | No |
| **Y** — Yearly | Each calendar year close (derived from daily bars) | No |

Session and Hour monitors support **Auto** mode (follows the current live
session / hour) or a fixed selection.

---

## Settings groups

### Roll / Base — Daily / Session / Hourly
Core rolling window length and baseline lookback years, shared by D, S, H.
Defaults: **10 instances / 5 years**.

### Roll / Base — Weekly / Monthly / Yearly
Each timeframe has its own independent Roll and Base inputs so you can use
a longer historical window for slower-moving timeframes without affecting the
intraday monitors. Yearly defaults: **5 instances / 30 years**.

### Monitor 1 → 6
Toggle each monitor on/off. Session and Hour mode selectors.

### Regime Status
Live overlay controls — all independent:
- **Live Label** — show `live ##%` in header
- **Live Roll'** — show `roll' ##%` in header
- **Live Marker ◆** — draw diamond + dotted lead-line on chart
- **Live Square ■** — live-updating regime square (vs static last-completed square)
- **N Sample** — show baseline sample count in header
- **Scale ◆ Sqrt-Time** — apply sqrt-time volatility scaling to the live return
  before diamond projection (recommended ON)

### Square Ratio
Elevated / Compressed ratio thresholds (default 1.05 / 0.95).

### SE(SD) Confidence Band
Shaded band around the base line showing `±z × SD/√(2n)` uncertainty.
Default z = 1.96 (~95% CI). Useful for gauging baseline stability.

### Monitor Colors / Layout
Panel height, gap, horizontal offset, bars shown, colour overrides for
every element.

---

## Notes on accuracy

- **Yearly / Monthly** baselines accumulate slowly (1 instance per year/month).
  The SE(SD) band is intentionally wide on those panels — that is statistically
  honest, not a bug.
- **Hourly / Session sample count** depends on chart resolution. For the
  highest `n`, view the chart at 1h or coarser while reading those monitors —
  at finer resolutions TradingView reconstructs 60-minute bars from intraday
  data, which covers less calendar history for the same bar budget.
- The diamond does **not** predict where price will finish. It answers:
  *"if this period closed at the current price, what would roll(N) print?"*

---

## Requirements
- TradingView account (any plan)
- Pine Script v6
- Sufficient chart history for the baseline lookback you configure
  (5Y default for D/S/H; 30Y default for Yearly)

- > **Tradingview:** (https://www.tradingview.com/script/oFrjS4G4-Market-Regimes-Dynamic-NQStats-Desiringmachine/)
