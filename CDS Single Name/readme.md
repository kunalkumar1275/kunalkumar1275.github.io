CDS Pricing & Hazard Rate Bootstrap

Bootstraps a piecewise-constant hazard rate curve from market CDS spreads (6m, 1y, 2y, 3y, 5y), then prices the CDS off that curve on a daily grid.

Why CDS trades

A CDS lets an investor buy or sell credit risk on a reference entity without owning its bonds or loans. The protection buyer pays a periodic spread and receives a payout if a credit event (default) occurs; the protection seller takes the opposite side. It's used to hedge bond/loan exposure, take a pure directional view on credit quality, or trade the basis between cash bonds and CDS-implied spreads — all without funding the underlying position.

What is a hazard rate?

The hazard rate λ is the instantaneous conditional probability of default per unit time, given survival up to that point — a forward default intensity. Survival probability follows Q(t) = exp(−∫λ ds); here λ is bootstrapped as piecewise-constant per segment (0–6m, 6m–1y, 1–2y, 2–3y, 3–5y), solved so the model-implied par spread matches the market-quoted spread at each tenor in sequence — the same sequential-bootstrap logic used for discount curves, applied to default intensity instead.

DV01 and CS01

DV01 — dollar sensitivity to a 1bp move in the interest rate curve. For a CDS this is usually a secondary risk, since rates only affect discounting, not survival probabilities.
CS01 — dollar sensitivity to a 1bp move in the credit spread curve, the dominant risk for a CDS position. Computed here as RPV01 × Notional / 10000, i.e. the risky annuity (RPV01) scaled by notional and one basis point — the standard market convention for spread risk, holding survival probabilities fixed to first order.

Limitations

Flat, deterministic risk-free rate — a real bootstrap would discount off an actual OIS curve, not a single flat r.
Piecewise-constant hazard rate is a modelling choice — it assumes flat forward default risk between tenor points and won't capture intra-period term structure.
Recovery rate assumed flat (40%, market convention) rather than uncertain or state-dependent — real recovery is uncertain and can be correlated with the default event itself.
No counterparty risk on the protection seller — the model assumes the protection seller never defaults, ignoring wrong-way risk between the reference credit and the counterparty.
Simplified conventions relative to the full ISDA Standard CDS Model (day counts, accrual, big-bang conventions) — fine for illustrating the mechanics, not a drop-in replacement for production pricing.
