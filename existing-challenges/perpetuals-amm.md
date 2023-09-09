---
description: Automated Market Maker
---

# Perpetuals AMM

AMM is a set of contracts protocol that allows anyone to create a market of their choice. Uniswap is the first spot AMM by solving the bottleneck problems of the legacy order-book exchange that plague the spot DEXes at that time.

To create an AMM for Perpetuals, Derivable has to solve the following problems:

* Elimination of unique positions.
* Smooth derivative pricing curves with an Auto-Deleveraging mechanism.
* Infinite liquidity with a limited pool reserve.
* Deterministic fee and rates: interest, funding rate, premium, and protocol fee.
* Price oracle manipulation resistant.
* MEV front-running resistant.
