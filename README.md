# DeFi Incident Notes

Personal notes from studying DeFi security incidents. Each file covers one incident with:

- What happened (timeline)
- Root cause (technical)
- What the auditors missed (if audited)
- What I would check for in future reviews

Not comprehensive — these are the ones I found technically interesting or where the failure mode taught me something new.

## Index

| Incident | Date | Loss | Root Cause | File |
|----------|------|------|------------|------|
| Euler Finance | Mar 2023 | $197M | Donate + self-liquidation | [euler.md](euler.md) |
| Curve Pool Reentrancy | Jul 2023 | ~$70M | Vyper compiler bug | [curve-reentrancy.md](curve-reentrancy.md) |
| Radiant Capital | Oct 2024 | $50M | Developer device compromise | [radiant.md](radiant.md) |
| Bybit | Feb 2025 | $1.5B | Safe frontend supply chain | [bybit.md](bybit.md) |

## Observations

After writing up ~20 incidents (only posting the interesting ones), a few patterns are clear:

1. **Smart contract logic bugs are declining** — auditors are getting better, tooling is improving, formal verification is becoming standard for high-TVL protocols
2. **Operational security is the new attack surface** — key management, developer devices, deployment pipelines, vendor trust chains
3. **Social engineering precedes almost every >$50M hack since 2024** — Radiant, Bybit, Drift, Step Finance all started with a human being tricked
4. **Multisig is not a solution if all signers use the same frontend** — a compromised UI makes 3-of-5 equivalent to 1-of-1
5. **Move/Sui contracts have a different threat model** — resource semantics prevent many classic EVM bugs but introduce new ones around capabilities and object ownership

I am spending more time on non-EVM chains now. Sui Move in particular has interesting security properties worth studying.
