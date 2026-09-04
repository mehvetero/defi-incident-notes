# DeFi Incident Notes

Personal notes from studying DeFi security incidents. Each file covers one incident with what happened, root cause, and what I learned.

## Index

| Incident | Date | Loss | Root Cause | File |
|----------|------|------|------------|------|
| Euler Finance | Mar 2023 | $197M | Donate + self-liquidation | [euler.md](euler.md) |
| Curve Pool Reentrancy | Jul 2023 | ~$70M | Vyper compiler bug | [curve-reentrancy.md](curve-reentrancy.md) |
| Radiant Capital | Oct 2024 | $50M | Developer device compromise | [radiant.md](radiant.md) |
| Bybit | Feb 2025 | $1.5B | Safe frontend supply chain | [bybit.md](bybit.md) |
| Cetus Protocol | May 2025 | $260M | Math overflow in concentrated liquidity | [cetus.md](cetus.md) |
| Typus Finance | Oct 2025 | $3.44M | Oracle `update_v2` missing assert on auth check | [typus.md](typus.md) |
| Aftermath Finance | Apr 2026 | $1.14M | Negative integrator fee — signed integer boundary | [aftermath.md](aftermath.md) |
| Drift Protocol | Apr 2026 | $285M | 6-month social engineering + durable nonces | [drift.md](drift.md) |
| Scallop sSUI Spool | Apr 2026 | $142K | Uninitialized `last_index` in deprecated V2 spool — 17-month-old callable package | [scallop.md](scallop.md) |
| BlueMove DEX | Jul 2026 | ~$528K | Cross-version reserve desync — V1/V-latest write different values to shared `reserve_x` | [bluemove.md](bluemove.md) |
| Term Finance | Aug 2026 | $8.5M | Governance takeover — $951 bought 90.66% voting power, drained vaults through legitimate proposals | [term-finance.md](term-finance.md) |

## Observations

After covering incidents from 2023 to 2026, the pattern is clear:

**Smart contract logic bugs are declining. Operational security failures are the primary attack surface for >$50M exploits.**

Every single >$100M exploit since mid-2024 involved either:
1. Key compromise (social engineering → developer device → signing key)
2. Vendor supply chain (compromised frontend, compromised dependency)
3. Operational trust (fake business relationships, insider threats)

The smart contract itself was fine in Radiant, Bybit, and Drift. The attack bypassed the contract entirely.

For Cetus ($260M), it was a genuine math bug — but notably the first major one on Sui, a chain whose marketing emphasized "safety by construction." Typus ($3.44M) and Aftermath ($1.14M) followed on Sui — both smart contract bugs, both in modules excluded from or added after the audit.

Scallop ($142K) and BlueMove ($528K) form a pair: both exploited deprecated package versions that remained callable on Sui. Scallop's was an uninitialized variable in a 17-month-old spool contract. BlueMove's was a storage relocation that left two versions writing different values to the same accounting field. Different bugs, same root cause: old code sharing mutable state with new code.

The pattern within Sui specifically: the audited core is solid; the unaudited periphery (oracles, integrator configs, late-added modules, **deprecated packages**) is where the money leaves. Five Sui incidents, and the last two share a vulnerability class that is unique to Sui's upgrade model — deprecated-but-callable code.

I am spending more time on upgrade-safety patterns now. The deprecated-package class is becoming the Sui-specific analog of the proxy-storage-collision class on EVM.

Term Finance ($8.5M, Aug 2026) is the first non-Sui, non-smart-contract-bug entry here. It earns its place because the attack class — governance takeover via thin token markets — is not chain-specific. Any protocol that gates vault control behind a governance vote with low participation is carrying the same risk. The code was clean; the electorate was empty. $951 bought the country.











