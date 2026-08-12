# BlueMove DEX — ~714K SUI / $528K (July 2026)

## What happened

An attacker drained approximately 714,000 SUI ($528,000) from 363 liquidity pools on BlueMove DEX in 23 minutes on July 11, 2026. The pools were MovePump meme-token pairs. The funds were swapped to USDC across 11 DEXes and bridged off Sui through Wormhole `deposit_for_burn`.

BlueMove attributed it to "a long-standing arithmetic overflow bug in the legacy AMM contract." Community researcher Tyler Simpson (Quantum Void Labs) called it a backdoor shipped in a May 31 package upgrade. Neither description matched what the bytecode actually shows.

The contract was made immutable on June 3 (UpgradeCap burned). There was no on-chain path to patch or freeze.

## Root cause

Cross-version reserve desynchronization. Two callable versions of the same package write the same `reserve_x` field with values from different token stores.

BlueMove upgraded its AMM package on May 31, moving token storage from `pool.token_x` (a Balance field on the Pool struct) to `EscrowCoinsV2` (a dynamic object field). On Sui, upgrading a package does not retire the old version — both remain callable on the same shared Pool object.

The two versions disagree about where tokens live:

- V1 swap: deposits into `pool.token_x`, sets `reserve_x = pool.token_x.value()`
- V-latest swap: deposits into `EscrowCoinsV2.token_x`, sets `reserve_x = escrow.token_x.value()`

When a V1 swap runs (e.g., through an external aggregator that still references the V1 package), it overwrites `reserve_x` with `pool.token_x.value()` — which is much smaller than the escrow balance. This creates a gap between the escrow (large) and `reserve_x` (small).

The attacker exploited this gap by calling V-latest `add_liquidity_returns` (which computes LP from escrow minus `reserve_x`, inflating the result) followed by V1 `remove_liquidity` (which withdraws from `pool.token_x` based on LP share). Deposit 0.057 SUI into escrow, withdraw ~7,000 SUI from pool.token_x.

## BlueMove's overflow claim

BlueMove called it an arithmetic overflow bug. This is incorrect. Move's integer arithmetic aborts on overflow — it does not wrap. I verified this with three `sui move test` cases: u128 multiplication, u128 addition, and u128→u64 downcast all abort at runtime. An overflow in Move is a DoS, never a fund extraction vector.

## The attack pattern

From the attack transaction's raw JSON, every pool was drained with the same four-command sequence:

```
SplitCoins   (0.057 SUI)
SplitCoins   (meme tokens)
MoveCall     0x35f3190a... / router / add_liquidity_returns   ← V-latest
MoveCall     0xb24b6789... / router / remove_liquidity        ← V1
```

Two different package addresses in the same programmable transaction. V-latest to mint LP (priced against the desynced reserve), V1 to burn LP (withdrawing from pool.token_x).

## Addresses

- Attacker: `0xb29e79198742e84ddf6a5a952238990f7d80565826ff3d97dc92d7056b097335`
- Bridge wallet: `0xa74ee820821d52994785eb9160b3a114a6338de1b3d9e188c00f09ba4f75065e`
- V1 package: `0xb24b6789e088b876afabca733bed2299fbc9e2d6369be4d1acfa17d8145454d9`
- V-latest package: `0x35f3190a41b98da22c997c9266143523816d73a902123dde6f60fac2e0f656d7`

## What I learned

1. Sui's upgrade model preserves old package versions as callable entry points. Any upgrade that relocates storage without disabling old-version access creates a state split.
2. `reserve_x` was a shared mutable field written by two incompatible update functions. The AMM's invariants hold within each version but break across versions.
3. "Arithmetic overflow" is not the right label when the VM aborts on overflow. The actual vulnerability class is closer to stale-state or storage-migration bugs.
4. External aggregators (OKX DEX Router references V1 in its Move.toml) can trigger the desync even without the attacker's involvement — the old package stays live in the ecosystem's routing tables.
5. Making a contract immutable (UpgradeCap burn) after introducing a cross-version bug locks the vulnerability in permanently.

Full analysis: [mehvetero.com/bluemove-was-not-an-overflow-bug](https://mehvetero.com/bluemove-was-not-an-overflow-bug-how-a-cross-version-reserve-desync-drained-714-000-sui)
