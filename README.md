# Multi-Method Options Pricing Engine with Greeks Analysis

A comparative implementation of three option pricing methods: closed-form Black-Scholes, Monte Carlo simulation with variance reduction, and a binomial tree with Richardson extrapolation. The project benchmarks convergence and computational cost across methods, calibrates against real SPY options market data, and includes a working test suite and a small API layer.

## What this project does

Three pricing methods are implemented behind a shared interface (`PricingMethod`), so they can be tested and compared on equal footing:

- **Black-Scholes**: closed-form price and Greeks, validated against known reference values and put-call parity, including a property-based test across randomized inputs.
- **Monte Carlo**: naive simulation, antithetic variates, and control variates, with a measured comparison of standard error reduction across methods.
- **Binomial tree**: CRR parameterization with early exercise for American options and Richardson extrapolation to accelerate convergence.

These are compared directly on an accuracy versus runtime basis, not just accuracy alone, since a method's practical value depends on both.

The engine is then calibrated against a real SPY options chain to extract implied volatility by strike and expiry, producing a volatility smile that shows where the Black-Scholes constant-volatility assumption breaks down against actual market prices. Solver failures during calibration are logged and explained rather than filtered out silently.

## Key findings

- The binomial tree outperforms both Monte Carlo variants on accuracy per unit of runtime for this single-underlying case, which matches the theoretical convergence rates of each method (O(1/n) for the tree versus O(1/sqrt(N)) for Monte Carlo).
- Control variate Monte Carlo outperforms antithetic variates once implemented correctly, consistent with theory.
- American puts show a real early exercise premium that grows with moneyness and time to expiry, while American calls without dividends match their European counterparts almost exactly, as expected.
- Real SPY implied volatilities show clear skew rather than a flat curve across strikes, directly contradicting the Black-Scholes constant-volatility assumption.
- About 19 percent of calibration attempts failed to converge, concentrated in two regimes: deep in-the-money contracts and near-the-money contracts on the shortest-dated expiry. Both cases share a common cause: vega near zero, which makes implied volatility unidentifiable from price alone.

## Repository contents

- `Multi_Method_Options_Pricing_Engine_with_Greeks_Analysis.ipynb`: the full project, including implementation, tests, plots, and written analysis.
- `spy_options_snapshot.csv`: the SPY options chain snapshot used for calibration, saved so results are reproducible without a live data pull.
- `snapshot_metadata.txt`: the pull timestamp, spot price, and risk-free rate at the time the snapshot was taken.

## Running this notebook

Open it in Google Colab using the link above, or upload it to Colab directly. All dependencies are installed in the first cell. The calibration section loads from the saved CSV snapshot rather than pulling live data, so results will match what is shown in the notebook regardless of when you run it.

An optional API demo near the end of the notebook spins up a FastAPI service and tunnels it with ngrok for a live request and response. This requires a free ngrok account and auth token, and the public URL will not remain active after the Colab session ends. The API code itself is fully functional and can be run in any standard Python environment.

## Background and prerequisites

This project assumes familiarity with stochastic calculus (Brownian motion, Ito's lemma), risk-neutral pricing, and basic numerical methods. The derivations and reasoning behind each method are discussed in the notebook itself rather than assumed as background knowledge.

