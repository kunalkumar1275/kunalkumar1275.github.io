1. SABR is stochastic volatility model, widely used to construct volatility curve for swaptions
2. SABR has 4 parameters: alpha, beta, rho (skew, correlation), vol of vol (convexity)
3. This example uses market data of USD swaptions for given expiry/maturity for a range of strikes.
4. Market prices are present for the same instruments (exp, mat, strike) for 5 days
5. Inputs to the model for each instrument (exp/mat) across strikes per day: ATM forward rate, Prices
6. Beta is assumed to be fixed for a given ccy. For USD, beta = 0
7. Outputs of the model: alpha (sigma), pho (skew/correlation), vol of vol (convexity)
8. Once the paramters are solved, we can find out the price of a given swaption across any strike as the 4 parameters gives a continous
   curve across strikes

Assumptions:
1. Prices are given in terms of notional which is market standard for each swaption (say 1y1y, 1y5y)
2. Prices are for ATM straddle, receivers on low strikes, payers on high strikes
3. Annuity is assumed to be 1 for each swaption. Annuity assumption is an extra (non-market) assumption, this is done to focus
   only on estimating the volatility curve. This means the discounting curve is known or already calibrated.

Model Setup:
1. Beta = 0 means Bachelier model, which is followed for any ccy (USD, EUR, JPY) which has seen negative or very low rates
2. We use Hagan's approximation to back-out vol at a given strike, with alpha/sigma, rho, vol of vol, beta known
3. Optimize for alpha, rho, vol of vol to minimize cost function for each exp/mat pair. Every day new calibration for a swaption
4. We first solve for alpha by setting for ATM, model vol = market vol. This is important as ATM strikes are most liquid and hence
   exact calibration in this region is relatively more important than other region/strikes
5. From market prices for non-ATM strikes, using brent inversion, we get the implied volatility for a given strike. Hagan gives model IV
   for a given strike with the parameters of SABR
6. Cost function for a given Swaption = Sum across strikes [ Vega * (IV_market - IV_sabr) ^ 2]
7. We want to minimize this cost using Levenverg Marquardt optimizer with constaints: - 1 < rho < 1, vol of vol > 0.0001
8. Vega is taken as market vega ie vega wrt Market IV (to save compute :) ). Model IV vega is better
9. It helps to have a sound starting value of pho, vol of vol as the cost function may have a flat valley.
10. Hence previous day value would be a better start for next day calibration
