# Bybit — $1.5B (February 2025)

## What happened

Largest cryptocurrency theft in history. North Korean Lazarus Group compromised a Safe{Wallet} developer's machine, injected malicious JavaScript into the Safe web application, and modified the multisig transaction that Bybit's CEO was about to sign.

## Attack chain

1. Safe{Wallet} developer's personal device compromised (exact vector undisclosed, likely similar to Radiant — social engineering + malware)
2. Attacker gained access to Safe's deployment infrastructure — AWS/Vercel (where app.safe.global is hosted)
3. Malicious JS injected into the production Safe frontend — only activated for Bybit's specific Safe address
4. When Bybit CEO Ben Zhou initiated a routine ETH cold wallet → warm wallet transfer, the Safe UI showed the expected transaction
5. The actual transaction submitted was a delegatecall to an attacker-controlled implementation contract that changed the wallet's logic
6. Once the logic was changed, the attacker drained 401,347 ETH (~$1.5B) in a series of transactions
7. $160M laundered within 48 hours through mixers and cross-chain bridges

## Root cause

Same pattern as Radiant but at infrastructure level:
- The trust chain: Bybit CEO trusts Safe UI → Safe UI trusts Safe's deployment infra → Safe's deployment infra trusts a developer's machine → developer's machine was compromised
- At no point was there a smart contract vulnerability
- Bybit's own security was irrelevant — the attack went through a vendor's infrastructure

## Lesson

1. Your security is only as strong as your weakest vendor
2. Hardware wallets displaying only a hash (blind signing) provide zero protection when the frontend is compromised
3. Infrastructure-level attacks (code injection via CI/CD) are the new meta — not smart contract exploits
4. The $1.5B was held in a 3-of-? multisig — even one compromised frontend was enough because all signers used the same frontend
