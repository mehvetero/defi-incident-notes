# Aftermath Finance Perps — $1.14M (April 2026)

## What happened

An attacker drained $1.14M USDC from Aftermath Finance's perpetual futures exchange on Sui in 36 minutes across 11 transactions. The exploit required no privileged access — just a fresh account and 100 USDC.

The attacker registered as their own integrator, set a negative taker fee, and opened a market order against themselves. The negative fee subtracted from the collateral calculation turned subtraction into addition — synthetic collateral appeared in the taker account, which the attacker then withdrew as real USDC.

## Root cause

The clearing house fee path validates integrator fees with an upper-bound check only:

```move
// clearing_house (reconstructed from on-chain disassembly)
assert!(integrator_taker_fee <= max_taker_fee, invalid_integrator_taker_fee);
```

The attacker set `max_taker_fee = 0` on the taker account, then passed a negative signed fixed-point value as the fee. The protocol uses a signed fixed-point type internally — `less_than_eq` compares with `(a ^ SIGN_BIT) <= (b ^ SIGN_BIT)`, so any negative value passes the `<= 0` check.

The downstream accounting does:

```move
collateral_delta = filled_value - (taker_fee_total + integrator_fee_total);
```

When `integrator_fee_total` is negative, subtracting it *increases* the collateral delta. The attacker then deallocates this phantom collateral and withdraws it as USDC.

The missing line:

```move
assert!(integrator_taker_fee >= 0, invalid_integrator_taker_fee);
```

One comparison. That is the entire fix.

## The signed integer trap

This is not a Move-specific bug. It is a signed-integer-in-financial-code bug. The pattern:

1. A protocol uses signed types for values that should never be negative (fees, amounts, rates)
2. Validation checks the upper bound (`fee <= max`) but not the lower bound (`fee >= 0`)
3. A negative value passes validation and inverts arithmetic downstream

The `to_balance` function in the fixed-point library actually aborts on negative values (`if (value >= SIGN_BIT) abort 0`), which means the developers *knew* negative values were dangerous. But the check was at the conversion boundary, not at the entry point where the fee is first accepted.

Defense-in-depth failure: the inner function rejects negative, but the outer function lets it in. The attacker never hits the inner function because the collateral path doesn't convert to balance until after the phantom credit is created.

## OtterSec missed it

OtterSec audited Aftermath Perps in November 2025. The vulnerability was introduced in August 2025. The audit did not flag the missing lower-bound check.

This is not a criticism of OtterSec — they are a strong firm. It demonstrates that manual review has systematic blind spots for "obvious in hindsight" bugs. A signed integer flowing through a fee path without a non-negativity check is exactly the kind of thing a linter catches and a reviewer's eye skips because the code *reads* correct.

## What I learned

1. Signed types in financial code need non-negativity asserts at every entry point, not just at conversion boundaries. The fee should have been validated where it enters `create_integrator_info`, not where it exits through `to_balance`.

2. Upper-bound-only validation is a pattern worth linting for. If a function checks `x <= max` on a signed value but never checks `x >= 0`, that is a finding. This is automatable.

3. OtterSec auditing a codebase does not mean all input validation paths were exercised. Audits check critical paths under time pressure — a "set your own fee rate" path in an integrator config may not have been prioritized.

4. The attack was a single PTB (Programmable Transaction Block) — open two accounts, deposit, set negative fee, trade against yourself, withdraw. No multi-transaction setup, no flash loans, no oracle manipulation. Clean.

5. Source was not public — the DarkNavy analysis was done from on-chain bytecode disassembly. The fact that a post-mortem this detailed can be reconstructed from disassembly alone is a testament to Move's bytecode readability.

## References

- [DarkNavy analysis](https://www.darknavy.org/web3/exploits/aftermathfi-perpetuals-negative-integrator-fee-collateral-inflation/) — the only detailed technical breakdown at time of writing
- Attacker address: `0x1a65086c85114c1a3f8dc74140115c6e18438d48d33a21fd112311561112d41e`
- Clearing house package: `0x21d001e8b07da2e3facb3e2d636bbaef43ba3c978bd84810368840b7d57c5068`
