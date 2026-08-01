# Typus Finance TLP Oracle — $3.44M (October 2025)

## What happened

An attacker drained $3.44M from Typus Finance's TLP liquidity pool on Sui by setting arbitrary oracle prices. No exploit contract, no flash loan, no reentrancy — just direct calls to a public function that checked authorization and then threw away the answer.

Ten separate manipulation-then-swap sequences emptied the pool in 34 minutes. The stolen tokens (588K SUI, 1.6M USDC, 0.6 xBTC, 32 suiETH) were swapped to USDC on Cetus, Haedal, and Turbos, then bridged to Ethereum via Circle CCTP in 14 transactions.

The oracle module had been deployed for eleven months. MoveBit audited the protocol in May 2025 — the oracle was excluded from scope.

## Root cause

The `update_v2` function in `typus_oracle::oracle` was supposed to restrict price updates to whitelisted addresses. It called the right function — and never checked the result:

```move
public fun update_v2(
    oracle: &mut Oracle,
    update_authority: &UpdateAuthority,
    price: u64,
    twap_price: u64,
    clock: &Clock,
    ctx: &mut TxContext
) {
    // check authority
    vector::contains(&update_authority.authority, &tx_context::sender(ctx));
    version_check(oracle);
    update_(oracle, price, twap_price, clock, ctx);
}
```

Source: [commit `a918e98`](https://github.com/Typus-Lab/typus/blob/a918e98c4f7d3a28d0d809d3263d8c21e90d3c01/typus_oracle/sources/oracle.move), three weeks before the exploit.

`vector::contains()` returns `bool`. That `bool` is computed and discarded. Move does not warn on unused pure-function results. The whitelist existed but was never enforced. Every address on Sui had oracle-write access.

The missing line:

```move
assert!(
    vector::contains(&update_authority.authority, &tx_context::sender(ctx)),
    E_UNAUTHORIZED
);
```

## Why nobody noticed for eleven months

`UpdateAuthority` was a shared object — anyone could reference it. But every legitimate oracle update came from a whitelisted address, so the check "worked" in the sense that authorized callers passed it. It also passed for unauthorized callers. The two populations were indistinguishable to the code.

The module was deployed November 13, 2024. The May 2025 audit explicitly excluded it. The component that controls every price in the system — the one that determines what every swap is worth — was the one nobody reviewed.

## The fix

Five days after the exploit ([commit `4e118f0`](https://github.com/Typus-Lab/typus/blob/4e118f0f4c5d8b09837b3d0dcfb6501dc79f687f/typus_oracle/sources/oracle.move)), Typus replaced `update_v2` entirely with `update_with_update_cap`:

```move
public fun update_with_update_cap(
    oracle: &mut Oracle,
    update_cap: &UpdateCap,
    price: u64,
    twap_price: u64,
    clock: &Clock,
    ctx: &TxContext,
) {
    version_check(oracle);
    assert!(update_cap.`for` == ctx.sender(), EInvalidUpdateCap);
    update_(oracle, price, twap_price, clock, ctx);
}
```

`UpdateCap` binds a `for: address` field to a specific sender. But it is still a shared object (`transfer::share_object()` in `create_update_cap`) — the access control is still a runtime `assert!`, not ownership-based. The codebase already has an owned-capability pattern (`ManagerCap` via `transfer::transfer`); `UpdateCap` was likely kept shared because multiple cranker addresses need concurrent access.

The entire defense is still one `assert!`. That is worth staring at: a protocol that lost $3.44M to a missing check was fixed by adding a check.

## The attack path

The attacker set one oracle price to 651,548,270 and another to 1. With a 651-million-to-1 ratio, 1 SUI swapped for 60 million xBTC base units (0.6 xBTC at 8 decimals). Ten rounds emptied the pool. Initial funding: 0.041 ETH from Tornado Cash on BSC.

Timeline:
- 13:05 UTC — first malicious tx (`6KJvWtmrZDi5MxUPkJfDNZTLf2DFGKhQA2WuVAdSRUgH`)
- 13:24 UTC — team alerted by community
- 13:39 UTC — contracts paused
- 13:42 UTC — root cause identified (three minutes after pausing)

## What I learned

1. **An unused return value is invisible in Move.** Rust has `#[must_use]`. Move does not. A `bool` has the `drop` ability, so discarding it is silent. A `vector::contains()` call without `assert!` compiles, passes tests, and ships — the compiler gives no signal that the result matters. This is a language-level gap.

2. **Shared-object authorization is fragile by design.** `UpdateAuthority` was shared — anyone could reference it in a transaction. The authorization logic ran inside the function, not at the object-access level. The owned-capability pattern (`ManagerCap`) would have prevented the exploit structurally, but the team chose shared for operational reasons (multiple updaters). When you pick shared over owned, the `assert!` is load-bearing. There is no backup.

3. **The unaudited module was the highest-value target.** Same pattern as Aftermath (perps module added late, audited months later) and Cetus (arithmetic overflow in a library function). The module that controls the most money is systematically the one that ships outside audit scope.

4. **The comment was the trap.** The code said `// check authority` right above a line that did not check authority. The comment matched the developer's intent. The code did not. Comments cannot substitute for `assert!`.
