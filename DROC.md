# DROC: Decentralized Resting Order Commitments

## Enhancing On-Chain Liquidity Transparency Across the Solana Ecosystem

**Author:** @SHA256
**Date:** July 7, 2025
**Proposal Type:** Ecosystem Protocol Enhancement
**Status:** Final Draft for Community Proposal

---

## Executive Summary

Solana’s on-chain trading infrastructure is rapidly evolving. While Just-In-Time (JIT) liquidity, AMMs, and hybrid models have driven composability and efficiency, they often obscure pre-trade visibility of liquidity. Traders across Solana DeFi face uncertainty about order book depth, slippage, and execution reliability.

**DROC (Decentralized Resting Order Commitments)** introduces a composable primitive for any Solana-based DEX, aggregator, or trading interface: short-lived, collateral-backed resting orders that are transparently visible before execution. DROC improves user trust, strategic execution, and staking-aligned capital efficiency, while complementing existing models like Drift's DLOB, Phoenix’s CLOB, Gavel’s AMM, SolFi’s hybrid system, and routing aggregators like Jupiter and Meteora.

By integrating with Solana-native assets such as JitoSOL and leveraging Solana's parallel runtime, DROC provides a forward-compatible liquidity commitment layer for the entire ecosystem—not just one protocol.

---

## Problem Statement

### Systemic On-Chain Liquidity Challenges

* **Invisible Liquidity**: JIT makers only reveal liquidity post-match, reducing confidence for traders pre-execution
* **Price Discovery Gaps**: AMMs expose predictable curves but lack price-time priority and resting order precision
* **Uncertain Execution**: Traders can't gauge execution likelihood when interacting with opaque books or aggregators
* **MEV Exposure**: Lack of visible commitments invites toxic flow, sandwiching, or adverse selection

These limitations affect:

* Institutional players seeking predictable execution and visible depth
* Retail traders trying to understand order book behavior
* Aggregators and smart order routers seeking consistent liquidity signals
* Builders optimizing for composable and efficient liquidity layers

---

## Market Microstructure: Solana’s Diverse Design Space

Solana has become a proving ground for high-performance market structure experimentation. Builders are exploring alternatives to the FIFO-dominated CLOB model of traditional finance:

* **Phoenix** offers a pure on-chain CLOB with near-instantaneous matching but suffers from limited liquidity incentives.
* **Drift** leverages a distributed order book with JIT liquidity, reducing gas but obscuring book visibility.
* **Gavel** introduces a MEV-resistant AMM tuned for Solana's leader rotation, prioritizing fairness over resting depth.
* **SolFi** uses a liquidity-efficient AMM-CLOB hybrid tailored for deep spot markets.
* **Jupiter & Meteora** route across these systems but are often blind to non-committed liquidity.

These are **purpose-built**, but none fully solve the **transparency vs. performance** tradeoff.

**DROC fills that gap**—a modular extension to any engine that introduces time-bound, collateralized, visibly resting orders, enabling:

* Transparent, predictable liquidity depth
* Time-sensitive maker commitments
* Aggregator-level visibility for better routing
* Incentivized staking-based liquidity provisioning

---

## Solution: DROC Framework

### Core Mechanics

DROC introduces the following features, composable across protocols:

1. **Collateralized Commitments**: Makers lock SOL or liquid staking derivatives (e.g., JitoSOL) when placing an order
2. **Time-Bound Orders**: Short lifespan (e.g., 10 blocks) ensures rapid refresh and capital rotation
3. **Pre-Trade Visibility**: Orders appear in compatible interfaces and aggregators as active liquidity
4. **Auto-Matching**: Matched orders settle via native DEX logic or aggregator integrations

DROC can plug into existing order flow via wrappers, or be integrated natively into engines with minimal overhead.

### Why Not FIFO?

Traditional CLOBs follow **FIFO (First-In, First-Out)**: earliest orders at a price level fill first. It's simple and predictable, like a vending machine.

But in crypto, FIFO often leads to:

* Latency wars: bots compete in nanoseconds
* Queue games: constant reordering and cancel/resubmit
* Inefficient capital: early arrival matters more than order quality

Solana's block-level determinism and parallelism allow for **new queuing logic**—DROC prioritizes **commitment, not latency**.

---

## Technical Implementation

### DROC Order Structures

```rust
#[account]
pub struct DROCOrder {
    pub maker: Pubkey,
    pub collateral: u64,      // JitoSOL collateral
    pub price: u64,           // Price per unit
    pub quantity: u64,        // Order size
    pub expiry: i64,          // 10-block expiry
    pub market: Pubkey,       // Market identifier
    pub order_type: OrderType, // Buy/Sell
    pub filled: bool,
}

#[account]
pub struct RestingLiquidityVault {
    pub total_collateral: u64,
    pub active_orders: u64,
    pub jito_rewards_collected: u64,
    pub authority: Pubkey,
}
```

---

## Oracle Integration

DROC relies on Solana-native oracles such as Pyth and Switchboard for:

* Price safety checks (ensure orders are within X% of market)
* Expiry slot conversion
* Collateral valuation and yield accrual tracking

Example logic:

```rust
let price_data = load_price(&ctx.accounts.oracle)?;
let spot_price = price_data.get_current_price_unchecked();
require!(
    order.price >= spot_price * 95 / 100 &&
    order.price <= spot_price * 105 / 100,
    ErrorCode::InvalidOrderPrice
);
```

---

## Aggregator Compatibility

DROC orders are designed for integration with Solana routing layers. Aggregators like Jupiter and Meteora can:

* Read DROC resting orders as part of best-route calculations
* Use oracle feeds or contract event streams for order visibility
* Benefit from predictable pre-trade liquidity data for better UX

DROC acts as a transparency layer between fragmented liquidity sources.

---
