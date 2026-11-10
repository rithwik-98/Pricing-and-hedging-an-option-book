# Pricing and Hedging a Small Option Book
## Summary

**Project focus.** This project asks how much risk an options desk can remove by moving from a simple delta hedge to hedges for gamma and vega, and what it gives up in return. I built a Black–Scholes pricing engine, checked its accuracy against established benchmarks, and applied its prices and sensitivities to an illustrative option book over 20 years of S&P 500 and VIX data (1999–2018) [9, 10].

### The approach

The pricing engine starts from the Black–Scholes PDE [1]. After transforming to log-price and time-to-expiry, I solve the resulting heat equation using Crank–Nicolson finite differences [2]. Rannacher start-up helps control oscillations near the payoff kink, particularly when computing gamma [3]. For American puts, a projected SOR solver imposes the early-exercise constraint [4]. Prices are checked against closed-form Black–Scholes values, a Cox–Ross–Rubinstein binomial tree [5] and Longstaff–Schwartz Monte Carlo [6]. On the tested grids, price, delta and gamma show roughly second-order convergence. A representative American put is priced at 6.0899, compared with 6.0905 from the binomial tree.

The historical experiment uses a short at-the-money call, a short 95%-strike American put, and a long 105%-strike call. Each book starts with about three months to expiry and is held for 21 trading days. Across 236 non-overlapping monthly windows, I compare an unhedged book, static and daily delta hedges, and gamma/vega hedges using additional calls. Hedge options are set at the beginning of a cycle, while the stock delta hedge is updated daily. The primary comparisons are before transaction costs.

### What the results show

| Hedge strategy | Monthly P&L standard deviation | Reduction in variance vs. unhedged |
|---|---:|---:|
| Unhedged | 1.85% | — |
| Static delta | 1.47% | 37% |
| Daily delta using VIX | 0.98% | 72% |
| Daily delta using realized volatility | 1.27% | 53% |
| Daily delta + static gamma hedge | 0.48% | 93% |
| Daily delta + static gamma and vega hedges | 0.28% | 98% |

**The largest improvement comes from moving beyond delta alone.** Daily delta rebalancing reduces the standard deviation of monthly P&L from 1.85% to 0.98% of starting index notional. Hedging gamma and then vega reduces it further to 0.48% and 0.28%. In this model, volatility changes account for much of the risk remaining after delta hedging: a regression on the change in VIX explains about 87% of daily-delta-hedged cycle P&L variation. This is a strong association within the experiment, not a causal estimate.

**Stress can arrive after a seemingly ordinary starting point.** The worst cycle begins on 11 September 2008, when VIX is around 24—elevated but below the study's stress threshold of 30. During the cycle, the index falls about 28% and VIX rises by roughly 46 points. The modelled loss falls from 20.7% of notional unhedged to 9.2% with daily delta, 5.0% with gamma hedging, and 2.5% with gamma and vega hedging. Classifying risk only by VIX at trade initiation would miss this episode.

**The volatility estimate used for delta matters.** Replacing VIX with trailing 63-day realized volatility worsens overall monthly P&L dispersion: 1.27% versus 0.98% with VIX-based deltas. In the calm subsample, the realized-volatility delta hedge is even noisier than leaving the book unhedged. The comparison should be understood within this particular marking model, which itself uses VIX for option valuations.

**Risk reduction is not free.** The fully hedged strategy has substantially less dispersion and smaller losses in the stress example, but it also removes much of the volatility exposure that generates premium for the original short book. Over the sample, the sum of unhedged cycle returns is about +21% of initial notional per cycle, compared with about −66% for the gamma-and-vega-hedged strategy. Those are sums of separately normalized model P&Ls, not a compounded live trading return. Rebalancing frequency matters for costs: weekly delta adjustment costs about 0.034% per cycle, compared with 0.058% for daily, with similar gross error in this setup.

**Linear VaR misses nonlinear option exposure.** At the 99% one-day level, delta-normal VaR is breached on 3.7% of days for the unhedged book; delta–gamma simulation gives 1.2%, and filtered historical simulation with full revaluation [7] gives 1.0%. For a delta-hedged book, the delta-normal estimate collapses to zero, even though observed P&L still varies because of gamma. The full-revaluation method is closest to its nominal exception rate in these tests. The exception rates are evaluated using the Kupiec coverage test [8].

### Interpretation and limitations

The main result is a risk decomposition, not a proposed trading strategy: delta hedging addresses first-order spot exposure, while substantial gamma and especially vega risk remain. Hedging those terms makes the P&L distribution much tighter, but changes the book's expected return and adds trading costs. A sensible hedge therefore depends on whether the desk values premium income or protection against large adverse repricings more highly.

Several modelling decisions limit how literally the numbers should be read. The option marks are generated from a flat volatility surface set equal to VIX/100, rather than observed option quotes. The model ignores dividends and skew, holds the risk-free rate at 2%, and includes an American put in an otherwise index-style book. The 236 monthly windows do not overlap but are not necessarily independent; only 23 begin in the high-VIX regime. The VaR back-test holds implied volatility fixed over the next-day scenarios, so it measures spot risk rather than joint spot–volatility risk.

The most useful next step would be to repeat the exercise with actual option-chain quotes and a volatility model that can represent skew and changes in volatility, then compare dynamically rebalanced option hedges after realistic trading costs.


### References and data sources

1. Black, F., and Scholes, M. (1973). “The pricing of options and corporate liabilities.” *Journal of Political Economy*, 81(3), 637–654. [10.1086/260062](https://doi.org/10.1086/260062).
2. Crank, J., and Nicolson, P. (1947). “A practical method for numerical evaluation of solutions of partial differential equations of the heat-conduction type.” *Proceedings of the Cambridge Philosophical Society*, 43(1), 50–67. [10.1017/S0305004100023197](https://doi.org/10.1017/S0305004100023197).
3. Rannacher, R. (1984). “Finite element solution of diffusion problems with irregular data.” *Numerische Mathematik*, 43, 309–327. [10.1007/BF01390130](https://doi.org/10.1007/BF01390130).
4. Wilmott, P., Howison, S., and Dewynne, J. (1995). *The Mathematics of Financial Derivatives: A Student Introduction*. Cambridge University Press. [10.1017/CBO9780511812545](https://doi.org/10.1017/CBO9780511812545).
5. Cox, J. C., Ross, S. A., and Rubinstein, M. (1979). “Option pricing: A simplified approach.” *Journal of Financial Economics*, 7(3), 229–263. [10.1016/0304-405X(79)90015-1](https://doi.org/10.1016/0304-405X(79)90015-1).
6. Longstaff, F. A., and Schwartz, E. S. (2001). “Valuing American options by simulation: A simple least-squares approach.” *The Review of Financial Studies*, 14(1), 113–147. [10.1093/rfs/14.1.113](https://doi.org/10.1093/rfs/14.1.113).
7. Hull, J., and White, A. (1998). “Incorporating volatility updating into the historical simulation method for value-at-risk.” *Journal of Risk*, 1(1), 5–19. [Publisher page](https://www.risk.net/journal-of-risk/2161156/incorporating-volatility-updating-into-the-historical-simulation-method-for-value-at-risk).
8. Kupiec, P. H. (1995). “Techniques for verifying the accuracy of risk measurement models.” *The Journal of Derivatives*, 3(2), 73–84. [10.3905/jod.1995.407942](https://doi.org/10.3905/jod.1995.407942).
9. Sheppard, K. *arch* Python package, built-in `arch.data.sp500` dataset (S&P 500 daily prices, 1999–2018). [Dataset documentation](https://bashtage.github.io/arch/devel/data.html).
10. Data Packages / Cboe. *finance-vix* dataset: daily Cboe Volatility Index closing values, accessed via the project’s GitHub repository. [Dataset repository](https://github.com/datasets/finance-vix).
