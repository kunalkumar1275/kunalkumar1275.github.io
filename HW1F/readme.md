This folder talks about how to calibrated or solve for Hull White 1-Factor model using market data curves and caplet (vol) prices
HW1F is a commonly used models fit the today's yield curve exactly and then provides analytical formula for cap/caplets and european
swaptions.
Once solved, it can provide distribution for any rate in a future time (say distribution of 5y spot rate at future time = 1y)
HW1F is a good model to price early exercise products like Bermudan swaptions (trinomial), callable bonds, callable swaps on a 
single rate (say 5y swap rate, not (5y - 2y) swap rate), callable bond (bond - bermudann swaption), callable swaps
Path dependent options can also be priced by HW1F but we have to use a MC simlulation as storing multiple average values across 
multiple paths at a given node can be compute heavy
Limitations: One factor (vol is not stochastic), no smile, normal rates (rates can go negative) - issues across tree or MC

# Hull-White One-Factor (HW1F) — Dual-Curve Cap Calibration

This folder calibrates a **Hull-White One-Factor** short-rate model to a **dual-curve** market (separate discounting and projection curves) using **cap/caplet** prices, solved **live in Excel via Solver**

## 1. Model Intuition

HW1F models the short rate as mean-reverting and normally distributed:

```
dr(t) = a(θ(t) − r(t))dt + σ·dW(t)
```

- **θ(t)** is calibrated so the model reprices today's yield curve *exactly* — no fitting error on the initial term structure.
- **a** (mean reversion speed) and **σ** (short-rate vol) are the two free parameters — these are what get calibrated to market prices here.
- Because the model gives the full future *distribution* of rates (not just today's curve), it can answer questions like "what's the distribution of the 5Y spot rate in 1Y?" — which a curve alone cannot.
- Caps/floors and European swaptions have closed-form (analytical) prices under HW1F, which is what makes it fast enough to calibrate and use in a tree.

## 2. What HW1F Is Used For

- Early-exercise, single-rate products: Bermudan swaptions (via trinomial tree), callable bonds, callable swaps — as long as the exercise decision depends on one rate (e.g. the 5Y swap rate), not a spread between two rates (e.g. 5Y−2Y).
- Callable bond ≈ bond − Bermudan swaption, priced consistently off the same calibrated tree.
- Path-dependent payoffs are possible via Monte Carlo, but become compute-heavy since you need to store/average state across many paths at each time node — a tree is far cheaper when it's applicable.

## 3. What's In This Workbook:
There are solved sheets and **practice sheets** for the user

| Sheet | Role |
|---|---|
| `Instruments` | Raw market quotes (input only — blue cells) |
| `OIS_Curve` | Bootstrapped discounting curve from OIS money market + OIS swaps |
| `Proj_Curve` | Bootstrapped projection curve from SOFR futures (0–2Y) + IRS (2–10Y) |
| `Par_Rates` | Dual-curve ATM par swap rates — used as cap strikes K |
| `HW_Caplets` | Analytical HW1F caplet pricing, per cap maturity |
| `Cap_Prices` | Model vs. market cap prices, squared error — the Solver target |
| `Parameters` | a, σ, r₀ — a and σ are the Solver's changing cells |
| `TRY_*` | Blank practice copies of the above (yellow cells + a check column) so you can redo the bootstrap and caplet math yourself and confirm against the solved answer |

## 4. Calibration Workflow

1. Bootstraps a **dual-curve setup** (OIS discounting + SOFR projection curve)
2. Derives ATM cap strikes from the dual-curve par rates
3. Prices caplets analytically under HW1F
4. **Calibrates (a, σ) via Excel Solver** to match market cap prices (minimizes SSE)

## 5. How To Reproduce This Yourself

1. Download the `.xlsx` and open in Excel.
2. Enable Solver: `File → Options → Add-ins → Excel Add-ins → Solver Add-in`.
3. Go to `Data → Solver` and enter the setup from step 7 above (it's also written out on the `Cap_Prices` and `Parameters` sheets).
4. Click **Solve**.
5. Optional: work through the `TRY_*` sheets yourself (yellow input cells only) — the check column tells you if each step matches the solved version.

## 6. Limitations

- **One factor**: a single Brownian motion drives the whole curve, so all rates are instantaneously perfectly correlated — the model can't capture partial decorrelation across tenors that real curves show.
- **No smile**: one constant σ per calibration, so it can't fit a volatility skew/smile across strikes — only ATM-level vol.
- **Normal rates**: the short rate is Gaussian, so it can go negative with positive probability, which can cause issues in tree/MC construction if unbounded.
- **Calibrated to caps only, no swaptions**: this is the main limitation of **this specific calibration**. Caps are a strip of independent caplets, each pricing off the volatility of a single forward rate — they're informative about σ but only weakly identify a, since a's main effect is on how strongly *different* points on the curve decorrelate from each other, which caplets don't see. That weak identification shows up directly in the result above: solved a = 0.163 vs. true a = 0.15, even though the cap fit itself is excellent.
  Swaptions, by contrast, price a *single* rate (the swap rate) that is itself a blend of multiple forward rates — so swaption prices are far more sensitive to a. Calibrating jointly to caps *and* swaptions pins down both parameters more robustly: σ mainly from caps, a mainly from swaptions. This also matters practically — Bermudan swaptions and callable swaps (the products HW1F is normally used for) reference swap-rate dynamics, so a model calibrated only to caps can misprice the early-exercise value even while fitting the cap market perfectly.

## 7. Possible Extensions

- Add a swaption panel and a joint objective (cap SSE + swaption SSE) to better identify a.
- Port the calibration to Python (`scipy.optimize`) for a reproducible, testable version alongside this Excel reference.
- Move to a multi-factor Hull-White or a stochastic-vol extension to address the smile limitation.
