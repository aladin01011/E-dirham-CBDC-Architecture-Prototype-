# Digital Dirham: Technical Architecture (v2.0)

**Status**: Research Phase
**Project Type**: Personal study architecture

> **Disclaimer**  
> This document is part of an independent personal CBDC study based on public sources.  
> It is not an official Bank Al-Maghrib document and contains no confidential Bank Al-Maghrib information.

## 1. System Overview

```
┌─────────────────────────────────────────────────────────────┐
│           DIGITAL DIRHAM ARCHITECTURE (Layered)            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  LAYER 1: DEVICE/SECURE ELEMENT (Offline-First)           │
│  ├─ SIM Card Applet (Java Card, 50-64KB)                  │
│  ├─ eSIM Partition (TEE-backed, smartphones)              │
│  ├─ Hardware Wallet (Optional, Thales HSM)                │
│  └─ Operations: CREATE_TX, RECEIVE_TX, GET_BALANCE        │
│                                                             │
│  LAYER 2: APPLICATION/UX                                  │
│  ├─ Smartphone App (Go SDK, iOS/Android)                 │
│  ├─ USSD Menu (*997# for feature phones)                 │
│  ├─ Merchant PoS (Android)                               │
│  └─ Operations: SELECT_RECIPIENT, CONFIRM, SYNC          │
│                                                             │
│  LAYER 3: CONNECTIVITY (Multi-Modal)                      │
│  ├─ Online: LTE/3G → Go API Gateway (real-time)          │
│  ├─ Offline: NFC D2D (direct phone-to-phone)            │
│  ├─ Deferred: Agent batch via USSD                       │
│  └─ Latency: 0ms (offline) to 100ms (LTE)                │
│                                                             │
│  LAYER 4: SETTLEMENT ENGINE (Rust)                        │
│  ├─ Validate signatures (Ed25519)                        │
│  ├─ Detect double-spend (Bloom filter)                   │
│  ├─ Atomic settlement (UTXO model)                       │
│  ├─ Throughput: 50K-100K TPS                             │
│  └─ Latency: sub-millisecond local target (lab scope)    │
│                                                             │
│  LAYER 5: LEDGER (PostgreSQL, 3-AZ)                       │
│  ├─ Primary (Rabat): Canonical state                     │
│  ├─ Standby (Casablanca): Sync replication              │
│  ├─ ReadReplica (Tangier): Async analytics              │
│  └─ Durability: WAL-based, point-in-time recovery       │
│                                                             │
│  LAYER 6: AUDIT (Hyperledger Fabric, 7-node)            │
│  ├─ BFT Consensus (5 of 7 required)                      │
│  ├─ Tamper-proof verification                           │
│  ├─ Latency: 1-3 seconds (async)                        │
│  └─ Role: Independent oversight                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 2. Key Components

### 2.1 Secure Element (SE)

**Deployment Options:**
- SIM Card Applet (Java Card): Standard, widely compatible
- eSIM Partition (TEE): Modern smartphones only
- Hardware Wallet: Premium security (optional)

**Persistent State (64KB Total):**
```
Private Key (Ed25519)      : 32 bytes
Blind Sig Key (RSA-2048)   : 256 bytes
Transaction Counter        : 4 bytes
Nonce Cache (Bloom)        : 2KB
Recent Tx Log              : 10KB
Device ID                  : 16 bytes
Free Space                 : ~51KB
```

### 2.2 Rust Settlement Engine

**Core Functions:**
- Verify Ed25519 signatures
- Check Bloom filter for duplicates
- Validate payer balance
- Atomic UTXO-style updates
- Create settlement blocks (5-of-7 consensus)
- Async publish to Fabric

**Performance:**
- Throughput: 50K-100K TPS
- Latency (P99): <500 microseconds for local in-process benchmark path (not WAN end-to-end)
- Memory: <100MB
- CPU: hardware-dependent; Ed25519 verification is typically far above 1,000 cycles (benchmark before production claims)

### 2.3 Go API Gateway

**Endpoints:**
```
POST   /api/v1/transfer              # P2P transfer
POST   /api/v1/reconcile             # Offline sync
GET    /api/v1/wallet/{id}/balance   # Balance query
POST   /api/v1/merchant/payment      # PoS payment
POST   /api/v1/ussd/menu             # USSD gateway
```

**Performance:**
- Throughput: 100K+ req/sec per instance
- Concurrent: 1M+ connections
- Latency (P99): <100ms

### 2.4 PostgreSQL Ledger

**Core Tables:**
- wallets (balance, kyc_tier, se_id)
- transactions (payer, recipient, amount, status)
- settlement_blocks (merkle_root, validator_sigs)
- double_spend_log (attempts)

**Replication (3-AZ):**
- Primary (Rabat): All writes
- Standby (Casablanca): Synchronous
- ReadReplica (Tangier): Async

### 2.5 Hyperledger Fabric

**Network (7 Nodes):**
- 3 Orderers (Rabat)
- 2 Peers (Casablanca + Independent)
- 2 Peers (Tangier + External)

**Consensus:** BFT (5 of 7)
**Block Time:** 5 seconds
**Fault Tolerance:** Survives 2 failures

## 3. Transaction Processing

### Online Real-Time
```
T+0ms   User app → Send Money
T+2ms   → Go API Gateway
T+3ms   → Rust Settlement Engine
T+4ms   ├─ Verify signature
        ├─ Check Bloom
        ├─ Query balance
        └─ Atomic update
T+5ms   ← Return confirmed
T+8ms   User sees: "✓ Sent"
```

### Offline NFC
```
T+0min   Payer taps Payee (NFC)
T+1min   Both sign, deduct/add balance
T+2min   Both show: "Pending Sync"

T+120min Merchant comes online
T+125min Rust settles transaction
T+126min Merchant sees: "✓ Confirmed"

Result:  Local settlement: Instant
         Central settlement: ~125 min (first sync)
         Audit trail: Immutable (Fabric)
```

## 4. Double-Spend Prevention

| Layer | Mechanism | Coverage |
|-------|-----------|----------|
| 1 | Device-level enforcement | <10% (SE compromise risk) |
| 2 | Cryptographic nonce | Replay-resistance control (illustrative estimate) |
| 3 | Transaction counter | Illustrative low violation-rate target (monotonic checks) |
| 4 | Bloom filter dedup | False positives may temporarily flag/reject honest tx; reconciliation resolves final status |
| 5 | Balance reconciliation | 100% (post-hoc detection) |

**Combined:** Illustrative low undetected double-spend objective; false positives increase rejects/friction, not successful double-spends

## 5. Cryptographic Algorithms

| Operation | Algorithm | Key Size | Note |
|-----------|-----------|----------|------|
| Signing | Ed25519 | 256 bits | Fast, no GC |
| Privacy | RSA + Blind | 2048 bits | Merchant-unlinkable |
| ZKP | Groth16 | Bls12-381 | Balance proofs |
| Hash | SHA3-256 | 256 bits | Collision-resistant |
| Encrypt | AES-256-GCM | 256 bits | NIST-approved |
| HMAC | HMAC-SHA256 | 256 bits | Auth codes |

## 6. Deployment

### Infrastructure (3-AZ)

**AZ1: Rabat**
- PostgreSQL Primary: 32 CPU, 256GB, 4TB NVMe
- Rust Settlement: 16 CPU, 64GB
- Go API: 8 CPU, 32GB (scalable)
- Fabric Orderer: 8 CPU, 32GB

**AZ2: Casablanca**
- PostgreSQL Standby: 32 CPU, 256GB, 4TB
- Fabric Peer (BAM): 8 CPU, 32GB
- Redis: 8 CPU, 96GB

**AZ3: Tangier**
- PostgreSQL ReadReplica: 16 CPU, 128GB, 2TB
- Elasticsearch: 8 CPU, 64GB × 3 nodes
- Fabric Peer: 8 CPU, 32GB

## 7. Performance Targets

| Metric | Target |
|--------|--------|
| Settlement Latency (P99) | <500μs local-engine target (excludes cross-AZ/WAN effects) |
| Throughput (TPS) | 50K+ |
| API Latency (P99) | <100ms |
| Database Latency | <50ms |
| Fabric Block Time | <5s |
| Uptime | 99.99% target |
| Double-spend Rate | Illustrative low-rate target |

---

**Document Version**: 2.0
**Last Updated**: May 2025
**Project Type**: Personal study architecture
