# ft-core

## Overview

FT-Core is a Rust-based blockchain fungible token (FT) management library that provides core functionality for handling token operations across THX Network's multi-chain ecosystem. It manages both native tokens and custom fungible tokens across different blockchain networks, with built-in safeguards against race conditions using Redis-backed atomic operations.

## Tech Stack

- **Language:** Rust 2024 Edition
- **Runtime:** Tokio (async/await)
- **Blockchain:** subxt (Substrate/Polkadot RPC), sp-core (SS58, AccountId32)
- **Cache:** Redis with Lua scripting for atomic operations
- **Serialization:** serde, Protocol Buffers (gRPC)
- **Precision:** rust_decimal for financial calculations
- **Observability:** tracing, tracing-subscriber

## Features

- **Native Token Management:** Query balances, transfer tokens, paginated transfer history
- **Non-Native Token Management:** Multi-asset balance queries, custom token transfers
- **Race Condition Prevention:** Atomic Redis operations with Lua scripting, dual-cache system
- **Balance Details:** Total, free, transferable, locked, bonded, redeemable amounts
- **Transaction Tracking:** Track ID-based transaction finality status
- **Testnet Utilities:** Token faucet for development
- **Multi-Chain Support:** L0 (rootchain) and L1 (leaf-chain) network abstraction

## Architecture

The library follows a **Facade Pattern** with **Layered Architecture**:

```
ft-core/
├── src/
│   ├── lib.rs                          # Module exports
│   ├── ftcore.rs                       # FtCore struct (main API facade)
│   ├── utils.rs                        # Decimal conversion, SS58 validation
│   ├── entity/                         # Data models
│   │   ├── asset.rs                    # AssetEntity (token metadata)
│   │   └── redis_config.rs             # Configuration structs
│   ├── ft_redis.rs                     # Redis wrapper with Lua atomics
│   ├── collections/                    # Asset configuration per network
│   │   └── ft_non_native_collection.rs
│   └── helpers/                        # Operation implementations
│       ├── transfer_native_token.rs
│       ├── transfer_non_native_token.rs
│       ├── get_native_token_balance.rs
│       ├── get_non_native_token_balance.rs
│       ├── get_balance_details.rs
│       └── ...
└── docs/                               # RPC specification docs
```

## Key Technical Decisions

- **Precision Handling:** Decimal type for all financial calculations (avoiding f64 precision loss)
- **Dual-Cache Strategy:** DB cache + blockchain cache with reconciliation on stale data
- **SS58 Validation:** Early address validation before network calls
- **Concurrent Safety:** Lua scripts for atomic Redis operations (prevents TOCTOU bugs)
- **Async-First:** All I/O operations are non-blocking
- **gRPC Integration:** Responses mapped to Protocol Buffer message types

## API Reference

### Native Token Operations
- `get_native_balance_human_value()` - Query balance (10 decimals)
- `transfer_native_token()` - Atomic transfer with balance preview
- `get_my_native_token_transfer_history()` - Paginated history (1-500 items)

### Non-Native Token Operations
- `get_non_native_balance()` - Query multiple asset balances
- `transfer_non_native_token()` - Transfer with per-asset rate limiting
- `get_my_non_native_token_transfer_history()` - Asset-specific history

### Utilities
- `get_balance_details()` - Comprehensive balance breakdown
- `check_transaction_result_with_track_id()` - Query finality status
- `get_sand_faucet()` - Testnet token faucet
