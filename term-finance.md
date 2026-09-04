# Term Finance — Governance Takeover ($8.5M, Aug 2026)

## What Happened

On August 23, 2026 at 06:25 UTC, an attacker drained approximately $8.5 million from Term Finance's Meta Vaults on Ethereum. No smart contract bug was exploited. The attacker bought a governance majority and voted the funds out.

Term Finance is a fixed-rate lending protocol. Its yield vaults sit on Yearn V3 infrastructure with a custom governance layer on top. Depositors receive a governance token (tmvETH) proportional to their stake.

## The Attack

The total staked supply of gtmvETH was 0.5352 at the time. The attacker purchased 0.4852 tmvETH for roughly $951, giving them 90.66% of all voting power.

With that majority, the attacker submitted governance proposals containing two actions:

1. Set the timelock cooldown to zero — removing the delay on all future proposals
2. Add a new vault strategy pointing to the attacker's own address

Both actions passed. No one vetoed during the 6-day window. On execution, the malicious strategy drained 2,841.74 WETH (~$6.9M) through a multi-vault redemption chain (Aave, Morpho Blue, Yearn V3 wrappers). A separate execution drained ~1.68M USDC, later swapped to DAI.

## Key Transactions

- **Drain TX:** `0xd354a15b15cb73d30908f411aee3f795ec86737a4d080e9a818ac4d6d3014129` (block 25,816,049)
- **Attacker:** `0xa908b3472d76e7744bab0a5911768a4a6300612b`
- **Governance executor:** `0x64e477800051efb06ae4086f4b258b270668b4df`
- **Method:** `0x373058b8` (parameterless execute)
- **Consolidation wallet:** `0xD5183d8BfC65a50863C62aF2538198A8288FFc13`

78 log events in the drain TX — each vault redeemed from its underlying lending position before routing to the attacker's strategy.

## Timeline

| Date | Event |
|------|-------|
| Aug 17, 05:01 UTC | Attacker funded via Tornado Cash (2 ETH) |
| Aug 17, 05:21 UTC | 0.4852 tmvETH purchased (~$951) |
| Aug 17 (est.) | Governance proposals submitted |
| Aug 17–22 | Timelock window — no vetoes |
| Aug 23, 06:25 UTC | Proposals executed, $8.5M drained |
| Aug 23 | Term Labs shuts down Meta Vaults, revokes DAO roles |

## Root Cause

Not a code bug. The governance design assumed the token holder base would be large enough to prevent a hostile majority. The total staked supply was 0.5352 — a $951 purchase gave the attacker supermajority control.

The 7-day timelock was a safeguard, but the attacker's first action on execution was to set the cooldown to zero, disabling it for all subsequent proposals. The LP veto mechanism existed but required active monitoring — nobody was watching.

## What I Learned

This is a different class from every other incident in this repository. The code was audited and correct. The governance mechanism executed as designed. Every `require` passed.

The vulnerability was economic: governance security is a function of participation depth, not mechanism design. A perfectly designed voting system with half a participant is less secure than a flawed one with ten thousand.

The timelock paradox stands out: if the governance vote can modify the timelock parameters, the timelock provides exactly one window of protection. After that first execution, the attacker owns the clock.

Worth noting: $951 is cheaper than an audit. The attacker spent less on the entire operation than most protocols spend on a single review. The audit would have found nothing — the code was clean.

## Aftermath

Term Labs permanently shut down all Meta Vaults, revoked DAO governance roles, and left withdrawals open. No postmortem published as of this writing. Consolidation wallet still held ~$7.8M at time of analysis.

Yearn clarified the issue was in Term's governance wrapper, not standard Yearn V3 vault architecture.
