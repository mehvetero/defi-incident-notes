# Scallop sSUI Spool — 150K SUI / $142K (April 2026)

## What happened

An attacker drained 150,000 SUI (~$142K) from Scallop's sSUI spool reward pool on April 26, 2026. The attack targeted a deprecated V2 spool package deployed in November 2023 — 17 months earlier — that remained callable on Sui because deployed packages are immutable and old versions stay live unless explicitly version-gated.

The attacker staked 136,000 sSUI through the deprecated package, harvested approximately 162 trillion reward points, and redeemed them one-to-one for 150,000 SUI. The core lending protocol was unaffected. Scallop covered the full loss.

The exploit transaction: `6WNDjCX3W852hipq6yrHhpUaSFHSPWfTxuLKaQkgNfVL`.

## Root cause

Uninitialized `last_index` in the deprecated V2 spool package.

The spool reward system works like a Synthetix-style reward accumulator: a global `index` increases over time as rewards accrue. Every user account stores a `last_index` — the value of the global index at the time they staked. Rewards are calculated as:

```
reward = staked_amount × (current_index − last_index)
```

In the current package, `last_index` is set to the current global index when a new spool account is created. This ensures a new staker earns nothing retroactively.

In the deprecated V2 package, `last_index` was never initialized. New spool accounts got `last_index = 0` by default. The reward formula then became:

```
reward = staked_amount × (current_index − 0)
       = staked_amount × current_index
```

After 20 months of operation, the global index had grown to approximately 1.19 billion. The attacker created a fresh account through the V2 package, staked 136,000 sSUI, and claimed the entire difference as earned rewards — as if they had been staking since the spool's creation in August 2023.

The current package had fixed the initialization. But the deprecated V2 package, still callable on Sui, had not.

## The deprecated-package pattern

This is the same pattern as BlueMove (July 2026): an old version of a package remains callable on-chain and interacts with the same shared objects as the current version. The vulnerability was in the old code, not the new code.

On Sui, upgrading a package publishes a new version but does not disable the old one. Both versions can read and modify the same shared objects. If the old version has a bug that the new version fixed, the bug is still exploitable through the old entry point.

Scallop's V2 package was deprecated in the sense that the frontend no longer used it. But nothing on-chain prevented a direct call. The attacker bypassed the frontend entirely and called the V2 package's staking function directly.

## The number

The attacker's reward claim:
```
136,000 sSUI staked × 1,190,000,000 index × 1e-12 scaling ≈ 162,000 SUI
```

The reward pool held approximately 150,000 SUI. The attacker drained it.

## Aftermath

- Scallop froze the affected contract immediately
- Core contracts restored at 14:42 UTC (under 2 hours)
- Scallop covered 100% of the loss
- The attacker offered to return 80% in exchange for a white-hat bounty
- The V2 package was not patched (immutable on Sui) — instead, version gating was added to prevent future calls

## What I learned

1. A deprecated package on Sui is not a dead package. If it compiles and it shares objects with the current version, it is a live attack surface.
2. The reward accumulator pattern (Synthetix-style `index − last_index`) is correct exactly when `last_index` is initialized at stake time. If any entry point skips that initialization, the entire reward pool is claimable in one transaction.
3. This is the second Sui incident caused by old package versions remaining callable (after BlueMove). The pattern is becoming a Sui-specific vulnerability class: **deprecated-but-callable code sharing mutable state with the current version**.
4. The 17-month gap between deployment and exploitation shows that these deprecated packages sit as dormant attack surfaces. Nobody audits code they think is retired.
5. Frontend deprecation is not on-chain deprecation. A frontend can stop linking to a package, but any user can call it directly. Version gating inside the shared objects is the only real mitigation on Sui.
