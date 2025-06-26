# Technical Design

Derion comprises a set of well-isolated, immutable smart contracts, designed for deployment on multiple EVM-compatible chains.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The core protocol features a single **Pool Logic** contract that serves as the **ERC-3448** implementation for all Derion speculation market pools. Each individual pool is instantiated as a separate **MetaProxy**, defined with immutable configurations and linking to the shared, immutable Pool Logic contract as its implementation.

This architecture allows pools to be deployed with diverse **Fetcher** logic, enabling seamless adaptation to any type of oracle feed. This includes on-chain sources like **DEXs (e.g., Uniswap)** and robust off-chain oracles like **Chainlink**.
