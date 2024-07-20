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
| Euler Finance | Mar 2023 | $197M | Donate + liquidation | [euler.md](euler.md) |
| Curve Pool Reentrancy | Jul 2023 | ~$70M | Vyper compiler bug | [curve-reentrancy.md](curve-reentrancy.md) |
| Radiant Capital | Oct 2024 | $50M | Developer device compromise | [radiant.md](radiant.md) |
