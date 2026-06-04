---
name: solidity-foundry-style
description: Use when writing, refactoring, or testing Solidity and Foundry code for the Arc Water Rights Futures Prototype. Applies Arc-specific conventions for explicit storage layout, bitmap replay protection, uint8 status enums, Foundry TDD, OpenZeppelin usage, no per-fill lots onchain, Fireblocks custody boundaries, the docs/reference protocol invariant registry, and audit-preventive DeFi safety patterns before formal solidity-audit review.
---

# Solidity Foundry Style

Use this skill for Solidity contract, interface, library, deployment, and Foundry test work in the Arc Water Rights Futures Prototype, especially under `onchain/`.

This is a build-time style and safety guide. It should prevent common audit findings before invoking `solidity-audit`, `solidity-audit-dex`, `solidity-audit-lending`, or `solidity-audit-staking`.

## First Moves

1. Read the local `AGENTS.md`, `onchain/foundry.toml`, and nearby contracts/tests before editing.
2. Read `docs/reference/protocol-invariants.md` before Solidity edits and identify the invariant IDs the change can affect.
3. Preserve the prototype boundaries: futures-market-only exchange, offchain matching, onchain clearing, USDC-only collateral, Fireblocks client custody, and netted positions.
4. Use TDD for behavior changes: add or update a failing Foundry test first, implement the smallest contract change, then run focused tests.
5. Keep changes scoped. Do not add isolated margin, portfolio margin, hedge mode, complex fee tiers, token discounts, custody abstractions, or per-fill onchain lots unless the user explicitly changes product scope.

## Invariant Registry

The project source of truth for protocol invariants is `docs/reference/protocol-invariants.md`.

Before implementation:

- Read the registry and list the impacted invariant IDs in your working notes.
- Preserve every `Enforced` invariant.
- Treat `Planned` and `Strengthening` invariants as forward-compatibility constraints.
- If the requested change conflicts with an invariant, stop and explain the conflict before editing.

During implementation:

- Add or update Foundry tests for impacted invariant IDs.
- Keep invariant IDs stable in comments, test names, or commit/PR notes where useful.
- Do not silently remove, weaken, or reclassify an invariant. Update the registry with rationale when a design intentionally changes one.

Before finishing:

- Report which invariant IDs were affected and which focused tests were run.
- If broader invariant tests were not run, say so explicitly.

## Project Style

- Solidity version follows `onchain/foundry.toml` unless the repo changes it. Current style uses `pragma solidity 0.8.28;`.
- Use official Solidity style guide ordering with clear section banners when the surrounding file does.
- Prefer custom errors over revert strings.
- Emit events for every successful state change. Emit after state has been written.
- Use NatSpec for public/external functions, roles, state variables with storage significance, and non-obvious security decisions.
- Prefer small internal helpers and libraries for repeated bit math or guard logic; do not introduce abstractions for one-off code.
- Keep interfaces in `onchain/src/interfaces/` synchronized with implementation behavior and events.

## OpenZeppelin

- Prefer OpenZeppelin for standard, battle-tested primitives: `AccessControlEnumerable`, `IERC20`, `SafeERC20`, `ReentrancyGuard`, `Pausable`, and related interfaces.
- Use role checks for privileged protocol actions. Check `msg.sender`, never a caller-supplied admin/operator address.
- Validate constructor and setter addresses with a shared guard or explicit zero-address check.
- Use `SafeERC20` for token transfers and account for non-standard ERC20 behavior. For USDC, remember 6-decimal accounting when converting to 18-decimal risk or price math.
- Do not rebuild Fireblocks RBAC in contracts. Contracts should validate protocol roles, signatures, settlement proofs, and account-manager bindings only where those are onchain responsibilities.

## Storage Layout

Make storage layout explicit and durable.

- Document storage layout in contract NatSpec for any contract with persistent protocol state.
- Pack related fields deliberately. Position state should stay netted and compact; collateral records should avoid unnecessary slots.
- Use `uint8` status enums instead of multiple booleans for lifecycle state. Store enum/status fields in packed structs where possible.
- Avoid upgradeable proxy patterns unless already present. If proxies are introduced, reserve gaps and explicitly verify storage collision risks.
- Never store per-fill lots onchain. Store netted account/market position, aggregate variation margin, funding, collateral, fee, and replay state.
- Avoid unbounded arrays or loops over user-controlled state in settlement, liquidation, collateral, and admin paths.

Preferred packed lifecycle shape:

```solidity
enum PositionStatus {
    None,
    Open,
    Settled,
    Liquidating,
    Closed
}

struct Position {
    int128 quantity;
    int128 entryValue;
    int128 realizedPnl;
    uint64 lastFundingTime;
    uint32 maturity;
    uint8 status;
}
```

Use the exact fields that fit the local contract, but keep the discipline: netted state, explicit widths, documented slot intent.

## Bitmap Replay Protection

Use bitmaps for order nonces, settlement batch IDs, claim IDs, and other replay domains.

- Map replay state by domain first, then word: `mapping(bytes32 => mapping(uint256 => uint256))`.
- Derive `wordIndex = id >> 8` and `bitIndex = uint8(id)` or use the repo's `Bitmap` library.
- Scope replay domains by account, market, batch type, settlement coordinator, or chain/context as needed. A nonce valid in one domain must not block or authorize another.
- Cancellation should permanently block later consumption. If using separate consumed/cancelled words, cancelled nonces generally set both bits.
- Test nonce `0`, adjacent bits, bit `255`, bit `256`, large IDs, cross-account/domain independence, double consume, double cancel, and consume-after-cancel.
- Prefer bitmap reads/writes over `mapping(uint256 => bool)` for per-ID replay protection.

## Custody Boundary

Fireblocks is authoritative for client custody and approvals.

- CMDT/provider contracts must not become the custody operator or signer of client assets.
- Do not encode Fireblocks workspace RBAC, user permissions, or approval workflows onchain.
- Onchain code may enforce protocol roles, whitelisted settlement actors, account-manager bindings, EIP-712 domains, signatures/proofs, and collateral movement rules.
- Settlement should assume fills are produced offchain, micro-batched, committed, and then cleared onchain. Contracts verify commitments and replay protection; they do not run a matching engine.
- Client deposits/withdrawals should be explicit, evented, access-controlled where appropriate, and compatible with client-owned Fireblocks workflows.

## Futures Accounting

Design for institutional futures clearing, not AMM spot trading.

- Store netted positions per account/market/maturity. Do not store individual fills or fill lots.
- Keep collateral, fee, variation margin, funding, and liquidation state synchronized across contracts.
- Apply accrual/checkpoint logic before balance, position, liquidation, or settlement mutations.
- Make liquidation liveness a first-class invariant: an unhealthy account must remain liquidatable unless a documented global halt applies.
- Bound all basis-point parameters and risk coefficients. Reserve factors, fees, liquidation penalties, and margin rates must not exceed their explicit maximums.
- Favor protocol-conservative rounding. Rounding should not let users extract value across repeated settlement, funding, fee, liquidation, or collateral flows.

## Audit-Preventive Guardrails

Before writing code, identify the invariants the function must preserve. At minimum, consider:

- CEI and reentrancy: checks first, storage writes next, external calls last; use `nonReentrant` for stateful token/callback paths.
- Token accounting: recorded collateral/liabilities must reconcile with token balances and account totals.
- Access control: every setter, pause/halt, settlement, oracle, liquidation, vault, and fee action must have the intended role or per-account authorization.
- Event completeness: every state-changing success path emits accurate post-mutation values.
- Replay resistance: signatures, nonces, and batch IDs are scoped and one-use.
- Oracle safety: reject stale, zero, negative, or out-of-window prices; prefer TWAP/VWAP windows where the protocol expects them.
- Flash-loan resistance: no single transaction should profit by manipulating price, collateral, funding, or fee state.
- Cross-contract sync: when one contract updates settlement, collateral, fees, margin, or liquidation state, dependent contracts must observe a coherent state.
- Precision: multiply before divide; document scaling units; avoid lossy casts; use `SafeCast` where narrowing is necessary.
- Admin risk: privileged changes should be bounded, evented, and routed through local timelock/halt patterns where the repo already uses them.
- Denial of service: avoid push-payment loops, user-controlled iteration, and receiver-callback dependencies in critical flows.

DEX/lending/staking audit skills are reference material, not product scope expansion. Import only applicable lessons:

- From DEX: guard read-only reentrancy in price feeds and never use manipulable spot prices for margin.
- From lending: enforce health/margin checks after collateral removal and before non-liquidating exits; liquidations must not be blockable by the borrower.
- From staking/reward systems: checkpoint global accumulators before per-account balance changes; reward/funding indexes should be monotonic unless an explicit slash/settlement event explains otherwise.

## Foundry TDD

For each behavior change:

1. Add a focused test under `onchain/test/` matching nearby naming style.
2. Start with the failing security/behavior case, then add the success path.
3. Prefer exact custom-error selectors with `vm.expectRevert`.
4. Assert storage state, emitted events, role boundaries, and token balances.
5. Map tests to impacted invariant IDs from `docs/reference/protocol-invariants.md`.
6. Include boundary tests for packed integers, status transitions, bitmap word edges, stale prices, zero values, decimal scaling, and authorization failures.
7. Add invariant or fuzz tests when the change affects accounting, liquidation, replay, or cross-contract synchronization.
8. Run the narrow test first, then broader `forge test` when feasible.

Useful command pattern:

```bash
cd onchain && forge test --match-contract ContractNameTest -vv
```

## Done Criteria

A Solidity change is not done until:

- Impacted invariant IDs from `docs/reference/protocol-invariants.md` are identified.
- The storage layout remains documented and intentionally packed.
- Replay protection uses scoped bitmaps where IDs/nonces are involved.
- Status/lifecycle state uses `uint8` enum-style fields, not scattered booleans.
- No per-fill lots or matching-engine responsibilities moved onchain.
- Fireblocks custody boundaries are preserved.
- Access control, oracle freshness, token accounting, liquidation/funding/fee invariants, and event emission have tests mapped to impacted invariant IDs.
- No `Enforced` invariant is weakened, and no `Planned` or `Strengthening` invariant is made harder to implement without an explicit registry update.
- Focused Foundry tests pass, and any skipped broader tests are called out clearly.
