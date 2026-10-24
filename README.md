# Digital Dirham: Central Bank Digital Currency for Morocco

**Status**: Research & Development (Phase 0: Planning)
**Last Updated**: May 2025
**Project Type**: Personal study repository

> **Disclaimer**  
> This repository is an independent personal study inspired by publicly available CBDC research.  
> It is **not** affiliated with, endorsed by, owned by, or led by Bank Al-Maghrib, and it does not contain confidential Bank Al-Maghrib information.

## 📌 Quick Links

- **[Executive Summary](#executive-summary)** - Start here (2 minutes)
- **[Full BIS Research Paper](./docs/research/BIS-RESEARCH-PAPER.md)** - Complete analysis (1 hour)
- **[Technical Architecture](./docs/ARCHITECTURE.md)** - For engineers
- **[Implementation Roadmap](./docs/implementation/PROJECT-PLAN.md)** - For project managers
- **[Compliance Framework](./docs/compliance/)** - For legal/regulatory

## 📋 Executive Summary

Digital Dirham is a CBDC architecture study for Morocco's financial landscape. It uses the following **illustrative assumptions** (not verified production statistics in this repository):

1. **~50% unbanked assumption** – Used for inclusion-focused scenario planning
2. **~\$7B annual remittance assumption** – Used for remittance cost modeling
3. **~35% low-connectivity assumption** – Used to motivate offline-first design

### Unique Design Features

| Feature | Why It Matters |
|---------|----------------|
| **Offline-first** | Works without internet (USSD, NFC, SE) |
| **Privacy-tiered** | Anonymous for <500 MAD/day; full audit for compliance |
| **Multi-layer double-spend prevention** | Illustrative low fraud target; Bloom false positives can flag/reject valid tx and require reconciliation |
| **mBridge integration** | Sub-10-minute diaspora transfers (<0.5% cost) |
| **Hybrid ledger** | Rust settlement engine + Hyperledger Fabric audit trail |

### Key Metrics (Year 1 Illustrative Targets)

- **5M+ wallets** (12% of population, hypothesis)
- **1M+ daily active users** (hypothesis)
- **Low fraud/double-spend rate target** (illustrative)
- **<1 second settlement (online target); T+24h (offline target)**
- **\$50M+ diaspora volume captured** (hypothesis)
- **99.99% uptime target**

## 🏗️ Architecture Layers

```
Device Layer (Offline)
  ├─ Smartphone app (iOS/Android, Go SDK)
  ├─ Feature phone USSD (*997#)
  ├─ SIM card applet (Java Card, 50KB)
  └─ Hardware wallet (optional, Thales HSM)
  
  ↓ NFC, USSD, LTE
  
API Layer (Online)
  ├─ Go API Gateway (gRPC + REST)
  ├─ Rate limiting, auth, monitoring
  └─ Load balanced (Kubernetes)
  
  ↓ gRPC
  
Settlement Layer
  ├─ Rust Settlement Engine
  ├─ 50K+ TPS throughput
  ├─ Sub-millisecond local processing target (lab benchmark)
  └─ Multi-validator consensus (5-of-7)
  
  ↓ Atomic write
  
Ledger Layer
  ├─ PostgreSQL (3-AZ, synchronous replication)
  ├─ Canonical state (balances, immutable log)
  └─ Row-level security (RLS) + encryption (pgcrypto)
  
  ↓ Async publish
  
Audit Layer
  ├─ Hyperledger Fabric (7 nodes, BFT)
  ├─ Tamper-proof verification
  └─ Independent oversight (HRF, auditors)
```

## 🔐 Privacy Architecture

### Tiered Anonymity Model

**Tier 1: Anonymous (Entry-level)**
- Limit: 500 MAD/day
- KYC: None (phone number only)
- Privacy: Blind signatures (merchant cannot link payer)
- Use case: Street vendor, laborer, refugee

**Tier 2: Semi-Anonymous (Emerging Middle)**
- Limit: 5,000 MAD/day
- KYC: Name + DOB (basic)
- Privacy: Optional blind signature; optional audit
- Use case: Small trader, SME

**Tier 3: Identified (Banked)**
- Limit: Unlimited
- KYC: Full (FATF compliance)
- Privacy: Full audit trail (required)
- Use case: Bank customer, merchant, government employee

## 📱 Offline Payments (Core Innovation)

### Problem
Illustrative assumption: ~35% of Moroccans live in areas with no reliable internet. Existing CBDC designs often assume online connectivity.

### Solution
**Offline-first** architecture:
1. **Device-level settlement**: Secure Element (SE) signs vouchers locally
2. **D2D transfer**: NFC tap between phones (instant, no network)
3. **Deferred reconciliation**: Settlement when online (eventually consistent)
4. **Multi-layer double-spend prevention**: SE balance + nonce + Bloom filter

### Transaction Flow (Offline NFC)

```
T+0min:  Payer & Payee tap phones (NFC contact)
         SE-to-SE transfer of signed voucher
         Both balances updated locally
         Status: "Pending Sync"

T+0-24h: Either party comes online (LTE, USSD, agent)

T+24h:   Merchant syncs → Rust Engine → PostgreSQL
         Settlement confirmed, Fabric records

T+72h:   Payer syncs → Already settled
         "Sync successful (settled 48h ago)"

Result:  Offline transaction with eventual settlement
         No internet required for T+0 gratification
         Audit trail immutable (Fabric)
```

## 🌍 Cross-Border Integration

### mBridge (Multi-CBDC Settlement)

Digital Dirham integrates with mBridge for instant diaspora remittances:

```
Moroccan worker (Dubai) sends 1000 MAD to family (Fez):

T+0s:   Worker app: Send 1000 MAD
T+1s:   Dubai Dirham node: Validate + check sanctions
T+2s:   mBridge consensus: All 5+ nodes agree
T+3s:   Morocco Dirham node: Credit recipient
T+4s:   Both apps show: "Confirmed ✓"

Cost: 0 basis points (direct CBDC transfer)
Time: 4 seconds
vs. Western Union: 15min-3 days, 5.5% fee
vs. Bank transfer: 1-3 days, 2-3% fee
```

## 🛠️ Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Settlement | **Rust** | Memory-safe, high throughput target, low-latency local processing target, no GC |
| API Gateway | **Go** | Cloud-native, 1M+ concurrent, simple ops |
| Ledger | **PostgreSQL** | ACID, 3-AZ replication, pgcrypto, RLS |
| Audit | **Hyperledger Fabric** | BFT consensus, tamper-proof, proven in Africa |
| SE Applet | **Java Card** | Standard for SIM, proven (20M+ cards) |
| Crypto | **Ed25519, RSA, Groth16, AES-256-GCM** | Mixed primitives: AES-256-GCM is NIST-standardized; Ed25519 and Groth16 are used separately and are not "NIST-approved" as a pair |
| Deployment | **Kubernetes** | Auto-scaling, multi-cloud, standard |

## 📈 Implementation Timeline

### Phase 1: Foundation (Q1-Q2 2025) – \$2-3M
- Rust settlement engine
- PostgreSQL 3-AZ cluster
- Go API gateway
- SE firmware
- Pilot: 10K government employees

### Phase 2: Agent Network (Q3-Q4 2025) – \$4-5M
- NFC capability
- USSD feature phone support
- 10K agents in rural areas
- 1M users

### Phase 3: Diaspora Corridor (Q1-Q2 2026) – \$3-4M
- mBridge integration
- 2M diaspora users
- 10% of annual \$7B remittance volume (illustrative target)

### Phase 4: Retail Expansion (Q3-Q4 2026) – \$2-3M
- 50K merchants
- QR code payments
- 5M+ total users

### Phase 5: Financial Services (2027+) – \$2-3M/year
- Loans, insurance, savings
- 10M+ wallets (25% of population)

**Total 3-year**: \$13-18M (illustrative planning estimate)
**ROI (hypothesis)**: 15-30x (illustrative annual benefit estimate: \$240-425M by Year 3)

## 📚 Documentation Structure

```
research/                  – Academic research (BIS paper, literature review)
technical/                 – Implementation details (code specs, pseudocode)
architecture/              – System diagrams and transaction flows
compliance/                – Legal/regulatory framework
implementation/            – Project management (roadmap, budget, hiring)
comparative-analysis/      – Lessons from eNaira, eCedi, Digital Rand
security/                  – Threat model, penetration testing
offline/                   – Offline payment mechanics
cross-border/              – mBridge, Icebreaker integration
operations/                – Runbooks, monitoring, disaster recovery
testing/                   – QA strategy, test plans
deployment/                – Infrastructure, CI/CD, rollout
training/                  – Developer onboarding, architecture primers
stakeholder-comms/         – Executive summaries, parliamentary briefs
```

## 🚀 Getting Started

### For Executives/Decision-Makers
1. Read [Executive Summary](#executive-summary) (this page)
2. Review [2-page brief](./docs/stakeholder-comms/EXECUTIVE-SUMMARY.md)
3. Schedule demo of architecture

### For Engineers
1. Start: [Architecture Overview](./docs/ARCHITECTURE.md)
2. Read: [Technology Stack](./docs/TECHNOLOGY-STACK.md)
3. Clone: [Rust settlement engine example](./docs/technical/rust/settlement-pseudocode.rs)
4. Build: [Local development setup](./docs/training/SETUP-INSTRUCTIONS.md)

### For Regulators/Compliance
1. Read: [Privacy Design](./docs/PRIVACY-DESIGN.md)
2. Review: [Threat Model](./docs/THREAT-MODEL.md)
3. Check: [AML/CFT Compliance](./docs/compliance/AML-CFT-COMPLIANCE.md)
4. Study: [CBDC Law Draft](./docs/compliance/CBDC-LAW-DRAFT.md)

### For Project Managers
1. Review: [Implementation Roadmap](./docs/implementation/PROJECT-PLAN.md)
2. Check: [Phase 1 Checklist](./docs/implementation/PHASE-1-CHECKLIST.md)
3. Study: [Budget Breakdown](./docs/implementation/BUDGET-BREAKDOWN.md)
4. Plan: [Hiring](./docs/implementation/HIRING-PLAN.md)

## 📖 Full Documentation

- **[BIS Research Paper](./docs/research/BIS-RESEARCH-PAPER.md)** (18,000 words)
- **[Technical Architecture](./docs/ARCHITECTURE.md)**
- **[Privacy Design](./docs/PRIVACY-DESIGN.md)**
- **[Offline Payments](./docs/OFFLINE-PAYMENTS.md)**
- **[Cross-Border Integration](./docs/CROSS-BORDER.md)**
- **[Comparative Analysis](./docs/comparative-analysis/)**
- **[Implementation Roadmap](./docs/implementation/PROJECT-PLAN.md)**

## 🔗 External References

- [BIS Papers on CBDC](https://www.bis.org/publ/work_wps.htm)
- [IMF CBDC Research](https://www.imf.org/)
- [Hyperledger Fabric Documentation](https://hyperledger-fabric.readthedocs.io/)
- [mBridge Initiative](https://www.imf.org/en/News/Articles/2023/08/04/mbridge-central-banks-announce-multi-cbdc-platform)
- [eNaira (Central Bank of Nigeria)](https://www.cbn.gov.ng/)
- [eCedi (Bank of Ghana)](https://www.bog.gov.gh/)

## 📞 Contact

- **Maintainer**: Personal study by [@aladin01011](https://github.com/aladin01011)
- **Repository**: [E-dirham-CBDC-Architecture-Prototype-](https://github.com/aladin01011/E-dirham-CBDC-Architecture-Prototype-)
- **Questions**: Open GitHub Issues

## 📜 License

**Personal-study notice (non-confidential):**  
This repository is shared for public technical discussion and learning.  
It does not claim confidential institutional ownership.  
If you need formal reuse terms, add a dedicated `LICENSE` file in a future update.

---

**Last Updated**: May 31, 2025
**Version**: 2.0
