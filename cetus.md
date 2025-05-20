# Cetus Protocol — $260M (May 2025)

## What happened

The largest Sui DeFi exploit. An attacker exploited an arithmetic overflow in the concentrated liquidity math of Cetus DEX. The overflow was in a custom `checked_shlw` (shift left with wrap) function used to calculate liquidity deltas.

## Root cause

The `checked_shlw` function was supposed to detect overflow in a 256-bit left shift operation, but the check had an off-by-one error. For specific input values near the boundary, the function returned a wrapped (incorrect) result instead of aborting.

This allowed the attacker to open a liquidity position where the computed liquidity delta was astronomically larger than the actual token deposit justified. The attacker then removed the (inflated) liquidity, receiving far more tokens than they deposited.

## Timeline

- Attack executed across multiple Cetus pools within minutes
- Sui Foundation froze the attacker's on-chain address (controversial — Sui validators can deny transactions from specific addresses)
- Most funds recovered through a combination of the freeze and negotiation
- Community debate about whether Sui's ability to freeze addresses undermines decentralization

## What I learned

1. Move type safety does not prevent math bugs — `u128` overflow in a shift operation is the same class of bug as a Solidity `uint256` overflow
2. Custom math libraries need dedicated math-focused review, not just security review
3. The "Move is safe by construction" narrative is overstated — it prevents memory safety issues and reentrancy, but logic bugs are equally likely
4. Concentrated liquidity math (Uniswap V3 style) is inherently complex regardless of the language — ticking, rounding, bit manipulation all have edge cases
5. Sui's validator-level address freeze is a double-edged sword — useful for incident response, dangerous for censorship
