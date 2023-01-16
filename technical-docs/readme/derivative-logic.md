# Derivative Logic

Derivative Logic is the user-implemented contract that specifies the token conversion ratio and how each market event and scenario is handled. Logic contracts do not store or mint their Collateral token (C) and Derivatives (D). Pool contract invokes Logic contract functionalities in a caller-agnostic manner, enforcing Logic contracts to be as decentralized as possible.
