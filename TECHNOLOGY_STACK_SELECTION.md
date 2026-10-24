# Digital Dirham CBDC v2 - Technology Stack Selection & Justification

> **Disclaimer**  
> This file is part of an independent personal study based on publicly available CBDC resources.  
> It is not an official Bank Al-Maghrib document and does not rely on confidential institutional information.

## Executive Summary

The Digital Dirham v2 technology stack balances **security**, **scalability**, **CBDC-specific requirements**, and **Moroccan deployment constraints**. This document provides detailed comparative analysis and final recommendations for each major component.

---

# PART 1: BACKEND LANGUAGE SELECTION

## Comparison: Rust vs Go vs Java

### 1.1 Overview Table

| Criterion | **Rust** | **Go** | **Java** |
|-----------|----------|--------|---------|
| **Memory Safety** | Compile-time guarantees | GC-managed (risk) | GC-managed (risk) |
| **Performance** | Extremely High (C-like) | High (Excellent) | Moderate (GC pauses) |
| **Concurrency** | Async/await + Channels | Goroutines (excellent) | Threads + Reactive |
| **Latency Predictability** | Deterministic (no GC) | ~10-100ms GC pauses | ~50-500ms GC pauses |
| **Development Speed** | Slow (strict compiler) | Fast (simple syntax) | Very Fast (verbose) |
| **Learning Curve** | Steep (ownership system) | Gentle (C-like) | Moderate (OOP) |
| **Ecosystem** | Growing (crypto strong) | Excellent (cloud-native) | Mature (enterprise) |
| **CBDC Fit** | Excellent | Good | Fair |

---

### 1.2 Detailed Analysis

#### 1.2.1 RUST - Backend Core Services

**Best for:** Settlement Layer, Monetary Layer, Consensus Coordination, Cryptography-Heavy Operations

##### Security Properties
```
✅ Memory Safety at Compile Time
   - No buffer overflows
   - No use-after-free
   - No data races (enforced by borrow checker)
   - Result: Impossible to exploit memory vulnerabilities

✅ No Garbage Collector
   - Deterministic latency (critical for T+0 settlement)
   - Predictable resource usage
   - No 50-500ms GC pauses during transaction processing
   - Result: Sub-millisecond local-path transaction finality can be achievable in controlled benchmarks

✅ Strong Type System
   - Algebraic Data Types (Sum/Product types)
   - Pattern matching for exhaustive case handling
   - Result: Impossible to forget edge cases in transaction logic

✅ Explicit Error Handling
   - Result<T, E> type forces error propagation
   - No silent failures or null pointer exceptions
   - Result: Every error path is accountable
```

##### Performance Characteristics
- **Memory overhead:** ~0-5% (RAII model)
- **CPU performance:** Within 10% of C (often faster due to better optimizations)
- **Latency (p99):** <100μs local-path benchmark target for typical operation
- **Throughput:** 1M+ operations/second on single core (with async)

##### CBDC-Specific Strengths
1. **Cryptography Libraries**
   - `arkworks-rs`: zk-SNARK proving (Groth16 optimized)
   - `ring`: Fast, audited elliptic curve operations
   - `blake3`: 200GB/s hash throughput (Merkle trees)
   - No Java/Go equivalents match these performance levels

2. **Low-Level Hardware Access**
   - Direct HSM integration (PKCS#11)
   - CPU instruction-level optimizations (AES-NI)
   - NUMA-aware memory allocation

3. **Formal Verification Readiness**
   - Strong type system enables automated proofs
   - Linear types enforce protocol invariants
   - Mutually exclusive state prevents corrupted ledger states

##### Moroccan Deployment Fit
- **Server Requirements:** Minimal (Rust compiles to single binary)
- **Operational Complexity:** Lower (no runtime, no JVM)
- **Training Curve:** Steeper (requires functional programming mindset)
- **Local Talent:** Limited but growing (Morocco has ~200-300 Rust engineers)

##### Recommended Use
```rust
// Settlement Layer (Rust)
pub struct TransactionValidator {
    blockchain_state: Arc<RwLock<LedgerState>>,
    signature_verifier: Ed25519Verifier,
}

impl TransactionValidator {
    pub async fn validate_and_settle(
        &self,
        tx: &SignedTransaction,
    ) -> Result<TransactionReceipt, SettlementError> {
        // Compiler enforces: no data races, no null pointers, no buffer overflows
        // Runtime target (local benchmark): <100μs, no GC pauses
        self.verify_signature(tx)?;
        self.check_balance(tx)?;
        self.update_ledger(tx)?;
        Ok(TransactionReceipt::new(tx.hash()))
    }
}
```

---

#### 1.2.2 GO - Distributed Services & API Gateway

**Best for:** API Gateway, Identity Service, Distribution Layer, Cross-Border Service, Kubernetes controllers

##### Security Properties
```
✅ Network Communication
   - Built-in TLS support (production-grade)
   - Fast HTTP/2 implementation
   - No buffer overflows (bounds checking)

⚠️  Garbage Collector
   - 10-100ms GC pauses (acceptable for non-settlement services)
   - Concurrent GC reduces pause impact
   - Tunable GC frequency (tradeoff: memory vs latency)

✅ Simplicity
   - Fewer lines of code = fewer bugs
   - No null pointer exceptions (nil tracking)
   - Explicit error handling (but not as strict as Rust)

✅ Goroutines
   - 1M+ concurrent connections per node
   - Perfect for API gateway handling retail transactions
```

##### Performance Characteristics
- **Memory overhead:** 5-10% (GC overhead)
- **CPU performance:** 5-20% slower than Rust (GC + lack of optimizations)
- **Latency (p99):** 10-100ms (GC pause dependent)
- **Throughput:** 100K-500K requests/second per instance

##### CBDC-Specific Strengths
1. **Cloud-Native Excellence**
   - Built for microservices, Kubernetes, Docker
   - Fast container startup (seconds vs minutes for JVM)
   - Minimal resource footprint

2. **Concurrency for I/O Bound Services**
   - API Gateway needs to handle 1000s concurrent connections
   - Goroutines are perfect for this (not compute-intensive)
   - Each connection doesn't need separate OS thread

3. **DevOps Friendly**
   - Single binary deployment (no dependencies)
   - Excellent observability (pprof, traces)
   - Fast iteration (compile time <5 seconds)

##### Moroccan Deployment Fit
- **Server Requirements:** Very low (minimal RAM, CPU)
- **Operational Complexity:** Low (no JVM, no Rust compiler)
- **Training Curve:** Gentle (C-like syntax, Pythonic error handling)
- **Local Talent:** Good (Go is popular in Moroccan startups)

##### Recommended Use
```go
// API Gateway (Go)
func (gw *APIGateway) HandleTransfer(w http.ResponseWriter, r *http.Request) {
    // Goroutines handle 1000s concurrent requests efficiently
    // GC pauses don't matter here (not latency-critical)
    
    var req TransferRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    // Forward to Rust settlement service (gRPC)
    receipt, err := gw.settlementClient.Transfer(context.Background(), &req)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(receipt)
}
```

---

#### 1.2.3 JAVA - Enterprise Services (NOT Recommended as Primary)

**Could be used for:** Legacy bank integration, optional backup for compliance services

##### Why NOT for Digital Dirham Core
```
❌ CBDC Settlement Layer
   - 50-500ms GC pauses unacceptable for T+0 settlement
   - Cryptography libraries slower than Rust
   - Memory overhead 20-30% (JVM startup, GC overhead)

❌ Offline Payment Validation
   - Cannot run on SE (Secure Element) - SE cannot run JVM
   - Cannot run on feature phones efficiently
   - Cannot run in ultra-low-power environments (IoT)

❌ Operational Overhead in Morocco
   - JVM requires 500MB-2GB per instance
   - Slow startup (10-30 seconds)
   - Complex operations (heap size tuning, GC tuning)
   - Requires experienced DevOps engineers

✅ Optional: Enterprise Legacy Integration
   - If existing Moroccan banks mandate Java
   - Could run as separate adapter layer
   - Use async/reactive frameworks (Spring WebFlux) to minimize GC impact
```

##### If Used: Constraints
- **Only for:** Integration adapters, not settlement
- **Framework:** Project Loom (virtual threads) + Spring Boot Reactive
- **GC Config:** G1GC with MaxGCPauseMillis=10ms (aggressive)
- **Monitoring:** Constant GC pause monitoring (must be <50ms p99)

---

### 1.3 FINAL RECOMMENDATION: Polyglot Stack

```
┌─────────────────────────────────────────────────────────┐
│         DIGITAL DIRHAM RECOMMENDED STACK                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  80% RUST (Core, Security-Critical)                   │
│  ├─ Settlement Layer (Layer 2)                        │
│  ├─ Monetary Layer (Layer 1)                          │
│  ├─ Cryptography (Layer 4)                            │
│  ├─ Consensus Coordination                            │
│  ├─ Offline SE Communication                          │
│  └─ Mobile/Wallet Backend SDK                         │
│                                                         │
│  15% GO (Operational, I/O Bound)                      │
│  ├─ API Gateway                                       │
│  ├─ Identity Service (Layer 5)                        │
│  ├─ Distribution Layer (Layer 6)                      │
│  ├─ Kubernetes Controllers                            │
│  └─ Operational Dashboards                            │
│                                                         │
│  5% OTHER                                              │
│  ├─ Frontend: TypeScript/React (Web Wallet)           │
│  ├─ Mobile: React Native (iOS/Android)                │
│  ├─ Hyperledger Fabric: Go (chaincode)                │
│  └─ Analytics: Python (Data Science)                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 1.4 Language Selection Decision Matrix

| Service | Rust | Go | Java | Reason |
|---------|------|-----|------|--------|
| **Settlement** | ✅✅ | ❌ | ❌ | Needs low-latency local-path execution, no GC |
| **Consensus Coord** | ✅✅ | ✅ | ❌ | Rust for cryptography, Go ok for orchestration |
| **API Gateway** | ❌ | ✅✅ | ✅ | I/O bound, goroutines ideal, Go simpler |
| **Identity Verify** | ✅ | ✅✅ | ✅ | Rust for biometric processing, Go for API routing |
| **Compliance AML** | ✅ | ✅✅ | ❌ | Needs real-time ML, Go good for streaming |
| **Blockchain Node** | ✅ | ✅ (Fabric: Go) | ❌ | Fabric is built in Go |
| **Wallet Backend** | ✅✅ | ❌ | ❌ | Needs efficiency for offline SE support |
| **Cross-Border** | ✅ | ✅✅ | ✅ | Network protocol, any language works; Go = ops simplicity |
| **Operational Tools** | ❌ | ✅✅ | ✅ | Speed of iteration matters; Go fastest |

---

---

# PART 2: DATABASE SELECTION

## Comparison: PostgreSQL vs CockroachDB

### 2.1 Overview Table

| Criterion | **PostgreSQL** | **CockroachDB** |
|-----------|---|---|
| **Consistency Model** | ACID (single region) | ACID (distributed) |
| **Replication** | Synchronous (standby) | Automatic (3+ zones) |
| **Failover Time** | Manual or 1-2 minutes | <3 seconds automatic |
| **Scalability** | Vertical (single primary) | Horizontal (unlimited) |
| **Sharding** | Manual (complex) | Automatic (transparent) |
| **Operational Complexity** | Low-Moderate | Moderate-High |
| **Cost** | Free (OSS) | Free (OSS) + Enterprise |
| **CBDC Fit** | Excellent (Primary) | Good (Optional Backup) |
| **Morocco Deployment** | Native (familiar) | Learning curve |

---

### 2.2 Detailed Analysis

#### 2.2.1 POSTGRESQL - Primary Operational Database

**Primary use:** Main transactional database, derived indices, audit trail

##### PostgreSQL Strengths for CBDC

```
✅ ACID Guarantees (Gold Standard)
   - Atomicity: All-or-nothing transactions
   - Consistency: Data never corrupted
   - Isolation: Serializable isolation by default
   - Durability: WAL (Write-Ahead Log) guarantees

✅ Row-Level Security (RLS)
   - Table policies control who sees what data
   - Encrypt KYC data at rest (pgcrypto)
   - Decrypt only for authorized queries (warrants)
   - Example: User A cannot SELECT User B's transactions

✅ Cryptography Built-In (pgcrypto)
   - AES-256-GCM encryption for PII columns
   - Transparent at query level
   - Keys stored in Vault (not in database)
   - Performance: <1% overhead

✅ Append-Only Tables (Temporal Tables)
   - CREATE TABLE ... WITH (fillfactor=10)
   - Log entries NEVER updated, only inserted
   - Audit compliance: Perfect for regulatory requirements
   - Immutability: Prevents accidental deletion

✅ Full-Text Search (Built-In)
   - tsvector columns for transaction searching
   - No separate Elasticsearch needed (though used for scaling)
   - Fast, indexed search: <10ms for 100M rows

✅ JSON/JSONB Support
   - Flexible schema for CBDC metadata
   - JSONB = binary JSON, indexed
   - Query: SELECT data->>'compliance_flags' FROM transactions
   - Combine relational + document power

✅ Materialized Views
   - Pre-computed aggregations (wallet balances, daily volumes)
   - Refresh on schedule or on-demand
   - Query speed: 10-100x faster than computing live

✅ Scaling via Streaming Replication
   - Primary → Standby replication (synchronous)
   - Read replicas for analytics (asynchronous)
   - Failover: Seconds to minutes
   - Good enough for 3-AZ setup

✅ Observability
   - pg_stat_statements: Every query analyzed
   - Auto EXPLAIN: Show slow query plans
   - TOAST tables: Handle 1GB+ columns (if needed)
```

##### PostgreSQL Architecture (for Digital Dirham)

```
┌──────────────────────────────────────────────────────┐
│           PostgreSQL Cluster (3 AZ)                 │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Primary (AZ1 - Rabat)                             │
│  ├─ Main transactional load                        │
│  ├─ Receives writes                                │
│  ├─ Synchronous WAL to Standby (AZ2)              │
│  ├─ Schema:                                        │
│  │  ├─ wallets (indexed by pk, kyc_tier)          │
│  │  ├─ transactions (indexed by timestamp)        │
│  │  ├─ kyc_data (encrypted, RLS protected)        │
│  │  ├─ audit_log (append-only)                    │
│  │  └─ sanctions_cache (materialized view)        │
│  └─ Size: ~100-500GB (grows ~5-10GB/month)       │
│                                                      │
│  Standby (AZ2 - Casablanca)                        │
│  ├─ Hot standby (can accept reads)                │
│  ├─ Receives sync WAL from Primary                │
│  ├─ Failover: 30 seconds manual, 3min auto       │
│  └─ Not primary (read-only for analytics)        │
│                                                      │
│  Read Replica (AZ3 - Tangier)                      │
│  ├─ Async replication (slight lag)                │
│  ├─ Accepts read-heavy analytics queries         │
│  ├─ Does NOT receive settlement writes            │
│  └─ Can afford 10-30 second replication lag      │
│                                                      │
└──────────────────────────────────────────────────────┘
```

##### Schema Example

```sql
-- WALLETS table (Primary Key indexed)
CREATE TABLE wallets (
    wallet_id UUID PRIMARY KEY,
    user_cnie_hash BYTEA UNIQUE,           -- De-duplication
    kyc_tier SMALLINT CHECK (kyc_tier IN (1,2,3)),
    balance_fils BIGINT DEFAULT 0,          -- In fils (1 MAD = 100 fils)
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    CONSTRAINT balance_non_negative CHECK (balance_fils >= 0)
);

-- TRANSACTIONS table (Immutable Append-Only)
CREATE TABLE transactions (
    tx_id UUID PRIMARY KEY,
    fabric_tx_id CHAR(64) UNIQUE,           -- Link to Fabric ledger
    sender_wallet_id UUID NOT NULL REFERENCES wallets(wallet_id),
    receiver_wallet_id UUID NOT NULL REFERENCES wallets(wallet_id),
    amount_fils BIGINT NOT NULL CHECK (amount_fils > 0),
    status VARCHAR(20) DEFAULT 'PENDING',   -- PENDING, CONFIRMED, REJECTED, AML_HOLD
    created_at TIMESTAMPTZ DEFAULT NOW(),
    CONSTRAINT finality_immutable CHECK (status != 'PENDING' OR created_at > NOW() - INTERVAL '5 seconds')
);
CREATE INDEX idx_transactions_sender ON transactions(sender_wallet_id, created_at DESC);
CREATE INDEX idx_transactions_timestamp ON transactions(created_at DESC);

-- KYC_DATA table (Encrypted, Row-Level Security)
CREATE TABLE kyc_data (
    kyc_id UUID PRIMARY KEY,
    wallet_id UUID NOT NULL UNIQUE REFERENCES wallets(wallet_id),
    full_name_encrypted BYTEA,             -- Encrypted with Vault key
    dob_encrypted BYTEA,
    address_encrypted BYTEA,
    cnie_number_hash BYTEA,                -- Hash only, never plaintext
    created_at TIMESTAMPTZ DEFAULT NOW(),
    verified_at TIMESTAMPTZ
);
-- Row-Level Security: Only owner's bank can access
ALTER TABLE kyc_data ENABLE ROW LEVEL SECURITY;
CREATE POLICY kyc_access ON kyc_data
    FOR SELECT
    USING (current_user_bank_id() = wallet_id::bank_id);  -- Custom function

-- AUDIT_LOG table (Append-Only, Immutable)
CREATE TABLE audit_log (
    log_id BIGSERIAL PRIMARY KEY,           -- Monotonically increasing
    event_type VARCHAR(50) NOT NULL,
    actor_id UUID,
    resource_id UUID,
    action_details JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    CHECK (created_at >= CURRENT_TIMESTAMP - INTERVAL '1 second')  -- Prevent backdating
);
CREATE INDEX idx_audit_timestamp ON audit_log(created_at DESC);
```

##### Performance Expectations

```
PostgreSQL + Rust Backend Performance:

Transaction Settlement:
- Initial SELECT (wallet check): ~5-10ms
- UPDATE (balance): ~2-5ms
- INSERT (audit): ~1-2ms
- Commit (WAL to disk): ~1-2ms
- Total: ~10-20ms per transaction
- With connection pooling + batching: 1000+ TPS per node

Query Performance:
- Simple SELECT by PK: <1ms
- JOIN on indexed column: <5ms
- Full-text search (100M rows): <50ms
- Materialized view aggregation: <100ms

Backup & Recovery:
- pg_basebackup: ~5-10GB/hour
- Point-in-time recovery: <5 minutes
- Full recovery from backup: ~1-2 hours
```

---

#### 2.2.2 COCKROACHDB - Distributed Alternative (Optional)

**Optional use:** If Morocco needs distributed, geo-redundant database (future consideration)

##### CockroachDB Strengths

```
✅ Distributed ACID
   - Multi-region consistency without complexity
   - Automatic replication (default: 3 copies)
   - No single point of failure

✅ Horizontal Scalability
   - Add nodes, data automatically re-balanced
   - No manual sharding
   - Transparent to application

✅ Resilience
   - Survives losing entire data center
   - <3 second failover
   - Zero data loss (synchronous replication)

✅ PostgreSQL Compatible
   - Use PostgreSQL drivers
   - Familiar SQL syntax
   - Can migrate from PostgreSQL relatively easily
```

##### CockroachDB Weaknesses (for Digital Dirham)

```
❌ Operational Complexity
   - Distributed systems add debugging difficulty
   - Network partition scenarios require deep understanding
   - More knobs to tune (replication factor, lease preference, etc.)

❌ Cost
   - No free option for production
   - Enterprise license: ~$100K-500K/year
   - Support: Separate cost

❌ Not Necessary for Morocco
   - BAM already has disaster recovery budget
   - 3-zone PostgreSQL with manual failover is simpler
   - If high availability needed: add another PostgreSQL standby

❌ Performance Overhead
   - Distributed consensus adds latency
   - Write latency: 50-100ms (vs 20ms for single-region PostgreSQL)
   - Replication overhead: ~30% CPU increase

❌ Fewer Encryption Features
   - No built-in column encryption (like pgcrypto)
   - RLS implementation is less mature
```

##### When to Use CockroachDB (Not Now, But Future)

```
Scenario: Morocco opens branch in Egypt or Tunisia
- Need distributed CBDC settlement across countries
- FATF regulations require 2+ jurisdictions for resilience
- Then: CockroachDB with replicas in (Rabat, Cairo, Tunis)

Otherwise: PostgreSQL is superior choice now
```

---

### 2.3 FINAL DATABASE RECOMMENDATION

```
PRIMARY STACK:
├─ Canonical Ledger: Hyperledger Fabric
│  └─ Immutable blockchain records
│
├─ Operational DB: PostgreSQL (3-AZ)
│  ├─ AZ1 Primary (Rabat)
│  ├─ AZ2 Standby (Casablanca) - Synchronous replication
│  └─ AZ3 Read Replica (Tangier) - Async replication
│  └─ Use: Wallets, transactions, KYC, audit trail
│
├─ Hot Cache: Redis
│  ├─ Use: Session state, rate limits, temporary data
│  └─ Eviction: LRU, TTL 24 hours
│
├─ Time-Series: TimescaleDB (PostgreSQL extension)
│  ├─ Use: Metrics, supply volumes, FX rates
│  └─ Retention: 10 years (regulatory)
│
├─ Full-Text Search: Elasticsearch
│  ├─ Use: Transaction search, merchant directory
│  └─ Replicas: 3 shards x 2 replicas
│
└─ Data Warehouse: S3 + Parquet (Cold Archive)
   └─ Daily export for compliance reporting
```

---

---

# PART 3: LEDGER TECHNOLOGY SELECTION

## Comparison: Hyperledger Fabric vs Centralized Ledger

### 3.1 Overview Table

| Criterion | **Hyperledger Fabric** | **Centralized Ledger (Rust)** |
|-----------|---|---|
| **Consensus Model** | BFT (Byzantine Fault Tolerant) | Single Orderer (Fast) |
| **Fault Tolerance** | f < n/3 (survive 2/7 nodes down) | Manual failover |
| **Latency** | 1-3 seconds | 100-500 microseconds |
| **Throughput** | 5,000-10,000 TPS | 50,000-100,000+ TPS |
| **Cryptographic Proof** | Native (tamper-evident) | Requires separate audit layer |
| **Regulatory Compliance** | BIS approved (deployed in eNaira) | Less proven in CBDC context |
| **Operational Complexity** | High (7+ nodes, consensus tuning) | Low (single binary) |
| **CBDC Fit** | Excellent (Audit Layer) | Excellent (Core Settlement) |
| **Morocco Deployment** | Good (proven in Africa) | Excellent (simplicity) |

---

### 3.2 RECOMMENDED HYBRID ARCHITECTURE

```
┌────────────────────────────────────────────────────────────┐
│     DIGITAL DIRHAM: TWO-LEDGER ARCHITECTURE               │
└────────────────────────────────────────────────────────────┘

         ┌─────────────────────────────────────┐
         │   Centralized Rust Settlement       │
         │   (PRIMARY LEDGER - Core)           │
         │   ├─ Sub-ms local processing target │
         │   ├─ 50,000+ TPS throughput        │
         │   ├─ State: UTXO + Account Model   │
         │   ├─ Merkle tree commitments       │
         │   ├─ PostgreSQL backing store      │
         │   └─ Quorum-signed blocks (5-of-7) │
         └──────────────┬──────────────────────┘
                        │
                        ↓ Async write
         ┌──────────────────────────────────────────┐
         │  Hyperledger Fabric Audit Layer          │
         │  (SECONDARY LEDGER - Verification)       │
         │  ├─ 1-3 second latency                  │
         │  ├─ 5,000-10,000 TPS                    │
         │  ├─ Cryptographic tamper-proofs         │
         │  ├─ BFT consensus (7 nodes, 5-of-7 quorum) │
         │  ├─ Independent verification            │
         │  └─ Regulatory audit compliance         │
         └─────────────────────────────────────────┘

PURPOSE:
- Rust = PERFORMANCE (T+0 settlement)
- Fabric = ASSURANCE (tamper-proof audit trail)

GUARANTEE:
If Rust ledger is compromised:
  → Fabric ledger still has immutable record
  → Auditors can reconstruct truth from Fabric
  → Fraud is detectable retroactively
```

---

### 3.3 Detailed Justification

#### 3.3.1 WHY NOT Fabric-Only?

```
❌ LATENCY INCOMPATIBLE WITH CBDC REQUIREMENTS
   - Fabric consensus: 1-3 seconds
   - Digital Dirham requirement: < 500ms
   - Offline payments need sync to finalize in <24 hours: OK with Fabric
   - Online payments need T+0 finality: NOT OK with 3-second Fabric

❌ THROUGHPUT INSUFFICIENT FOR MOROCCO
   - Fabric: 5,000-10,000 TPS
   - Peak load during salary season: 50,000+ TPS
   - Fabric cannot handle peak (queue builds, transactions fail)

❌ OPERATIONAL COMPLEXITY
   - 7+ BFT nodes (expensive infrastructure)
   - Requires distributed systems expertise (rare in Morocco)
   - Consensus tuning (block time, batch size) for production
   - More things to fail, more things to monitor

❌ NO ADVANTAGE OVER CENTRALIZED FOR CBDC
   - CBDCs are inherently centralized (single issuer = BAM)
   - Private blockchain doesn't need decentralization
   - Byzantine fault tolerance is overkill when you control all nodes
   - BAM can just run 3-5 hot failover nodes for availability
```

#### 3.3.2 WHY NOT Centralized-Only?

```
❌ REGULATORY CONCERNS
   - "How do we know BAM isn't lying about transactions?"
   - Centralized system is trust-based (not cryptographically-verifiable)
   - FATF auditors may question integrity

❌ SECURITY ASSUMPTION
   - If BAM's database is compromised, criminals could:
     • Modify transaction history
     • Delete evidence
     • Forge new transactions
   - No independent way to detect tampering

❌ STAKEHOLDER CONFIDENCE
   - Commercial banks may distrust sole reliance on BAM systems
   - Public may fear surveillance (no external verification)
   - International partners (mBridge countries) may want independent proof

✅ MITIGATION: Add Fabric Audit Layer (See Below)
```

#### 3.3.3 OPTIMAL SOLUTION: Hybrid Two-Ledger

```
┌──────────────────────────────────────────────────────────┐
│   WHY HYBRID IS BEST COMPROMISE                         │
└──────────────────────────────────────────────────────────┘

1. PERFORMANCE (✅ Rust Ledger)
   - Centralized Rust engine achieves <500ms finality
   - Sufficient for offline sync reconciliation (T+1)
   - Retail payment experience is instant (wallet confirms immediately)

2. ASSURANCE (✅ Fabric Audit Layer)
   - Every transaction ALSO written to Fabric (after finality)
   - Fabric lag: 1-3 seconds (OK because offline payments already waited)
   - Creates independent, tamper-proof audit trail
   - Auditors can verify Rust ledger against Fabric

3. SECURITY (✅ Dual Protection)
   - Attacking Rust ledger requires compromising Rust + Fabric (impossible)
   - Attacking Fabric requires Byzantine fault (99.999% reliable)
   - Defense in depth: breach of one ledger doesn't compromise other

4. REGULATORY SATISFACTION
   - BAM: "We run auditable Fabric blockchain"
   - FATF auditors: Can verify transactions independently
   - Public: "Central bank cannot secretly modify transactions"
   - Morocco's parliament: "We can inspect immutable Fabric ledger"

5. OPERATIONAL SIMPLICITY
   - Rust handles performance-critical path (95% of resources)
   - Fabric is fire-and-forget (async write, not blocking)
   - Fewer nodes (7 Fabric nodes) than if Fabric did everything
   - Clear separation: Performance vs. Verification

6. UPGRADE PATH
   - Start with Rust + PostgreSQL
   - Add Fabric audit layer in Phase 2 (once operational)
   - Can deploy Fabric independently without modifying Rust code
   - If Fabric fails, settlement continues (not on critical path)
```

---

### 3.4 Implementation Details: Hybrid Architecture

#### 3.4.1 Centralized Rust Settlement Ledger

```rust
// Core Settlement Engine (Rust)
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct SettlementBlock {
    pub height: u64,
    pub timestamp: u64,
    pub transactions: Vec<SignedTransaction>,
    pub merkle_root: [u8; 32],
    pub previous_hash: [u8; 32],
    pub validator_signatures: Vec<(ValidatorId, Signature)>,  // 5-of-7 quorum
}

pub struct SettlementLedger {
    blocks: Vec<SettlementBlock>,
    state: LedgerState,  // Current balances
    database: Arc<PostgresPool>,
}

impl SettlementLedger {
    pub async fn commit_block(&mut self, block: SettlementBlock) -> Result<(), Error> {
        // 1. Validate signatures (5+ of 7 nodes must sign)
        self.validate_quorum_signatures(&block)?;
        
        // 2. Verify Merkle root (all transactions accounted for)
        self.verify_merkle_root(&block)?;
        
        // 3. Update state (apply all transactions atomically)
        self.apply_transactions(&block)?;
        
        // 4. Commit to PostgreSQL WAL (durable)
        self.database.query(
            "INSERT INTO settlement_blocks (height, data, merkle_root) VALUES ($1, $2, $3)",
            &[&block.height, &block.encode(), &block.merkle_root],
        ).await?;
        
        // 5. Async: Send to Fabric audit layer (non-blocking)
        self.publish_to_fabric_async(&block);  // Fire-and-forget
        
        // 6. Return success to clients immediately
        Ok(())
    }
    
    pub async fn publish_to_fabric_async(&self, block: &SettlementBlock) {
        // Publish to Kafka topic "settlement-blocks-to-audit"
        // Fabric worker consumes asynchronously (1-3 second lag acceptable)
        self.kafka_producer.send("settlement-blocks", block).await;
    }
}
```

**Properties:**
- Latency: sub-millisecond is a local-engine benchmark target; end-to-end latency is topology-dependent and higher with cross-AZ sync/WAN
- Throughput: 50,000+ TPS (batched blocks, not individual transactions)
- Durability: WAL-based (synchronous to disk)
- Availability: Quorum of 5-of-7 validators (any 2 nodes can fail)
- Forensics: Merkle tree enables selective auditing

---

#### 3.4.2 Fabric Audit Layer (Independent Verification)

```go
// Hyperledger Fabric Chaincode (Go)
package main

import (
    "encoding/json"
    "github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type SettlementAudit struct {
    contractapi.Contract
}

type AuditRecord struct {
    Height       int64  `json:"height"`
    MerkleRoot   string `json:"merkle_root"`
    BlockHash    string `json:"block_hash"`
    ValidatorSigs []string `json:"validator_signatures"`
    Timestamp    int64  `json:"timestamp"`
}

// RecordSettlementBlock is called by the audit ingestion daemon
// It receives settlement blocks from Rust ledger and records immutably on Fabric
func (s *SettlementAudit) RecordSettlementBlock(
    ctx contractapi.TransactionContextInterface,
    blockHeight string,
    merkleRoot string,
    blockHash string,
    validatorSigs string,
) error {
    
    record := AuditRecord{
        Height:       parseInt(blockHeight),
        MerkleRoot:   merkleRoot,
        BlockHash:    blockHash,
        ValidatorSigs: parseJsonArray(validatorSigs),
        Timestamp:    ctx.GetStub().GetTxTimestamp().Seconds,
    }
    
    // Immutable write to Fabric ledger
    recordBytes, _ := json.Marshal(record)
    return ctx.GetStub().PutState("audit_"+blockHeight, recordBytes)
}

// AuditVerification: Public query to verify settlement block integrity
func (s *SettlementAudit) VerifySettlementBlock(
    ctx contractapi.TransactionContextInterface,
    blockHeight string,
) (string, error) {
    
    auditRecord, err := ctx.GetStub().GetState("audit_" + blockHeight)
    if err != nil {
        return "", err
    }
    
    // Return entire audit record (cryptographically proof-of-presence)
    return string(auditRecord), nil
}
```

**Properties:**
- 7 independent Fabric nodes verify independently
- BFT consensus (can tolerate 2 malicious/failed nodes)
- Every block immutably recorded on ledger
- Auditors can query: "Verify transaction TX-123" → Get cryptographic proof
- No way to delete or modify (would require 7 nodes to conspire)

---

#### 3.4.3 Reconciliation Protocol (Fabric ← Rust)

```
TIME    EVENT                           RUST LEDGER         FABRIC AUDIT
────────────────────────────────────────────────────────────────────────
T+0ms   User submits transaction        [PENDING]
        Rust validates signature         [VALIDATING]
        Consensus reached (5/7 sign)    [COMMITTED] ✓       [receiving]
        PostgreSQL persists              ✓ (durable)        
        
T+100ms Wallet shows receipt            ✓ Confirmed          [processing]
        User sees transaction            (T+0 finality)
        
T+1s    Kafka publishes block event     (async)             [Fabric validates]
                                                            BFT consensus:
                                                            Node 1-7 endorse
        
T+3s    Fabric commits block            -                   [COMMITTED] ✓
        Immutable audit record created   -                   (on ledger)

AUDIT VERIFICATION:
T+2025-05-31: Auditor queries
    GET Fabric: "Verify TX-123"
    Fabric returns: {
        "height": 12345,
        "merkle_root": "0xabc123...",
        "validator_sigs": [5+ signatures],
        "timestamp": 2025-05-31T12:34:56Z
    }
    Auditor cryptographically verifies: All 5+ signatures are valid
    Auditor confirms: This transaction is on immutable ledger
```

---

### 3.5 FINAL TECHNOLOGY STACK DECISION

```
┌─────────────────────────────────────────────────────────┐
│    DIGITAL DIRHAM v2 - FINAL TECHNOLOGY SELECTION      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  PRIMARY LEDGER: Centralized (Rust + PostgreSQL)      │
│  ├─ Core settlement engine (in Rust)                  │
│  ├─ State machine (UTXO model)                        │
│  ├─ Persistent storage (PostgreSQL + WAL)             │
│  ├─ Quorum signing (5-of-7 validators)                │
│  └─ SLA target: <500μs local processing, 50K+ TPS    │
│                                                         │
│  AUDIT LAYER: Hyperledger Fabric (Go Chaincode)       │
│  ├─ Independent verification                          │
│  ├─ Tamper-proof record                               │
│  ├─ BFT consensus (7 nodes, 5-of-7 quorum)           │
│  ├─ Async write (1-3s lag acceptable)                │
│  └─ Regulatory compliance                             │
│                                                         │
│  OPERATIONAL DATABASE: PostgreSQL                     │
│  ├─ Primary (AZ1 - Rabat)                             │
│  ├─ Standby (AZ2 - Casablanca)                        │
│  ├─ Read Replica (AZ3 - Tangier)                      │
│  ├─ Encryption: pgcrypto (KYC data)                   │
│  └─ RLS: Row-level security policies                  │
│                                                         │
│  HOT CACHE: Redis                                     │
│  ├─ Session state                                     │
│  ├─ Rate limiters                                     │
│  ├─ ZKP circuit cache                                 │
│  └─ TTL: 24 hours                                     │
│                                                         │
│  TIME-SERIES: TimescaleDB (PostgreSQL extension)      │
│  ├─ Supply metrics                                    │
│  ├─ Velocity analysis                                 │
│  ├─ Retention: 10 years                               │
│  └─ For central bank dashboards                       │
│                                                         │
│  SEARCH: Elasticsearch                                │
│  ├─ Transaction search                                │
│  ├─ Merchant lookup                                   │
│  ├─ Audit log search                                  │
│  └─ 3 shards x 2 replicas                             │
│                                                         │
│  ARCHIVE: S3 + Parquet (Cold Storage)                │
│  ├─ Compliance export                                 │
│  ├─ Long-term audit trail                             │
│  └─ Retention: 10 years (FATF requirement)            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

# PART 4: CROSS-LAYER TECHNOLOGY MAPPING

## 4.1 Technology Assignment by Architecture Layer

| Layer | Service | Primary Language | Database | Ledger |
|-------|---------|------------------|----------|--------|
| **1: Monetary** | Issuance Engine | Rust | PostgreSQL | Fabric |
| **1: Monetary** | Policy Engine | Rust | Git Repo + TimescaleDB | - |
| **1: Monetary** | Supply Monitor | Go | TimescaleDB | - |
| **2: Settlement** | Tx Validator | Rust | PostgreSQL | Rust Ledger |
| **2: Settlement** | Consensus | Rust | Rust Ledger | Rust Ledger |
| **2: Settlement** | Finality Service | Rust | PostgreSQL | Fabric (async) |
| **3: Compliance** | Sanctions | Go | Bloom Filter Cache | - |
| **3: Compliance** | AML Detector | Go | PostgreSQL | Kafka |
| **3: Compliance** | SAR Generator | Go | PostgreSQL | - |
| **4: Privacy** | Blind Signature | Rust | Redis | - |
| **4: Privacy** | ZKP Engine | Rust | Redis | - |
| **5: Identity** | CNIE Verify | Rust | PostgreSQL | - |
| **5: Identity** | KYC Tier | Go | PostgreSQL | - |
| **6: Distribution** | API Gateway | Go | Redis | - |
| **6: Distribution** | Wallet API | Go | PostgreSQL | - |
| **7: Offline** | Sync Engine | Go | PostgreSQL | Fabric |
| **7: Offline** | Double-Spend Detect | Rust | Bloom Filter | Fabric |
| **8: Cross-Border** | mBridge Conn | Go | PostgreSQL | Rust Ledger |
| **8: Cross-Border** | CapCtrl Engine | Go | PostgreSQL | - |

---

## 4.2 Data Flow Through Technology Stack

```
USER SUBMITS TRANSACTION
    ↓
Go API Gateway (REST endpoint)
    ↓
Go Session Manager (Redis lookup)
    ↓
Go Identity Service (PostgreSQL KYC check)
    ↓
Rust Settlement Validator (Memory-resident)
    • Verify Ed25519 signature
    • Check balance (PostgreSQL query)
    • Validate against policy (OPA rules)
    ↓
Rust Consensus Engine (Local BFT)
    • Batch transactions
    • Collect 5-of-7 quorum signatures
    • Create Merkle tree
    ↓
Rust Settlement Commit (PostgreSQL)
    • ATOMIC INSERT to settlement_blocks table
    • Synchronous WAL flush (durable)
    ↓
PostgreSQL + Rust Ledger
    • Primary commitment
    • Sub-millisecond local processing target
    ↓
Generate Finality Receipt (Rust)
    • Sign by quorum
    • Include Merkle proof
    ↓
Return to Client (Go API)
    • Send JSON receipt to wallet app
    ↓
Async: Publish to Kafka (Fire-and-forget)
    ↓
Fabric Audit Layer (Consume Kafka)
    • Validate settlement block
    • Append to Fabric ledger (1-3 second lag)
    • BFT consensus on audit record
    ↓
Immutable Audit Trail (Fabric Ledger)
    • Available for regulatory audits
    • Tamper-proof verification
```

---

## 4.3 Why This Combination Works for Morocco

### Security
- ✅ **Rust** = Memory-safe settlement (no buffer overflows, no use-after-free)
- ✅ **PostgreSQL** = ACID transactions (all-or-nothing, no partial updates)
- ✅ **Fabric** = Independent verification layer (tampering is externally detectable)
- ✅ **Cryptography** = Ed25519, Groth16 zk-SNARKs, AES-256-GCM

### Scalability
- ✅ **Rust Settlement** = 50,000+ TPS (single node, with batching)
- ✅ **PostgreSQL Replicas** = 100K+ concurrent read queries
- ✅ **Go API Gateway** = 1M+ concurrent connections (goroutines)
- ✅ **Kubernetes** = Auto-scale to 100+ pods based on load

### CBDC Requirements
- ✅ **Sub-millisecond local-path settlement target** (Rust latency profile)
- ✅ **High throughput** (Batching + async Fabric)
- ✅ **Offline support** (Rust's efficiency enables SE-friendly design)
- ✅ **Privacy** (Rust cryptography, ZKP proving)
- ✅ **Compliance** (Go ML engines, PostgreSQL audit trail, Fabric independent verification)

### Moroccan Deployment
- ✅ **Infrastructure** = Minimal (Rust binaries run on low-spec servers)
- ✅ **Skill set** = Go known in Moroccan tech community
- ✅ **Operational simplicity** = Go for ops, Rust for core; clear separation
- ✅ **Disaster recovery** = PostgreSQL failover is standard and operationally mature
- ✅ **Cost** = OSS technologies (PostgreSQL, Hyperledger Fabric, Rust, Go are free)

---

# PART 5: DEPLOYMENT & OPERATIONAL CONSIDERATIONS

## 5.1 Infrastructure Requirements

```
┌────────────────────────────────────────────────────────┐
│        HARDWARE SPECIFICATIONS (3-ZONE SETUP)        │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ZONE 1: RABAT (Primary)                              │
│  ├─ Rust Settlement Service                           │
│  │  └─ 16 cores, 64GB RAM, NVMe SSD 1TB              │
│  ├─ PostgreSQL Primary                                │
│  │  └─ 32 cores, 256GB RAM, NVMe SSD 4TB             │
│  ├─ Fabric Peer (Operator)                           │
│  │  └─ 8 cores, 32GB RAM, SSD 500GB                   │
│  └─ Total: ~16TB storage, ~350GB RAM                 │
│                                                        │
│  ZONE 2: CASABLANCA (Standby)                         │
│  ├─ PostgreSQL Standby (hot)                          │
│  │  └─ 32 cores, 256GB RAM, NVMe SSD 4TB             │
│  ├─ Fabric Peer (Backup)                             │
│  │  └─ 8 cores, 32GB RAM, SSD 500GB                   │
│  ├─ Elasticsearch cluster                             │
│  │  └─ 3x nodes (8 cores each, 64GB RAM)              │
│  └─ Total: ~12TB storage, ~288GB RAM                 │
│                                                        │
│  ZONE 3: TANGIER (Analytics)                          │
│  ├─ PostgreSQL Read Replica                           │
│  │  └─ 16 cores, 128GB RAM, SSD 2TB                   │
│  ├─ Fabric Peer (Independent)                        │
│  │  └─ 8 cores, 32GB RAM, SSD 500GB                   │
│  ├─ Redis Cache                                       │
│  │  └─ 8 cores, 96GB RAM, SSD 100GB                   │
│  ├─ TimescaleDB                                       │
│  │  └─ 16 cores, 128GB RAM, SSD 2TB                   │
│  └─ Total: ~10TB storage, ~288GB RAM                 │
│                                                        │
│  INTERNATIONAL (mBridge Participation)               │
│  ├─ Fabric Peer nodes (Cloud)                        │
│  │  └─ 4 nodes across EU, Asia (for cross-border)   │
│  └─ Total: 4x (8 core, 32GB RAM each)               │
│                                                        │
│  TOTAL INFRASTRUCTURE:                               │
│  └─ ~60TB storage, ~2TB RAM, ~400 cores             │
│  └─ Estimated cost: $500K-1M/year (colo + managed)  │
│                                                        │
└────────────────────────────────────────────────────────┘
```

## 5.2 Performance Targets

```
SETTLEMENT LAYER (Rust)
├─ Latency targets (local path): P50 <100μs, P99 <500μs
├─ Throughput: 50,000 TPS sustained
├─ Error rate: <0.001%
└─ Availability: 99.999% (5 nines)

API GATEWAY (Go)
├─ Latency: P50 <10ms, P99 <100ms
├─ Throughput: 100,000 requests/sec
├─ Concurrent connections: 1,000,000+
└─ Availability: 99.99%

DATABASE (PostgreSQL)
├─ Write latency: P99 <50ms
├─ Read latency: P99 <10ms
├─ Replication lag: <100ms
└─ RPO (Recovery Point Objective): 0 (synchronous)
└─ RTO (Recovery Time Objective): <5 minutes

FABRIC AUDIT LAYER
├─ Block confirmation: 1-3 seconds
├─ Throughput: 5,000-10,000 TPS
└─ Availability: 99.9%
```

## 5.3 DevOps Practices

```yaml
# kubernetes-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: settlement-service
  namespace: cbdc
spec:
  replicas: 3  # High availability
  template:
    spec:
      containers:
      - name: settlement
        image: bam.ma/settlement-service:v2.1.0
        resources:
          requests:
            memory: "4Gi"
            cpu: "2"
          limits:
            memory: "8Gi"
            cpu: "4"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - settlement
            topologyKey: topology.kubernetes.io/zone  # Spread across AZs
```

---

# PART 6: MIGRATION & PHASED ROLLOUT

## 6.1 Implementation Phases

```
PHASE 1: Foundation (Months 1-6)
├─ Deploy PostgreSQL (Primary + Standby)
├─ Build Rust Settlement Engine (core 80%)
├─ Go API Gateway (basic endpoints)
├─ Tier 1 Wallets (USSD + Web)
└─ Milestone: Handle 1,000 TPS in UAT

PHASE 2: Compliance (Months 7-12)
├─ Deploy Hyperledger Fabric (7 nodes)
├─ Implement Compliance Layer (Go)
├─ Deploy Elasticsearch
├─ Tier 2 KYC Onboarding
└─ Milestone: Handle 10,000 TPS with audit trail

PHASE 3: Privacy & Offline (Months 13-18)
├─ ZKP Engine (Rust)
├─ Blind Signatures (Tier 1 privacy)
├─ Secure Element support (offline)
├─ Offline sync reconciliation
└─ Milestone: Offline transactions working in pilot

PHASE 4: Cross-Border (Months 19-24)
├─ mBridge integration
├─ Icebreaker connector
├─ FX conversion engine
├─ Capital controls enforcement
└─ Milestone: First EUR-MAD remittance via CBDC

PHASE 5: National Rollout (Months 25-36)
├─ Full deployment (all zones)
├─ Merchant network (100K+ agents)
├─ Government G2P programs
├─ Full compliance audit
└─ Milestone: 5M+ active wallets
```

---

# FINAL TECHNOLOGY STACK SUMMARY

```
┌──────────────────────────────────────────────────────────┐
│      DIGITAL DIRHAM v2 - FINAL TECH STACK               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  BACKEND SERVICES                                       │
│  ├─ Settlement Layer: Rust (arkworks-rs, ring)         │
│  ├─ API Gateway: Go (with Istio mTLS)                  │
│  ├─ Compliance: Go (XGBoost, LSTM ML models)           │
│  ├─ Blockchain: Hyperledger Fabric v2.5 (Go chaincode)│
│  └─ Coordination: Kubernetes 1.30+ (3-AZ)              │
│                                                          │
│  DATABASES                                              │
│  ├─ Canonical: Rust-backed Settlement + PostgreSQL WAL │
│  ├─ Operational: PostgreSQL 16 (3-AZ, 4TB storage)    │
│  ├─ Cache: Redis 7.2 (96GB, session + rate limits)    │
│  ├─ TimeSeries: TimescaleDB (metrics, 10-year archive)│
│  ├─ Search: Elasticsearch 8.10 (3 shards x 2 replicas)│
│  └─ Archive: S3 Parquet (cold storage, GDPR compliant)│
│                                                          │
│  CRYPTOGRAPHY                                           │
│  ├─ Signatures: Ed25519 (transactions)                 │
│  ├─ PKI: RSA-2048 (blind signatures) + ECDH            │
│  ├─ zk-SNARKs: Groth16 (balance proofs, Bls12-381)   │
│  ├─ Encryption: AES-256-GCM (PII at rest)             │
│  ├─ Hashing: SHA3-256 (Merkle trees)                   │
│  └─ Post-quantum: CRYSTALS-Dilithium (migration plan) │
│                                                          │
│  MESSAGING & STREAMING                                 │
│  ├─ Event Stream: Apache Kafka (settlement events)     │
│  ├─ Queue: RabbitMQ (task queue for batch jobs)       │
│  └─ Real-time: WebSocket (wallet notifications)        │
│                                                          │
│  FRONTEND                                               │
│  ├─ Web: Next.js 14 (PWA, offline-first)              │
│  ├─ Mobile: React Native (iOS/Android + SE integration)│
│  ├─ Admin: React 18 (dashboard, Grafana for ops)       │
│  └─ USSD: Go-based USSD gateway (*999#)                │
│                                                          │
│  INFRASTRUCTURE & DEVOPS                                │
│  ├─ Container: Docker + Kubernetes (3 AZ)              │
│  ├─ Service Mesh: Istio (mTLS, traffic shaping)       │
│  ├─ Secret: HashiCorp Vault (key management)           │
│  ├─ Monitoring: Prometheus + Grafana (24/7)            │
│  ├─ Logging: ELK Stack (Elasticsearch + Kibana)        │
│  ├─ Tracing: Jaeger (distributed tracing)              │
│  ├─ HSM: Thales Luna (FIPS 140-3 Level 3)              │
│  └─ CDN: Cloudflare (DDoS protection)                   │
│                                                          │
│  INTEGRATION & INTEROP                                  │
│  ├─ APIs: ISO 20022 (settlement), gRPC (internal)     │
│  ├─ mBridge: Custom protocol (cross-border CBDCs)     │
│  ├─ Icebreaker: RESTful hub-and-spoke (multi-CBDC)   │
│  ├─ SWIFT: Legacy integration (interop)                │
│  └─ National ID: CNIE integration (OpenID Connect)    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## DECISION JUSTIFICATION MATRIX

| Technology | Selected | Justification | Alternative | Why Not |
|-----------|----------|---|---|---|
| **Rust** | ✅ (80%) | Memory-safe settlement, low-latency local-path target, no GC | Go, Java | GC pauses unacceptable for settlement |
| **Go** | ✅ (15%) | Cloud-native, goroutines for I/O, simple ops | Java, Node.js | Simpler than Java, faster than Node.js |
| **PostgreSQL** | ✅ Primary | ACID, pgcrypto, RLS, proven at scale | CockroachDB | Sufficient for 3-AZ, lower complexity |
| **Fabric** | ✅ Audit | BFT verification, regulatory trust, proven (eNaira) | Solo, Besu | Audit layer, not replacing Rust for performance |
| **Redis** | ✅ Cache | Fast in-memory, session state, rate limits | Memcached | Better data structures, pub/sub |
| **Elasticsearch** | ✅ Search | Full-text search, scalable indexing | PostgreSQL FTS | Better for large transaction volumes |
| **Kafka** | ✅ Events | Distributed streaming, replay capability, auditable | RabbitMQ | Better for immutable event log |
| **Kubernetes** | ✅ Orchestration | Standard, 3-AZ support, auto-healing | Docker Swarm | More features, widespread adoption |
| **Istio** | ✅ Mesh | mTLS between services, traffic policies | No mesh | Security + observability at scale |
| **Next.js** | ✅ Web | PWA support, offline-first, SSR | React SPA | Critical for offline capability |

---

## Conclusion

The **Digital Dirham v2 technology stack** is optimized for:

1. **SECURITY**: Rust for memory safety + Fabric for verification + cryptography throughout
2. **SCALABILITY**: 50K+ TPS (Rust), 100K+ concurrent (Go), 10-year retention (PostgreSQL)
3. **CBDC REQUIREMENTS**: Low-latency settlement targets, offline support, privacy-preserving compliance
4. **MOROCCAN DEPLOYMENT**: Minimal infrastructure, clear operational model, proven technologies

This stack is an illustrative reference architecture that could be adapted by a central bank (including Morocco-focused scenarios) for secure and scalable CBDC design.
