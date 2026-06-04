This folder talks about how to calibrated or solve for Hull White 1-Factor model using market data curves and caplet (vol) prices
HW1F is a commonly used models fit the today's yield curve exactly and then provides analytical formula for cap/caplets and european
swaptions.
Once solved, it can provide distribution for any rate in a future time (say distribution of 5y spot rate at future time = 1y)
HW1F is a good model to price early exercise products like Bermudan swaptions (trinomial), callable bonds, callable swaps on a 
single rate (say 5y swap rate, not (5y - 2y) swap rate), callable bond (bond - bermudann swaption), callable swaps
Path dependent options can also be priced by HW1F but we have to use a MC simlulation as storing multiple average values across 
multiple paths at a given node can be compute heavy
Limitations: One factor (vol is not stochastic), no smile, normal rates (rates can go negative) - issues across tree or MC
