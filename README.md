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
| Drift Protocol | Apr 2026 | $285M | 6-month social engineering + durable nonces | [drift.md](drift.md) |

## Observations

After covering incidents from 2023 to 2026, the pattern is clear:

**Smart contract logic bugs are declining. Operational security failures are the primary attack surface for >$50M exploits.**

Every single >$100M exploit since mid-2024 involved either:
1. Key compromise (social engineering → developer device → signing key)
2. Vendor supply chain (compromised frontend, compromised dependency)
3. Operational trust (fake business relationships, insider threats)

The smart contract itself was fine in Radiant, Bybit, and Drift. The attack bypassed the contract entirely.

For Cetus ($260M), it was a genuine math bug — but notably the first major one on Sui, a chain whose marketing emphasized "safety by construction."

I am spending more time on operational security and key management patterns now. The code audit alone misses the most expensive failures.














