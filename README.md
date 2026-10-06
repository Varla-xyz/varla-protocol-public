# Varla Protocol

**Liquidity for prediction market positions.**

Varla is a lending protocol that allows prediction market traders to use their positions as collateral to borrow stablecoins — without having to sell or wait for the market to resolve.

Prediction market positions can lock capital for weeks or months. Varla turns that locked capital into usable liquidity.

> Deposit prediction market positions → borrow against them → keep your market exposure.

---

## What is Varla?

A trader holding a prediction market position normally has two choices:

1. sell the position and lose the exposure, or  
2. wait until the market resolves.

Varla introduces a third option: **use the position as collateral**.

Users can:

- deposit supported prediction market positions;
- borrow stablecoins against their combined portfolio;
- maintain their prediction market exposure;
- repay and withdraw their positions later.

Liquidity is provided by lenders through the Varla lending pool.

---

## How it works

```text
                 ┌─────────────────────┐
                 │       Lenders       │
                 │  Provide liquidity  │
                 └──────────┬──────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  Varla Pool   │
                    └───────┬───────┘
                            │
                         stablecoins
                            │
                            ▼
┌────────────────┐    ┌───────────────┐
│ Prediction     │───▶│  Varla Core   │
│ Market Shares  │    │ Cross-margin  │
└────────────────┘    └───────┬───────┘
                              │
                    health factor / risk
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
        ┌──────────────┐            ┌──────────────┐
        │ Varla Oracle │            │ Liquidations │
        └──────────────┘            └──────────────┘
```

### 1. Deposit

A user deposits supported prediction market shares into Varla.

### 2. Collateral valuation

Varla evaluates the collateral using market prices, liquidity, and risk parameters.

All supported positions are combined into a **cross-margin account**.

### 3. Borrow

The user can borrow stablecoins according to the collateral value and the risk tier of each position.

### 4. Monitor

The protocol continuously evaluates the account's health factor as market conditions change.

### 5. Repay or liquidate

Users can repay their debt and withdraw their positions at any time.

If an account becomes undercollateralized, permissionless liquidators can repay part of the debt in exchange for discounted collateral.

---

## Why prediction markets need lending

Prediction markets are becoming increasingly liquid, but the capital deposited into them remains largely isolated.

A trader might hold $50,000 worth of positions while still needing additional capital for:

- new prediction market opportunities;
- hedging;
- trading;
- liquidity management;
- other on-chain activity.

Varla aims to make prediction market positions **capital-efficient financial assets**.

Instead of waiting for settlement:

```text
Prediction Market Position
          ↓
      Collateral
          ↓
        Credit
          ↓
   Additional Liquidity
```

---

## Protocol architecture

Varla combines **on-chain smart contracts** with supporting **off-chain services** for market data, oracle updates, liquidations, indexing, and analytics.

### On-chain smart contracts

The core lending logic currently runs on **Polygon**.

#### Varla Core

The main borrower accounting and collateral management contract.

It:

- holds deposited Polymarket positions;
- tracks user debt;
- calculates borrowing capacity and health factors;
- handles deposits, withdrawals, borrowing, and repayments;
- coordinates collateral seizure during liquidations.

#### Varla Pool

An **ERC-4626 lending vault** that manages lender liquidity.

It:

- receives stablecoin deposits from lenders;
- issues vault shares representing lender positions;
- provides liquidity to borrowers through the Varla Core;
- accrues borrower interest;
- maintains the protocol reserve used as the first layer of protection against bad debt.

#### Varla Oracle

The on-chain market and risk registry used by the protocol.

It stores and manages:

- supported prediction market positions;
- collateral risk tiers;
- resolution dates;
- price, TWAP, and liquidity data;
- collateral validity and effective LTV parameters.

The Varla Core reads this data when calculating borrowing power and account health.

#### Liquidation Contracts

Permissionless smart contracts handle undercollateralized accounts.

Liquidators can repay part of a borrower's debt and receive prediction market collateral at a discount.

Varla also includes alternative liquidation flows for complementary and multi-outcome positions.

#### Supporting contracts

Additional smart contracts handle:

- signed oracle update verification;
- dynamic interest rate calculation;
- protocol roles and permissions;
- contract upgrades.

---

### Off-chain services

Varla also runs several off-chain services that support the on-chain protocol.

#### Oracle Updater

Reads Polymarket market data including order books, prices, historical pricing, and liquidity.

It calculates the data used by Varla's risk system and publishes signed updates to the on-chain oracle.

#### Liquidation Keeper

Monitors borrower accounts and identifies positions that become liquidatable.

When required, it executes liquidations through the on-chain liquidation contracts and manages the acquired collateral.

Liquidations remain permissionless — the keeper is an automated participant, not a requirement of the protocol.

#### Analytics Indexer & Gateway

Indexes on-chain protocol events and stores structured protocol data.

It provides the data layer used by the liquidation keeper, analytics services, and frontend application.

#### Gems Indexer

Tracks protocol activity and calculates points for the Varla Gems rewards system.

---

### Frontend

The Varla web application provides the interface for:

- depositing prediction market positions;
- borrowing and repaying stablecoins;
- providing and withdrawing liquidity;
- monitoring collateral and account health;
- managing leveraged prediction market positions.

---

### External prediction market infrastructure

Varla currently integrates directly with **Polymarket**.

Polymarket provides:

- the ERC-1155 outcome tokens used as collateral;
- the CLOB used for market pricing and liquidity data;
- market metadata and resolution information;
- the venue where seized collateral can be sold after liquidation.

The current Varla architecture therefore combines **Varla's own lending and risk infrastructure** with Polymarket's prediction market settlement and trading infrastructure.

---

## Risk engine

Prediction market collateral behaves differently from traditional crypto collateral.

Positions can experience sudden repricing when new information becomes available.

Varla uses multiple risk controls, including:

- market whitelisting;
- collateral-specific LTVs;
- liquidity requirements;
- conservative collateral pricing;
- continuously updated health factors;
- decreasing borrowing power close to market resolution;
- permissionless liquidations;
- a protocol reserve for bad debt.

Different markets can be assigned different risk tiers depending on liquidity, maturity, and market characteristics.

---

## Cross-margin

Varla treats supported prediction market positions as a portfolio rather than requiring an isolated loan for every position.

```text
Position A ─┐
Position B ─┼──▶ Varla Account ──▶ Borrowing Power
Position C ─┘
```

This allows traders to use the combined value of their supported positions as collateral.

---

## Lending

Liquidity providers deposit stablecoins into the Varla Pool.

Borrowing rates dynamically change according to pool utilization.

```text
Lenders
   ↓
Varla Pool
   ↓
Borrowers
   ↓
Interest
   ↓
Lenders + Protocol Reserve
```

This creates a native credit market around prediction market collateral.

---

## Liquidations

Each borrower has a health factor based on:

```text
Collateral Value × Effective LTV
────────────────────────────────
              Debt
```

When the health factor falls below the required threshold, the account becomes liquidatable.

Liquidators can repay part of the outstanding debt and receive collateral at a discount.

This mechanism helps protect lender liquidity from undercollateralized positions.

---

## Current status

Varla is currently in **public beta**, following an initial private beta phase used to validate the protocol's core mechanics in a controlled environment, including lending and borrowing flows, collateral management, oracle updates, risk parameters, and liquidation behavior.

The current implementation runs on **Polygon** and uses **Polymarket positions as collateral**.

The protocol currently includes:

- lending and borrowing;
- prediction market collateral;
- cross-margin accounts;
- dynamic interest rates;
- collateral risk tiers;
- oracle infrastructure;
- permissionless liquidations;
- bad-debt handling;
- protocol indexing;
- liquidation automation;
- web application.

The public beta is focused on testing the protocol under broader real-world usage, validating the risk model and infrastructure, and gathering operational data before increasing protocol limits and expanding support to additional markets and chains.

The next phase of development focuses on expanding Varla to **Solana** and evolving the protocol into a venue-agnostic lending layer for prediction market positions.

---

## Repository structure

This public repository provides the technical overview used for Varla's Colosseum submission.

The active product is maintained across multiple private repositories:

```text
varla-protocol
Smart contracts, SDK, deployment tooling,
and core protocol logic

varla-api
Oracle updater, liquidation keeper,
indexing, analytics, and backend services

varla-frontend
User-facing web application
and protocol interaction flows
```

Access to the private repositories can be provided to the Colosseum team if required.

---

## Tech stack

**Smart Contracts**

- Solidity
- OpenZeppelin
- ERC-4626
- ERC-1155 collateral

**Frontend**

- Next.js
- TypeScript

**Backend**

- TypeScript
- PostgreSQL
- Ponder
- GraphQL

**Infrastructure**

- Railway
- Vercel
- Neon

**Prediction Markets**

- Polymarket CTF
- Polymarket CLOB

---

## Roadmap

Varla's development roadmap includes:

- expanding Varla into a **venue-agnostic lending layer on Solana**, starting with prediction market positions accessible through **Jupiter** and tokenized **Kalshi** markets;
- improving oracle decentralization;
- increasing liquidation infrastructure resilience;
- introducing additional collateral risk models;
- completing third-party security audits;
- progressively increasing protocol caps;
- broader protocol decentralization.

---

## Vision

Prediction markets are evolving from simple betting venues into a new financial primitive.

But their positions are still largely capital inefficient.

Varla's goal is to build the **credit layer for prediction markets** — allowing positions to become productive collateral across on-chain finance.

**Prediction markets create information assets.  
Varla makes those assets liquid.**

© 2026 Varla
