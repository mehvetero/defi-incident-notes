# Radiant Capital — $50M (October 2024)

## What happened

North Korean threat actor (UNC4736) compromised three developer machines through social engineering. The attacker impersonated a former contractor via Telegram and sent a ZIP file containing a fake audit report PDF bundled with macOS malware ("InletDrift").

Once the developer machines were infected, the malware intercepted hardware wallet signing requests. The Safe{Wallet} frontend showed legitimate transaction details, but the actual transaction submitted to the blockchain was different.

Three of eleven multisig signers approved the malicious transactions, meeting the 3-of-11 threshold. The attacker transferred ownership of lending pool contracts and drained ~$50M across Arbitrum and BSC.

## Root cause

Not a smart contract vulnerability. The attack surface was:
1. Social engineering (Telegram message from a "known" contact)
2. Device compromise (macOS malware via trojanized PDF)
3. UI spoofing (Safe frontend showed one transaction, blockchain received another)
4. Multisig threshold too low (3 of 11)

## Why the defenses failed

- Hardware wallet: useless when the host machine controls what is displayed — the wallet signed what it was told to sign
- Multisig: 3-of-11 is too low; attackers only needed to compromise 3 people
- Tenderly simulation: the malware intercepted the simulation response too — developers saw a clean simulation
- "Known contact" trust: the Telegram account appeared to be someone the team had worked with before

## What I learned

1. Device security is upstream of all other security — if the laptop is compromised, nothing else matters
2. Supply chain of trust: hardware wallet trusts the host, host trusts the user, user trusts the Telegram contact
3. Multisig threshold should be >50% of signers for any operation that can drain funds
4. Blind signing is still the norm — most multisig transactions are hashes, not human-readable intent
5. The attacker was patient — multiple Telegram conversations over days before sending the payload
