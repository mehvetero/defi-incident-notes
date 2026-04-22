# Drift Protocol — $285M (April 2026)

## What happened

North Korean UNC4736 spent SIX MONTHS building a fake quantitative trading company, established real business relationships with the Drift team (including in-person meetings at conferences), deposited $1M+ in real capital to build trust, and then exploited Solana's "durable nonces" feature to trick a Security Council member into signing a malicious transaction.

## Attack chain

1. Months 1-3: Fake company established with professional website, LinkedIn profiles, and real crypto trading activity
2. Months 3-5: Approached Drift team as a potential integration partner — legitimate business meetings, conference interactions, regular Telegram communication
3. Month 5: Established as a trusted counterparty with over $1M deposited in the protocol
4. Month 6: Created a fake collateral token (CVT), manipulated its on-chain price, used it to borrow $285M worth of real assets, then drained

The key technical vector was Solana's durable nonces — the attacker got a Security Council member to sign what appeared to be a routine governance transaction, but the durable nonce allowed the attacker to submit it at a later time in a different context.

## What I learned

1. Six months of patience — this is not a smash-and-grab, it is espionage-grade operational planning
2. Real money ($1M+) invested to build trust — the ROI was 285x
3. In-person meetings create trust that is almost impossible to overcome with technical controls
4. Durable nonces on Solana (and similar deferred-execution features on other chains) are a signing hazard — the signer does not control WHEN the signed transaction executes
5. "We know this person" is not a security control — nation-state actors can maintain a convincing persona for months
6. The entire DeFi security model assumes rational adversaries — nation-state actors have different incentive structures
