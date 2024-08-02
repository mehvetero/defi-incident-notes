# Curve Pool Reentrancy — ~$70M (July 2023)

## What happened

Several Curve pools using Vyper 0.2.15–0.3.0 were drained through a reentrancy attack. The vulnerability was in the Vyper compiler, not in the Curve contract logic.

## Root cause

Vyper's `@nonreentrant` decorator had a bug. The lock was supposed to use a dedicated storage slot, but due to a compiler regression, some pool contracts compiled with affected Vyper versions had the reentrancy guard placed at a storage slot that could be overwritten by the pool's own state variables.

This meant `@nonreentrant` silently did nothing. The pools were written correctly — they applied the guard to all state-changing functions. But the compiled bytecode did not enforce it.

## Affected pools

- alETH/ETH (Alchemix) — $22M drained
- msETH/ETH (Metronome) — $6M drained  
- pETH/ETH (JPEG'd) — $11M drained
- CRV/ETH — attacked but most MEV-recovered by c0ffeebabe.eth

## What I learned

1. Compiler bugs are real attack surface — you can audit the source perfectly and still be vulnerable if the compiler miscompiles it
2. Reentrancy is not "solved" — the concept is simple but the enforcement is a stack of trust (source → compiler → EVM)
3. Read-only reentrancy is a separate class — even if the state-changing lock works, a view function called mid-execution returns stale data
4. MEV bots as accidental white-hats — c0ffeebabe.eth front-ran the CRV/ETH attacker using the same exploit, then returned the funds to Curve
