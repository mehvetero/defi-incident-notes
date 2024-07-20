# Euler Finance — $197M (March 2023)

## What happened

1. Attacker flash-borrowed 30M DAI from Aave
2. Deposited into Euler → received eDAI
3. Used `donateToReserves()` to push their eDAI position into bad debt — the donated amount made their collateral worthless but left the debt
4. Self-liquidated the bad-debt position at a discount
5. Because the donated reserves count as protocol-owned, the liquidation gave the attacker fresh collateral at a discount
6. Repeated across DAI, WETH, USDC, stETH

## Root cause

`donateToReserves()` was a public function with no guard. It allowed any depositor to burn their own collateral without repaying the debt. The protocol assumed nobody would intentionally create bad debt because it would be a loss for the donor — but combined with the liquidation discount, the loss was smaller than the gain.

## What the audit missed

The donate function was audited. The individual behavior was correct — it moved tokens to reserves. What was missed was the interaction between donate (creates bad debt) and liquidate (profits from bad debt). The composition of two safe operations created an unsafe outcome.

## Lesson

Always trace what happens when a user intentionally puts themselves into an unhealthy state. "Why would anyone do this?" is not a security argument. If the protocol allows it, model the worst case.
