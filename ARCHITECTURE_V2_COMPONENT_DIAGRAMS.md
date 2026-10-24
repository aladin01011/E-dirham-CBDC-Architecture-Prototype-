# Digital Dirham CBDC v2 - Component Architecture Diagrams

## Overview

This document contains comprehensive Mermaid diagrams showing the Digital Dirham v2 architecture with all services, APIs, databases, message queues, and data flows.

> **Disclaimer**  
> This is a personal-study architecture artifact based on publicly available CBDC references. It is not an official Bank Al-Maghrib document and contains no confidential institutional information.

---

## DIAGRAM 1: System Context (High-Level)

```mermaid
graph TB
    subgraph ExternalActors["External Actors & Systems"]
        Citizens["🏠 Citizens & Residents"]
        Merchants["🏪 Merchants & Businesses"]
        Banks["🏦 Commercial Banks"]
        PSP["📱 PSPs (Orange Money, M2T, etc)"]
        CNIE_AUTH["🆔 CNIE Authority (Interior Ministry)"]
        UTRF["🔍 UTRF (Financial Intelligence Unit)"]
        SWIFT["🌐 SWIFT Network"]
        mBRIDGE["🌍 Project mBridge (Cross-Border)"]
    end

    subgraph BAM_CBDC["🏛️ Morocco CBDC Platform (Study Scenario)"]
        API_GATEWAY["API Gateway Layer"]
        CORE_SERVICES["Core Services (All 8 Layers)"]
        DATABASES["Persistent Storage"]
        QUEUES["Message Queues"]
    end

    Citizens -->|Wallet Apps, USSD| API_GATEWAY
    Merchants -->|POS Terminals, Web API| API_GATEWAY
    Banks -->|Wholesale Interface| API_GATEWAY
    PSP -->|Retail Distribution| API_GATEWAY
    CNIE_AUTH -->|Identity Data| CORE_SERVICES
    UTRF -->|Warrant Requests, SARs| CORE_SERVICES
    SWIFT -.->|Settlement| CORE_SERVICES
    mBRIDGE -.->|Cross-Border| CORE_SERVICES
    
    API_GATEWAY -->|Process Transactions| CORE_SERVICES
    CORE_SERVICES -->|Store/Retrieve| DATABASES
    CORE_SERVICES -->|Async Events| QUEUES
    QUEUES -->|Consume| CORE_SERVICES
```

---

## DIAGRAM 2: Full Component Architecture (8 Layers)

```mermaid
graph TB
    subgraph L1["LAYER 1: MONETARY LAYER"]
        IS_ENGINE["Issuance Engine"]
        RED_ENGINE["Redemption Engine"]
        POL_ENGINE["Monetary Policy Engine<br/>(OPA Rules)"]
        SUPP_MONITOR["Supply Monitoring Engine"]
    end

    subgraph L2["LAYER 2: SETTLEMENT LAYER"]
        TX_VALIDATOR["Transaction Validator"]
        CONSENSUS["Consensus Engine<br/>(7-node BFT, 5-of-7 quorum)"]
        LEDGER_MGR["Ledger Manager<br/>(Hyperledger Fabric)"]
        FINALITY["Finality Service"]
    end

    subgraph L3["LAYER 3: COMPLIANCE LAYER"]
        SANC_ENGINE["Sanctions Screening"]
        BEHAV_ANOMALY["Behavioral Anomaly Detector<br/>(ML Models)"]
        SAR_ENGINE["SAR Generator"]
        CAPCTRL_ENGINE["Capital Controls Engine"]
    end

    subgraph L4["LAYER 4: PRIVACY LAYER"]
        BLIND_SIG["Blind Signature Engine<br/>(Tier 1)"]
        PSEUDO_ENGINE["Pseudonymous Identity Engine<br/>(Tier 2)"]
        ZKP_ENGINE["Zero-Knowledge Proof Engine"]
        PRIVACY_AUDIT["Privacy-Preserving Audit"]
    end

    subgraph L5["LAYER 5: IDENTITY LAYER"]
        CNIE_ENGINE["CNIE Integration Engine"]
        KYC_TIER["KYC Tier Assignment"]
        VERIF_CRED["Verifiable Credentials Engine"]
        DID_MANAGER["DID Manager"]
    end

    subgraph L6["LAYER 6: DISTRIBUTION LAYER"]
        WHOLESALE["Wholesale Interface"]
        RETAIL["Retail Wallet Interface"]
        AGENT_MGR["Agent Network Manager"]
        OPEN_API["Open API Gateway"]
    end

    subgraph L7["LAYER 7: OFFLINE LAYER"]
        SE_MGR["Secure Element Manager"]
        OFFLINE_LEDGER["Offline Ledger Cache"]
        SYNC_ENGINE["Sync Engine"]
        DOUBLESPEND["Double-Spend Detector"]
    end

    subgraph L8["LAYER 8: CROSS-BORDER LAYER"]
        mBRIDGE_CONN["mBridge Connector"]
        ICEBREAKER_CONN["Icebreaker Connector"]
        FX_ENGINE["FX Conversion Engine"]
        CAPCTRL_BORDER["Border Capital Controls"]
    end

    subgraph DB["PERSISTENT STORAGE"]
        FABRIC_LEDGER["Hyperledger Fabric Ledger<br/>(Canonical)"]
        PG_MAIN["PostgreSQL (Main)<br/>- Wallets<br/>- Transactions<br/>- KYC Data<br/>- Audit Trail"]
        PG_CACHE["PostgreSQL (Cache)<br/>- Sanctions List Cache<br/>- Bloom Filters<br/>- ML Model State"]
        REDIS["Redis (Hot Cache)<br/>- Session State<br/>- Rate Limits<br/>- ZKP Circuits"]
    end

    subgraph MQ["MESSAGE QUEUES"]
        KAFKA["Apache Kafka<br/>- Settlement Events<br/>- AML Alerts<br/>- Audit Stream"]
    end

    % Interconnections between layers
    L1 -.->|Issues/Retires DD| L2
    L1 -.->|Policy Rules| L3
    L2 -.->|Receives Tx| L3
    L2 -.->|Validates| L4
    L3 -.->|Escalates| UTRF_EXT["UTRF"]
    L4 -.->|Privacy Tier| L5
    L5 -.->|KYC Data| L4
    L6 -.->|User Requests| L1
    L6 -.->|User Requests| L2
    L7 -.->|Offline Sync| L2
    L8 -.->|Cross-Border Tx| L2
    
    % Data flows to storage
    L1 --> FABRIC_LEDGER
    L2 --> FABRIC_LEDGER
    L2 --> PG_MAIN
    L3 --> PG_MAIN
    L5 --> PG_MAIN
    L6 --> REDIS
    L7 --> OFFLINE_LEDGER
    
    % Event flows
    L2 --> KAFKA
    L3 --> KAFKA
    L7 --> KAFKA
    KAFKA -.->|Consume Events| L1
    KAFKA -.->|Consume Events| L3
    KAFKA -.->|Consume Events| L8
```

---

## DIAGRAM 3: Layer 1 - Monetary Layer (Detailed)

```mermaid
graph LR
    subgraph MONETARY["MONETARY LAYER"]
        IS_ENGINE["Issuance Engine<br/>Rust Service"]
        RED_ENGINE["Redemption Engine<br/>Rust Service"]
        POL_ENGINE["Monetary Policy Engine<br/>OPA Rules Engine"]
        SUPP_MONITOR["Supply Monitoring<br/>Real-time Dashboard"]
    end

    subgraph EXTERNALS["External Inputs"]
        BAM_POLICY["BAM Policy Committee<br/>(Quarterly)"]
        MARKET_DATA["Market Data Feed<br/>(FX Rates, Inflation)"]
        FORECAST["Demand Forecast<br/>(ML Model)"]
    end

    subgraph STORAGE["Storage"]
        SUPPLY_LEDGER["Supply Ledger<br/>(Append-only)"]
        POLICY_CODE["Policy Code Repo<br/>(Git)"]
        METRICS_DB["Metrics Database<br/>(TimeSeries)"]
    end

    subgraph DOWNSTREAM["Downstream (Settlement Layer)"]
        SETTLEMENT["Settlement Engine"]
        AML_ENGINE["AML Engine"]
    end

    FORECAST -->|Hourly Demand| IS_ENGINE
    BAM_POLICY -->|Policy Updates| POL_ENGINE
    MARKET_DATA -->|FX, Inflation| POL_ENGINE
    
    IS_ENGINE -->|Allocate Budgets| SUPPLY_LEDGER
    RED_ENGINE -->|Record Redemptions| SUPPLY_LEDGER
    SUPP_MONITOR -->|Monitor Real-time| SUPPLY_LEDGER
    
    POL_ENGINE -->|Deploy Rules| POLICY_CODE
    POL_ENGINE -->|Apply Constraints| IS_ENGINE
    POL_ENGINE -->|Enforce Velocity| RED_ENGINE
    
    SUPP_MONITOR -->|Alert if Anomaly| BAM_POLICY
    SUPP_MONITOR -->|Metrics| METRICS_DB
    
    IS_ENGINE -->|Issue Tokens| SETTLEMENT
    RED_ENGINE -->|Redeem Tokens| SETTLEMENT
    POL_ENGINE -->|Policy Constraints| AML_ENGINE
```

---

## DIAGRAM 4: Layer 2 - Settlement Layer (Detailed)

```mermaid
graph TB
    subgraph SETTLEMENT["SETTLEMENT LAYER"]
        direction TB
        
        TX_VALIDATOR["Transaction Validator<br/>Rust Service"]
        CONSENSUS["Consensus Engine<br/>7-node BFT<br/>(5-of-7 quorum, Hyperledger Fabric)"]
        LEDGER_MGR["Ledger Manager<br/>(UTXO + Account)"]
        FINALITY_SVC["Finality Service<br/>Receipt Generator"]
    end

    subgraph INPUTS["Incoming Transactions"]
        ONLINE_TX["Online Transactions<br/>(Wallet, Merchant, Cross-Border)"]
        OFFLINE_SYNC["Offline Sync Packets<br/>(from Sync Engine)"]
        SETTLEMENT_INIT["Settlement Initiation<br/>(from Monetary Layer)"]
    end

    subgraph OUTPUTS["Transaction Outcomes"]
        FINALITY_RECEIPT["Finality Receipts<br/>(to Wallet/Merchant)"]
        AML_EVENTS["AML Events<br/>(to Compliance Layer)"]
        LEDGER_EVENTS["Ledger State Changes<br/>(to Storage)"]
    end

    subgraph FABRIC_NETWORK["Hyperledger Fabric Network"]
        NODE1["BAM Node 1<br/>(Rabat)"]
        NODE2["BAM Node 2<br/>(Casablanca)"]
        NODE3["BAM Node 3<br/>(Tangier)"]
        INTL_NODES["BAM Nodes 4-7<br/>(International)"]
        ORDERER["Orderer Node<br/>(Consensus)"]
    end

    subgraph STORAGE["Storage"]
        FABRIC_LEDGER["Fabric Ledger<br/>(Append-only)"]
        MERKLE_TREE["Merkle Tree<br/>(UTXO Summary)"]
        BLOOM_FILTER["Bloom Filter<br/>(Spent Tokens)"]
        PG_DERIVED["PostgreSQL<br/>(Derived Indices)"]
    end

    ONLINE_TX -->|Validate Sig| TX_VALIDATOR
    OFFLINE_SYNC -->|Validate Proof| TX_VALIDATOR
    SETTLEMENT_INIT -->|Validate Issuance| TX_VALIDATOR
    
    TX_VALIDATOR -->|Queue for Consensus| CONSENSUS
    
    CONSENSUS -->|Propose Block| ORDERER
    ORDERER -->|Distribute| NODE1
    ORDERER -->|Distribute| NODE2
    ORDERER -->|Distribute| NODE3
    ORDERER -->|Distribute| INTL_NODES
    
    NODE1 -.->|Endorse| ORDERER
    NODE2 -.->|Endorse| ORDERER
    NODE3 -.->|Endorse| ORDERER
    
    CONSENSUS -->|Commit Block| FABRIC_LEDGER
    CONSENSUS -->|Update State| LEDGER_MGR
    
    LEDGER_MGR -->|Update Merkle| MERKLE_TREE
    LEDGER_MGR -->|Update Bloom| BLOOM_FILTER
    
    CONSENSUS -->|Generate Receipt| FINALITY_SVC
    FINALITY_SVC -->|Receipt| FINALITY_RECEIPT
    
    TX_VALIDATOR -->|High-Value Tx| AML_EVENTS
    CONSENSUS -->|Tx Details| PG_DERIVED
```

---

## DIAGRAM 5: Layer 3 - Compliance Layer (Detailed)

```mermaid
graph TB
    subgraph COMPLIANCE["COMPLIANCE LAYER"]
        SANC_ENGINE["Sanctions Screening<br/>Bloom Filter + Fuzzy Match"]
        BEHAV_ANOMALY["Behavioral Anomaly Detector<br/>XGBoost + LSTM + GNN"]
        SAR_ENGINE["SAR Generator<br/>Regulatory Reporting"]
        CAPCTRL_ENGINE["Capital Controls<br/>Daily/Monthly/Annual Limits"]
    end

    subgraph INPUTS["Transaction Flow"]
        INCOMING_TX["Incoming Transaction"]
    end

    subgraph ML_MODELS["ML Models"]
        STRUCTURING_MODEL["Structuring Detector<br/>(LSTM)"]
        NETWORK_MODEL["Network Anomaly<br/>(GNN)"]
        CROSSBORDER_MODEL["Cross-Border Anomaly<br/>(Isolation Forest)"]
    end

    subgraph LISTS["Reference Data"]
        UTRF_LIST["UTRF Sanctions List<br/>(Daily Update)"]
        UN_LIST["UN Consolidated List<br/>(2-hourly Update)"]
        OFAC_LIST["OFAC SDN List<br/>(Daily Update)"]
        REVOCATION_LIST["CBDC Revocation List<br/>(Real-time)"]
    end

    subgraph STORAGE["Storage"]
        BLOOM_CACHE["Bloom Filter Cache<br/>(1M names in 1.2MB)"]
        ML_STATE["ML Model State<br/>(Feature vectors, embeddings)"]
        SAR_DB["SAR Database<br/>(UTRF Eyes Only)"]
        TRANSACTION_LOG["Transaction Log<br/>(for Pattern Detection)"]
    end

    subgraph OUTPUTS["Compliance Decisions"]
        BLOCK["BLOCK Transaction"]
        ESCALATE["ESCALATE to Manual Review"]
        MONITOR["MONITOR (Log for Analysis)"]
        ALLOW["ALLOW Transaction"]
    end

    subgraph EXTERNAL["External Systems"]
        UTRF_SYSTEM["UTRF System<br/>(File SARs)"]
        BAM_DASHBOARD["BAM Dashboard<br/>(Real-time Alerts)"]
    end

    INCOMING_TX -->|Sender/Receiver| SANC_ENGINE
    SANC_ENGINE -->|Check| BLOOM_CACHE
    BLOOM_CACHE -->|Hit = Possible Match| SANC_ENGINE
    SANC_ENGINE -->|Exact Match| OFAC_LIST
    SANC_ENGINE -->|Fuzzy Match| UN_LIST
    
    INCOMING_TX -->|Transaction History| BEHAV_ANOMALY
    BEHAV_ANOMALY -->|Extract Features| TRANSACTION_LOG
    BEHAV_ANOMALY -->|Structuring Detection| STRUCTURING_MODEL
    BEHAV_ANOMALY -->|Network Analysis| NETWORK_MODEL
    BEHAV_ANOMALY -->|Country Trend| CROSSBORDER_MODEL
    BEHAV_ANOMALY -->|Risk Score| SAR_ENGINE
    
    INCOMING_TX -->|Amount + Recipient| CAPCTRL_ENGINE
    CAPCTRL_ENGINE -->|Check Daily Limit| TRANSACTION_LOG
    
    SAR_ENGINE -->|Risk Score > 75| UTRF_SYSTEM
    SAR_ENGINE -->|Record| SAR_DB
    
    SANC_ENGINE -->|Decision| BLOCK
    SANC_ENGINE -->|Decision| ESCALATE
    BEHAV_ANOMALY -->|Decision| ESCALATE
    BEHAV_ANOMALY -->|Decision| MONITOR
    CAPCTRL_ENGINE -->|Decision| BLOCK
    
    INCOMING_TX -->|If All Clear| ALLOW
    
    SANC_ENGINE -->|Daily Update| UTRF_LIST
    SAR_ENGINE -->|Alerts| BAM_DASHBOARD
```

---

## DIAGRAM 6: Layer 4 - Privacy Layer (Detailed)

```mermaid
graph TB
    subgraph PRIVACY["PRIVACY LAYER"]
        BLIND_SIG["Blind Signature Engine<br/>(RSA-FDH)"]
        PSEUDO_ENGINE["Pseudonymous Identity<br/>(ECDH Derivation)"]
        ZKP_ENGINE["Zero-Knowledge Proof<br/>(Groth16 zk-SNARKs)"]
        PRIVACY_AUDIT["Privacy Audit<br/>(Cryptographic Proofs)"]
    end

    subgraph TIER_INPUTS["By KYC Tier"]
        TIER1_TX["Tier 1: Micro Transactions<br/>(<MAD 100)"]
        TIER2_TX["Tier 2: Standard Transactions<br/>(<MAD 50K)"]
        TIER3_TX["Tier 3: High-Value<br/>(Unlimited)"]
    end

    subgraph ALGORITHMS["Cryptographic Algorithms"]
        RSA["RSA-2048<br/>(Blind Signatures)"]
        ECDH["Curve25519 ECDH<br/>(Pseudonym Derivation)"]
        GROTH16["Groth16 zk-SNARKs<br/>(Balance Proofs)"]
    end

    subgraph STORAGE["Storage & Keys"]
        BLIND_TOKENS["Blind Token Store<br/>(Encrypted)"]
        PSEUDONYM_MAP["Pseudonym Map<br/>(User Master Key → Derived Addresses)"]
        ZKP_CIRCUITS["ZKP Circuit Proofs<br/>(Compiled WASM)"]
        AUDIT_COMMITMENTS["Audit Commitments<br/>(Merkle Roots)"]
    end

    subgraph OUTPUTS["Privacy-Preserving Outputs"]
        ANON_PROOF["Anonymity Proof<br/>(to Settlement)"]
        PSEUDONYM["Pseudonym Address<br/>(Transaction Record)"]
        COMPLIANCE_PROOF["Compliance Proof<br/>(to Auditor)"]
        SELECTIVE_DISCLOSURE["Selective Disclosure JWT<br/>(to Receiver)"]
    end

    TIER1_TX -->|Generate Token| BLIND_SIG
    BLIND_SIG -->|RSA Sign| RSA
    RSA -->|Blind Signature| BLIND_TOKENS
    BLIND_TOKENS -->|Spending Proof| ANON_PROOF

    TIER2_TX -->|Derive Address| PSEUDO_ENGINE
    PSEUDO_ENGINE -->|ECDH| ECDH
    ECDH -->|Shared Secret| PSEUDONYM_MAP
    PSEUDONYM_MAP -->|Transaction Under Pseudonym| PSEUDONYM

    TIER2_TX -->|Prove Sufficiency| ZKP_ENGINE
    TIER3_TX -->|Prove Compliance| ZKP_ENGINE
    ZKP_ENGINE -->|Groth16| GROTH16
    GROTH16 -->|Proof| ZKP_CIRCUITS
    ZKP_CIRCUITS -->|Compliance Proof| COMPLIANCE_PROOF

    COMPLIANCE_PROOF -->|Aggregate Stats| PRIVACY_AUDIT
    AUDIT_COMMITMENTS -->|Merkle Root| PRIVACY_AUDIT
    PRIVACY_AUDIT -->|Auditor Verification| COMPLIANCE_PROOF

    PSEUDO_ENGINE -->|Share Selected Claims| SELECTIVE_DISCLOSURE
    SELECTIVE_DISCLOSURE -->|Receiver Sees Only Needed Attributes| OUTPUTS
```

---

## DIAGRAM 7: Layer 5 - Identity Layer (Detailed)

```mermaid
graph TB
    subgraph IDENTITY["IDENTITY LAYER"]
        CNIE_ENGINE["CNIE Integration<br/>Digital Signature Verification"]
        KYC_TIER["KYC Tier Assignment<br/>Tier 1/2/3 Classification"]
        VERIF_CRED["Verifiable Credentials<br/>W3C VC Format"]
        DID_MANAGER["DID Manager<br/>Decentralized Identifiers"]
    end

    subgraph EXTERNAL["External Systems"]
        CNIE_DB["CNIE Database<br/>(Interior Ministry)"]
        EMPLOYER_DB["Employer Registry<br/>(CNSS)"]
        TAX_DB["Tax Authority<br/>(DGED)"]
    end

    subgraph BIOMETRIC["Biometric Processing"]
        FACIAL_RECOG["Facial Recognition<br/>(ML Model)"]
        LIVENESS_CHECK["Liveness Check<br/>(Multi-modal)"]
        FINGERPRINT["Fingerprint Matching<br/>(SE-based)"]
    end

    subgraph STORAGE["Storage"]
        KYC_DATA["KYC Data<br/>(Encrypted PG)"]
        CNIE_HASH["CNIE Hash Map<br/>(De-duplication)"]
        TIER_ASSIGNMENTS["Tier Assignments<br/>(with Expiry)"]
        CREDENTIALS_DB["Verifiable Credentials<br/>(W3C VC Store)"]
        DID_REGISTRY["DID Registry<br/>(DID → Public Key)"]
    end

    subgraph OUTPUTS["Identity Outputs"]
        TIER1_ONBOARD["Tier 1 Wallet<br/>(Minimal KYC)"]
        TIER2_ONBOARD["Tier 2 Wallet<br/>(Standard KYC)"]
        TIER3_ONBOARD["Tier 3 Wallet<br/>(Full KYC)"]
        VERIF_CRED_ISSUED["VC Issued<br/>(Signed Attestation)"]
    end

    CNIE_DB -->|Digital Signature| CNIE_ENGINE
    CNIE_ENGINE -->|Verify Signature| FACIAL_RECOG
    FACIAL_RECOG -->|Compare Photo| LIVENESS_CHECK
    LIVENESS_CHECK -->|Spoof Detect| CNIE_ENGINE

    CNIE_ENGINE -->|Classify| KYC_TIER
    KYC_TIER -->|Compute Hash| CNIE_HASH
    CNIE_HASH -->|De-duplicate| KYC_DATA

    KYC_TIER -->|Assign Tier 1| TIER_ASSIGNMENTS
    KYC_TIER -->|Assign Tier 2| EMPLOYER_DB
    KYC_TIER -->|Assign Tier 3| TAX_DB

    KYC_TIER -->|Generate VC| VERIF_CRED
    VERIF_CRED -->|Issue Credential| CREDENTIALS_DB
    CREDENTIALS_DB -->|Signed VC| VERIF_CRED_ISSUED

    VERIF_CRED -->|Generate DID| DID_MANAGER
    DID_MANAGER -->|Register DID| DID_REGISTRY
    DID_REGISTRY -->|DID → Public Key| OUTPUTS

    KYC_DATA -->|Link to Tier| TIER1_ONBOARD
    KYC_DATA -->|Link to Tier| TIER2_ONBOARD
    KYC_DATA -->|Link to Tier| TIER3_ONBOARD
```

---

## DIAGRAM 8: Layer 6 - Distribution Layer (Detailed)

```mermaid
graph TB
    subgraph DISTRIBUTION["DISTRIBUTION LAYER"]
        WHOLESALE["Wholesale Interface<br/>Bank Omnibus Accounts"]
        RETAIL["Retail Wallet Interface<br/>Customer-Facing UIs"]
        AGENT_MGR["Agent Network Manager<br/>Barber Shops, Pharmacies"]
        OPEN_API["Open API Gateway<br/>Third-Party Integration"]
    end

    subgraph WALLET_UIS["Wallet Applications"]
        WEB_WALLET["Web Wallet<br/>(Next.js PWA)"]
        MOBILE_ANDROID["Mobile App<br/>(Android - React Native)"]
        MOBILE_IOS["Mobile App<br/>(iOS - React Native)"]
        USSD_GATEWAY["USSD Gateway<br/>(*999# Menu)"]
    end

    subgraph MERCHANT_INTERFACE["Merchant Integration"]
        POS_TERMINAL["POS Terminal<br/>(NFC, QR, Offline)"]
        WEB_CHECKOUT["Web Checkout<br/>(API Integration)"]
        MERCHANT_DASHBOARD["Merchant Dashboard<br/>(Settlement, Reconciliation)"]
    end

    subgraph BACKEND_SERVICES["Backend Services"]
        SESSION_MGR["Session Manager<br/>(Redis-backed)"]
        RATE_LIMITER["Rate Limiter<br/>(Token Bucket)"]
        NOTIFICATION["Notification Service<br/>(SMS, Push, Email)"]
        AUDIT_LOG["Audit Logger<br/>(Every API Call)"]
    end

    subgraph EXTERNAL_PARTNERS["External Partners"]
        BANK_SYSTEMS["Bank Core Systems<br/>(for KYC, Settlement)"]
        MERCHANT_BANKS["Merchant Acquiring Banks"]
        THIRD_PARTY_DEVS["Third-Party Developers<br/>(OAuth Integration)"]
    end

    subgraph STORAGE["Storage"]
        WALLET_DB["Wallet Database<br/>(User Profiles)"]
        TRANSACTION_HISTORY["Transaction History<br/>(PG)"]
        SESSION_STORE["Session Store<br/>(Redis)"]
        API_KEYS["API Keys<br/>(OAuth Tokens)"]
    end

    BANK_SYSTEMS -->|Settlement Requests| WHOLESALE
    WHOLESALE -->|Update Omnibus| WALLET_DB
    WHOLESALE -->|Tx Confirmation| BANK_SYSTEMS

    RETAIL -->|Serve HTML| WEB_WALLET
    WEB_WALLET -->|User Interaction| SESSION_MGR
    SESSION_MGR -->|Store Session| SESSION_STORE

    RETAIL -->|Android App| MOBILE_ANDROID
    MOBILE_ANDROID -->|Biometric Auth| RATE_LIMITER
    RETAIL -->|iOS App| MOBILE_IOS

    RETAIL -->|USSD Routing| USSD_GATEWAY
    USSD_GATEWAY -->|Feature Phone| SESSION_MGR

    POS_TERMINAL -->|QR/NFC Tx| OPEN_API
    WEB_CHECKOUT -->|Web API| OPEN_API
    OPEN_API -->|Validate Request| RATE_LIMITER

    RATE_LIMITER -->|Call Settlement| OPEN_API
    OPEN_API -->|Generate Receipt| MERCHANT_DASHBOARD
    MERCHANT_DASHBOARD -->|Display Results| MERCHANT_BANKS

    OPEN_API -->|Log Event| AUDIT_LOG
    AUDIT_LOG -->|Store| TRANSACTION_HISTORY
    SESSION_MGR -->|Send Alert| NOTIFICATION
    NOTIFICATION -->|SMS/Push| RETAIL

    THIRD_PARTY_DEVS -->|API Token| API_KEYS
    THIRD_PARTY_DEVS -->|Construct Request| OPEN_API
    OPEN_API -->|Route to Backend| SETTLEMENT["Settlement Layer"]
```

---

## DIAGRAM 9: Layer 7 - Offline Layer (Detailed)

```mermaid
graph TB
    subgraph OFFLINE["OFFLINE LAYER"]
        SE_MGR["Secure Element Manager<br/>Hardware-backed Key Storage"]
        OFFLINE_LEDGER["Offline Ledger Cache<br/>Merkle Root + Bloom Filter"]
        SYNC_ENGINE["Sync Engine<br/>Reconciliation Protocol"]
        DOUBLESPEND["Double-Spend Detector<br/>Duplicate + Bloom-Flag Review"]
    end

    subgraph OFFLINE_DEVICE["Mobile Device (SE-backed)"]
        SE_CHIP["Secure Element Chip<br/>(eSE, SIM-SE, NFC Card)"]
        WALLET_APP["Wallet App<br/>(Offline-First)"]
        NFC_RADIO["NFC Radio<br/>(ISO 18092)"]
    end

    subgraph PAYMENT_FLOW["Offline Payment Flow"]
        PAYER_SE["Payer SE<br/>- Private Key<br/>- Merkle Root<br/>- Spent Cache"]
        RECEIVER_SE["Receiver SE<br/>- Public Key<br/>- Spent Bloom<br/>- Tx Counter"]
        TX_PACKET["Transaction Packet<br/>(Signed Token)"]
    end

    subgraph STORAGE["Storage"]
        MERKLE_ROOTS["Daily Merkle Roots<br/>(Published)"]
        BLOOM_SNAPSHOTS["Bloom Filter Snapshots<br/>(7-day retention)"]
        OFFLINE_LOG["Offline Tx Log<br/>(Local, Encrypted)"]
        SYNC_QUEUE["Sync Queue<br/>(Pending Reconciliation)"]
    end

    subgraph SYNC_PROCESS["Sync Process (When Online)"]
        SYNC_CLIENT["Sync Client<br/>(in Wallet App)"]
        SYNC_SERVER["Sync Server<br/>(BAM Backend)"]
        RECONCILIATION["Reconciliation Engine<br/>(Merkle Proof Verification)"]
        LEDGER_FINALITY["Ledger Finality Check"]
    end

    WALLET_APP -->|Request Token| SE_MGR
    SE_MGR -->|Generate Token| SE_CHIP
    SE_CHIP -->|Store Private Key| OFFLINE_LEDGER

    WALLET_APP -->|Cache Merkle| OFFLINE_LEDGER
    OFFLINE_LEDGER -->|Update Bloom| OFFLINE_LEDGER

    PAYER_SE -->|Sign Tx| TX_PACKET
    PAYER_SE -->|Mark Spent| OFFLINE_LOG
    NFC_RADIO -->|NFC Payment| RECEIVER_SE

    RECEIVER_SE -->|Verify Sig| TX_PACKET
    RECEIVER_SE -->|Check Bloom| DOUBLESPEND
    RECEIVER_SE -->|Record Tx| OFFLINE_LOG

    WALLET_APP -->|Connect Online| SYNC_CLIENT
    SYNC_CLIENT -->|Submit Offline Log| SYNC_SERVER
    SYNC_SERVER -->|Process Reconciliation| RECONCILIATION

    RECONCILIATION -->|Check Merkle Root| MERKLE_ROOTS
    RECONCILIATION -->|Verify Proofs| DOUBLESPEND
    RECONCILIATION -->|Update Ledger| LEDGER_FINALITY

    LEDGER_FINALITY -->|Confirm Settlement| OFFLINE_LOG
    SYNC_CLIENT -->|Confirm| WALLET_APP
    WALLET_APP -->|Update Balance| SE_CHIP
```

---

## DIAGRAM 10: Layer 8 - Cross-Border Layer (Detailed)

```mermaid
graph TB
    subgraph CROSSBORDER["CROSS-BORDER LAYER"]
        mBRIDGE_CONN["mBridge Connector<br/>Multi-CBDC Settlement"]
        ICEBREAKER_CONN["Icebreaker Connector<br/>Hub-and-Spoke API"]
        FX_ENGINE["FX Conversion Engine<br/>Rate Locking"]
        CAPCTRL_BORDER["Border Capital Controls<br/>Daily/Monthly Limits"]
    end

    subgraph CORRIDORS["Remittance Corridors"]
        EUR_CORRIDOR["EUR-MAD Corridor<br/>(France, Spain, Belgium)"]
        SAR_CORRIDOR["SAR-MAD Corridor<br/>(Gulf Countries)"]
        USD_CORRIDOR["USD-MAD Corridor<br/>(North America)"]
        INTRA_MENA["Intra-MENA Corridor<br/>(Tunisia, Algeria)"]
    end

    subgraph EXTERNAL_CBDCS["External CBDC Systems"]
        ECB_DIGITAL_EUR["ECB Digital Euro<br/>(Eurozone)"]
        PBC_eRMB["PBC e-CNY<br/>(China)"]
        OTHER_CBDCS["Other Central Banks"]
    end

    subgraph FX_MANAGEMENT["FX Management"]
        SPOT_RATE["Spot Rate Feed<br/>(Real-time)"]
        RATE_LOCK["Rate Lock Service<br/>(30-day)"]
        SLIPPAGE_MGMT["Slippage Management<br/>(Limit Orders)"]
        FX_RESERVE["FX Reserve Account<br/>(BAM Treasury)"]
    end

    subgraph CAPITAL_CONTROL["Capital Control Rules"]
        DAILY_LIMITS["Daily Limits per Person<br/>(<EUR 5,000)"]
        MONTHLY_LIMITS["Monthly Limits<br/>(<EUR 50,000)"]
        ANNUAL_LIMITS["Annual Limits<br/>(<EUR 500,000)"]
        COUNTRY_BLOCKS["High-Risk Country Blocks<br/>(Syria, Iran)"]
    end

    subgraph STORAGE["Storage"]
        CROSSBORDER_LOG["Cross-Border Transaction Log<br/>(Audit Trail)"]
        RATE_HISTORY["FX Rate History<br/>(TimeSeries DB)"]
        MONTHLY_COUNTERS["Monthly Counters<br/>(per User)"]
        LOCKED_RATES["Locked Rates<br/>(30-day Contracts)"]
    end

    subgraph EXTERNAL_SYSTEMS["External Systems"]
        SWIFT["SWIFT Network<br/>(Settlement)"]
        UTRF["UTRF<br/>(SAR Reporting)"]
        BIS["BIS Innovation Hub<br/>(mBridge Ops)"]
    end

    EUR_CORRIDOR -->|Sender in France| mBRIDGE_CONN
    EUR_CORRIDOR -->|Recipient in Morocco| mBRIDGE_CONN
    mBRIDGE_CONN -->|Digital EUR → DD| FX_ENGINE
    mBRIDGE_CONN -->|Settlement| ECB_DIGITAL_EUR

    SAR_CORRIDOR -->|Saudi Arabia, UAE| ICEBREAKER_CONN
    ICEBREAKER_CONN -->|Hub-and-Spoke| EXTERNAL_CBDCS

    CAPCTRL_BORDER -->|Check Limit| DAILY_LIMITS
    CAPCTRL_BORDER -->|Check Limit| MONTHLY_LIMITS
    CAPCTRL_BORDER -->|Check Limit| ANNUAL_LIMITS
    CAPCTRL_BORDER -->|Block High-Risk| COUNTRY_BLOCKS

    FX_ENGINE -->|Get Rate| SPOT_RATE
    FX_ENGINE -->|Lock Rate| RATE_LOCK
    RATE_LOCK -->|Store Contract| LOCKED_RATES
    FX_ENGINE -->|Convert Amount| FX_RESERVE

    mBRIDGE_CONN -->|Send Settlement| SWIFT
    mBRIDGE_CONN -->|Report High-Value| UTRF
    mBRIDGE_CONN -->|Coordination| BIS

    CAPCTRL_BORDER -->|Log Tx| CROSSBORDER_LOG
    RATE_LOCK -->|Log Rate| RATE_HISTORY
    CAPCTRL_BORDER -->|Update Counter| MONTHLY_COUNTERS
```

---

## DIAGRAM 11: API Gateway & External Integration

```mermaid
graph TB
    subgraph EXTERNAL["External Users & Systems"]
        CITIZENS["👥 Citizens<br/>(Web/Mobile)"]
        MERCHANTS["🏪 Merchants<br/>(POS/Web)"]
        BANKS["🏦 Banks<br/>(Wholesale)"]
        THIRD_PARTY["🔗 Third-Party Devs<br/>(OAuth)"]
        UTRF["🔍 UTRF<br/>(Warrant API)"]
    end

    subgraph API_GATEWAY_LAYER["API Gateway (Kong/AWS APIGateway)"]
        REST_API["REST API<br/>/api/v1/"]
        GRPC_API["gRPC API<br/>(Internal Only)"]
        WEBSOCKET["WebSocket<br/>(Real-time Notifications)"]
        OAUTH_SERVER["OAuth 2.0 Server<br/>(Token Management)"]
    end

    subgraph AUTHENTICATION["Authentication & AuthZ"]
        JWT_VALIDATOR["JWT Validator"]
        RATE_LIMITER["Rate Limiter<br/>(per API key)"]
        IP_WHITELIST["IP Whitelist<br/>(for Banks)"]
        CERTIFICATE_AUTH["mTLS Certificate Auth<br/>(for Fintechs)"]
    end

    subgraph ROUTING["Request Routing"]
        ENDPOINT_MAPPER["Endpoint Mapper<br/>(Route to Service)"]
        REQUEST_TRANSFORMER["Request Transformer<br/>(Normalize Input)"]
        RESPONSE_FORMATTER["Response Formatter<br/>(Consistent Output)"]
    end

    subgraph BACKENDS["Backend Services (via gRPC)"]
        WALLET_SVC["Wallet Service"]
        SETTLEMENT_SVC["Settlement Service"]
        COMPLIANCE_SVC["Compliance Service"]
        IDENTITY_SVC["Identity Service"]
    end

    subgraph LOGGING["Logging & Monitoring"]
        AUDIT_LOGGER["Audit Logger<br/>(Every call)"]
        METRICS["Prometheus Metrics"]
        TRACES["OpenTelemetry Traces"]
    end

    CITIZENS -->|HTTPS| REST_API
    MERCHANTS -->|HTTPS| REST_API
    BANKS -->|mTLS| CERTIFICATE_AUTH
    THIRD_PARTY -->|OAuth Token| OAUTH_SERVER
    UTRF -->|Warrant API| REST_API

    REST_API -->|Validate| JWT_VALIDATOR
    REST_API -->|Limit| RATE_LIMITER
    CERTIFICATE_AUTH -->|Verify Cert| BANKS

    JWT_VALIDATOR -->|Extract Identity| ENDPOINT_MAPPER
    ENDPOINT_MAPPER -->|Route| REQUEST_TRANSFORMER
    REQUEST_TRANSFORMER -->|Normalize| BACKENDS

    BACKENDS -->|gRPC Response| RESPONSE_FORMATTER
    RESPONSE_FORMATTER -->|Return| REST_API

    REST_API -->|Event| AUDIT_LOGGER
    REST_API -->|Metric| METRICS
    BACKENDS -->|Span| TRACES

    AUDIT_LOGGER -->|Store| AUDIT_DB["Audit Database<br/>(TimescaleDB)"]
    METRICS -->|Scrape| PROMETHEUS["Prometheus"]
    TRACES -->|Collect| JAEGER["Jaeger UI"]
```

---

## DIAGRAM 12: Message Queue Architecture (Kafka)

```mermaid
graph TB
    subgraph PRODUCERS["Event Producers"]
        SETTLEMENT["Settlement Layer"]
        COMPLIANCE["Compliance Layer"]
        OFFLINE_SYNC["Offline Sync Engine"]
        CROSSBORDER["Cross-Border Layer"]
    end

    subgraph KAFKA_TOPICS["Kafka Topics (Partitioned)"]
        TOPIC_SETTLEMENT["settlement-events<br/>(Partition: by tx_id)"]
        TOPIC_AML["aml-alerts<br/>(Partition: by risk_score)"]
        TOPIC_OFFLINE["offline-sync<br/>(Partition: by wallet_id)"]
        TOPIC_CROSSBORDER["crossborder-tx<br/>(Partition: by corridor)"]
        TOPIC_AUDIT["audit-stream<br/>(Partition: by timestamp)"]
    end

    subgraph CONSUMERS["Event Consumers"]
        COMPLIANCE_CONSUMER["Compliance Consumer<br/>(AML Analysis)"]
        ANALYTICS_CONSUMER["Analytics Consumer<br/>(Real-time Dashboard)"]
        AUDIT_CONSUMER["Audit Consumer<br/>(Archive to TimescaleDB)"]
        NOTIFICATION_CONSUMER["Notification Consumer<br/>(Alert Users)"]
    end

    subgraph CONSUMER_GROUPS["Consumer Groups"]
        CG_COMPLIANCE["compliance-group<br/>(Offset: saved to DB)"]
        CG_ANALYTICS["analytics-group"]
        CG_AUDIT["audit-group"]
        CG_NOTIFICATIONS["notification-group"]
    end

    subgraph STORAGE["Long-term Storage"]
        TIMESCALE_DB["TimescaleDB<br/>(Audit Archive)"]
        DATAWAREHOUSE["Data Warehouse<br/>(Parquet + S3)"]
        SEARCHDB["Elasticsearch<br/>(Full-text Search)"]
    end

    SETTLEMENT -->|Publish| TOPIC_SETTLEMENT
    COMPLIANCE -->|Publish| TOPIC_AML
    OFFLINE_SYNC -->|Publish| TOPIC_OFFLINE
    CROSSBORDER -->|Publish| TOPIC_CROSSBORDER
    SETTLEMENT -->|Publish| TOPIC_AUDIT

    TOPIC_SETTLEMENT -->|Consume| COMPLIANCE_CONSUMER
    COMPLIANCE_CONSUMER -->|Subscribe| CG_COMPLIANCE

    TOPIC_AML -->|Consume| ANALYTICS_CONSUMER
    ANALYTICS_CONSUMER -->|Subscribe| CG_ANALYTICS

    TOPIC_AUDIT -->|Consume| AUDIT_CONSUMER
    AUDIT_CONSUMER -->|Archive| TIMESCALE_DB
    AUDIT_CONSUMER -->|Subscribe| CG_AUDIT

    TOPIC_AML -->|Consume| NOTIFICATION_CONSUMER
    NOTIFICATION_CONSUMER -->|Send Alert| CITIZEN_PHONE["📱 Citizen Phone<br/>(SMS/Push)"]

    DATAWAREHOUSE -->|Query| ANALYTICS["Analytics Engine<br/>(Tableau/Looker)"]
    SEARCHDB -->|Query| SEARCHUI["Search UI<br/>(Find Transactions)"]
```

---

## DIAGRAM 13: Database Architecture

```mermaid
graph TB
    subgraph PRIMARY_DBS["Primary Databases"]
        FABRIC_LEDGER["Hyperledger Fabric Ledger<br/>(Canonical)<br/>- Block History<br/>- Merkle Trees<br/>- Chaincode State"]
        
        PG_MAINDB["PostgreSQL (Primary)<br/>- Wallets (KYC Tier, Balance)<br/>- Transactions (for Indices)<br/>- KYC Data (Encrypted)<br/>- Audit Trail"]
        
        REDIS_CACHE["Redis (Hot Cache)<br/>- Session State<br/>- Rate Limit Counters<br/>- ZKP Circuits<br/>- Merchant Balance<br/>- TTL: 24 hours"]
    end

    subgraph DERIVED_DBS["Derived Databases"]
        PG_REPLICAS["PostgreSQL Read Replicas<br/>(Analytics)<br/>- Materialized Views<br/>- Transaction Summaries<br/>- Customer Segments"]
        
        TIMESCALE_DB["TimescaleDB<br/>(Time-Series)<br/>- Supply Metrics<br/>- Transaction Volume<br/>- Price History<br/>- Audit Events"]
        
        ELASTICSEARCH["Elasticsearch<br/>(Full-text Search)<br/>- Transaction Search<br/>- Merchant Directory<br/>- Regulatory Reporting"]
    end

    subgraph OFFLINE_CACHES["Offline Caches"]
        BLOOM_FILTER["Bloom Filter<br/>(Sanctions)<br/>- 1M names in 1.2MB<br/>- Cached on SE"]
        
        MERKLE_ROOT_CACHE["Merkle Root Cache<br/>(Daily Published)<br/>- Enables Offline Verification<br/>- Signed by BAM"]
    end

    subgraph EXTERNAL_SYSTEMS["External Data Feeds"]
        CNIE_DB["CNIE Database<br/>(Read-only)<br/>- Interior Ministry"]
        
        SANCTIONS_FEEDS["Sanctions List Feeds<br/>- UTRF (Daily)<br/>- UN (2-hourly)<br/>- OFAC (Daily)"]
    end

    subgraph REPLICATION["Replication & Backup"]
        PG_STANDBY["PostgreSQL Standby<br/>(Synchronous Replication)<br/>- Same data center"]
        
        FABRIC_PEERS["Fabric Peer Backups<br/>(Full Ledger Copy)<br/>- International Sites"]
        
        S3_BACKUP["S3 Backup Vault<br/>(Daily Snapshot)<br/>- Encrypted<br/>- 10-year Retention"]
    end

    FABRIC_LEDGER -->|Source of Truth| PG_MAINDB
    PG_MAINDB -->|Hot Data| REDIS_CACHE
    PG_MAINDB -->|Replicate Async| PG_REPLICAS
    PG_MAINDB -->|Export to| TIMESCALE_DB
    PG_MAINDB -->|Index| ELASTICSEARCH

    PG_MAINDB -->|Backup| PG_STANDBY
    FABRIC_LEDGER -->|Replicate| FABRIC_PEERS
    PG_MAINDB -->|Archive| S3_BACKUP

    SANCTIONS_FEEDS -->|Update| BLOOM_FILTER
    MERKLE_ROOT_CACHE -->|Publish Daily| MERKLE_ROOT_CACHE
    
    CNIE_DB -->|Query on Demand| FABRIC_LEDGER
    SANCTIONS_FEEDS -->|Ingest| ELASTICSEARCH

    style FABRIC_LEDGER fill:#c1f0fc,stroke:#0066cc,stroke-width:3px
    style PG_MAINDB fill:#e1f5ff,stroke:#0066cc,stroke-width:2px
    style REDIS_CACHE fill:#fff9c4,stroke:#ffb300,stroke-width:2px
```

---

## DIAGRAM 14: Data Flow - Complete Transaction Journey

```mermaid
graph LR
    subgraph USER["User/Wallet"]
        USER_WALLET["Wallet App<br/>User Initiates Transfer"]
    end

    subgraph API["API Gateway"]
        API_LAYER["API Gateway<br/>(Validate Request)"]
    end

    subgraph TIER5["Identity Verification"]
        KYC_CHECK["KYC Tier Check<br/>(Layer 5)"]
    end

    subgraph TIER4["Privacy Preparation"]
        PRIVACY_PROOF["Generate Privacy Proof<br/>Blind Sig / ZKP<br/>(Layer 4)"]
    end

    subgraph TIER3["Compliance Screening"]
        COMPLIANCE_SCREEN["Compliance Screening<br/>- Sanctions Check<br/>- AML Behavioral<br/>- Capital Controls<br/>(Layer 3)"]
    end

    subgraph TIER2["Settlement"]
        VALIDATE["Validate Signature<br/>Check Balance<br/>(Layer 2)"]
        CONSENSUS["BFT Consensus<br/>(7-node, 5-of-7 quorum)"]
        LEDGER_UPDATE["Update Ledger<br/>(Fabric)"]
        FINALITY["Generate Finality Receipt<br/>(Layer 2)"]
    end

    subgraph TIER1["Monetary Update"]
        SUPPLY_UPDATE["Update Supply<br/>Monitoring<br/>(Layer 1)"]
    end

    subgraph STORAGE["Storage"]
        DB_COMMIT["Commit to DB<br/>- Fabric Ledger<br/>- PostgreSQL<br/>- Redis"]
    end

    subgraph QUEUE["Event Stream"]
        EVENT_PUBLISH["Publish to Kafka<br/>- Settlement Event<br/>- Audit Trail"]
    end

    subgraph RESPONSE["Response Path"]
        USER_NOTIFICATION["Notify User<br/>- Finality Receipt<br/>- Transaction Hash<br/>- Confirmation"]
    end

    USER_WALLET -->|POST /transfer| API_LAYER
    API_LAYER -->|JWT Validate| KYC_CHECK
    KYC_CHECK -->|Tier + Limits| PRIVACY_PROOF
    PRIVACY_PROOF -->|Privacy-Wrapped Tx| COMPLIANCE_SCREEN
    COMPLIANCE_SCREEN -->|Allow/Block| VALIDATE
    VALIDATE -->|Valid| CONSENSUS
    CONSENSUS -->|Propose Block| LEDGER_UPDATE
    CONSENSUS -->|Endorse Block| CONSENSUS
    LEDGER_UPDATE -->|Block Committed| FINALITY
    FINALITY -->|Receipt| SUPPLY_UPDATE
    SUPPLY_UPDATE -->|Metrics| DB_COMMIT
    LEDGER_UPDATE -->|Tx Record| DB_COMMIT
    DB_COMMIT -->|Event| EVENT_PUBLISH
    EVENT_PUBLISH -->|Async| QUEUE
    QUEUE -.->|Analytics| ANALYTICS["Analytics<br/>Consumer"]
    FINALITY -->|Receipt Hash| USER_NOTIFICATION
    USER_NOTIFICATION -->|SMS/Push| USER_WALLET

    style USER_WALLET fill:#c8e6c9,stroke:#2e7d32
    style CONSENSUS fill:#bbdefb,stroke:#1565c0,stroke-width:3px
    style LEDGER_UPDATE fill:#e1bee7,stroke:#6a1b9a,stroke-width:2px
    style FINALITY fill:#ffe0b2,stroke:#e65100,stroke-width:2px
```

---

## DIAGRAM 15: Infrastructure Deployment (Kubernetes)

```mermaid
graph TB
    subgraph KUBERNETES["Kubernetes Cluster (3 Availability Zones)"]
        
        subgraph AZ1["Availability Zone 1 (Rabat)"]
            K8S_AZ1["K8s Nodes<br/>(3x High-Memory)"]
            POD_AZ1["Pods:<br/>- Settlement Service<br/>- API Gateway<br/>- Wallet API<br/>- Compliance Engine"]
            STORAGE_AZ1["Storage:<br/>- Fabric Peer (AZ1)<br/>- PG Primary"]
        end
        
        subgraph AZ2["Availability Zone 2 (Casablanca)"]
            K8S_AZ2["K8s Nodes<br/>(3x Standard)"]
            POD_AZ2["Pods:<br/>- Identity Service<br/>- Privacy Layer<br/>- Distribution Layer"]
            STORAGE_AZ2["Storage:<br/>- Fabric Peer (AZ2)<br/>- Redis Cache"]
        end
        
        subgraph AZ3["Availability Zone 3 (Tangier)"]
            K8S_AZ3["K8s Nodes<br/>(2x Standard)"]
            POD_AZ3["Pods:<br/>- Cross-Border Service<br/>- Analytics<br/>- Logging"]
            STORAGE_AZ3["Storage:<br/>- Fabric Peer (AZ3)<br/>- Elasticsearch"]
        end
        
        subgraph ISTIO["Istio Service Mesh"]
            INGRESS["Ingress Controller<br/>(TLS Termination)"]
            SIDECARS["Envoy Sidecars<br/>(Per Pod)<br/>- mTLS<br/>- Traffic Shaping"]
        end
        
        subgraph MONITORING["Monitoring & Logging"]
            PROMETHEUS["Prometheus<br/>(Metrics)"]
            GRAFANA["Grafana<br/>(Dashboards)"]
            JAEGER["Jaeger<br/>(Distributed Traces)"]
            FLUENTD["Fluentd<br/>(Log Collection)"]
        end
    end

    subgraph EXTERNAL_INGRESS["External Ingress"]
        CDN["CloudFlare CDN<br/>(DDoS Mitigation)"]
        WAF["WAF<br/>(SQL Injection, XSS)"]
    end

    subgraph EXTERNAL_DATA["External Data"]
        CNIE_FEED["CNIE Authority<br/>(Identity Feed)"]
        SANCTIONS_FEED["Sanctions Lists<br/>(Real-time)"]
        FX_FEED["FX Rate Feed<br/>(Bloomberg)"]
    end

    CDN -->|Route| WAF
    WAF -->|HTTPS| INGRESS
    INGRESS -->|Route by Path| POD_AZ1
    INGRESS -->|Route by Path| POD_AZ2
    INGRESS -->|Route by Path| POD_AZ3

    POD_AZ1 -.->|Query| STORAGE_AZ1
    POD_AZ1 -.->|Update| STORAGE_AZ2
    POD_AZ1 -.->|Replicate| STORAGE_AZ3

    SIDECARS -->|mTLS| SIDECARS

    POD_AZ1 -->|Metrics| PROMETHEUS
    POD_AZ2 -->|Metrics| PROMETHEUS
    POD_AZ3 -->|Metrics| PROMETHEUS
    PROMETHEUS -->|Visualize| GRAFANA

    POD_AZ1 -->|Traces| JAEGER
    POD_AZ1 -->|Logs| FLUENTD
    FLUENTD -->|Store| TIMESCALE["TimescaleDB"]

    CNIE_FEED -->|Sync| POD_AZ1
    SANCTIONS_FEED -->|Sync| POD_AZ1
    FX_FEED -->|Stream| POD_AZ2
```

---

## DIAGRAM 16: Security Boundaries & Encryption

```mermaid
graph TB
    subgraph PUBLIC["Public Internet (Unencrypted)"]
        USER_BROWSER["User Browser"]
        MERCHANT_POS["Merchant POS"]
    end

    subgraph TLS_BOUNDARY["TLS Encryption Boundary"]
        HTTPS["All Traffic Encrypted<br/>(TLS 1.3, AEAD-AES-256-GCM)"]
        
        subgraph K8S_CLUSTER["K8s Cluster (Private VPC)"]
            PODS["Service Pods<br/>(All Microservices)"]
        end
    end

    subgraph HSM_ZONE["HSM Zone (Air-Gapped)"]
        ROOT_CA["BAM Root CA<br/>(FIPS 140-3 Level 4)"]
        ROOT_KEY["Root Private Key<br/>(5-of-7 Custody)"]
    end

    subgraph INTERMEDIATE_CA["Intermediate CA Zone (FIPS 140-2 Level 2)"]
        BANK_CAS["Bank CAs<br/>(per PSP)"]
    end

    subgraph VAULT["Secret Management (HashiCorp Vault)"]
        DB_CREDS["Database Credentials"]
        API_KEYS["API Keys"]
        ENCRYPTION_KEYS["Encryption Keys"]
        CERT_CHAIN["Certificate Chains"]
    end

    subgraph DATABASES["Encrypted Databases"]
        PG_ENCRYPTED["PostgreSQL (Encrypted)<br/>- PGCrypto for PII<br/>- AES-256-GCM"]
        REDIS_ENCRYPTED["Redis (Encrypted)<br/>- TLS + AEAD<br/>- Volatile (no long-term)"]
        FABRIC_LEDGER["Fabric Ledger<br/>- Blockchain-native<br/>- Tamper-evident"]
    end

    subgraph SE["Secure Element (Hardware)"]
        SE_KEY["Private Key (Non-extractable)"]
        SE_CACHE["Cached Data<br/>(Sandboxed)"]
    end

    USER_BROWSER -->|Untrusted| HTTPS
    MERCHANT_POS -->|Untrusted| HTTPS
    HTTPS -->|Decrypt| PODS
    PODS -->|Trusted| TLS_BOUNDARY

    PODS -->|Request Signing Key| ROOT_CA
    ROOT_CA -->|Guarded by| ROOT_KEY
    ROOT_CA -->|Issue Cert| BANK_CAS
    BANK_CAS -->|Distribute| PODS

    PODS -->|Request Secret| VAULT
    VAULT -->|Return Encrypted| DB_CREDS
    VAULT -->|Return Encrypted| ENCRYPTION_KEYS
    DB_CREDS -->|Decrypt| PG_ENCRYPTED
    ENCRYPTION_KEYS -->|Decrypt| REDIS_ENCRYPTED

    PODS -->|Ledger Tx| FABRIC_LEDGER
    FABRIC_LEDGER -->|Cryptographic Proof| PODS

    SE_KEY -->|Isolated| SE
    SE_CACHE -->|Sandboxed| SE
    PODS -.->|Cannot Access| SE_KEY

    style HSM_ZONE fill:#ffcccc,stroke:#cc0000,stroke-width:3px
    style SE fill:#ccffcc,stroke:#00cc00,stroke-width:3px
    style VAULT fill:#ccccff,stroke:#0000cc,stroke-width:2px
```

---

## DIAGRAM 17: Transaction Flow - Offline Payment

```mermaid
graph TB
    subgraph OFFLINE_SETUP["Offline Setup (Last Online Sync)"]
        USER_ONLINE["User Online<br/>(at home/cafe)"]
        SYNC_DL["Download:<br/>- Merkle Root<br/>- Spending Limit<br/>- Sanctions List<br/>- Public Certs"]
        SE_CACHE["SE Caches:<br/>- Private Key<br/>- CNIE Pubkey<br/>- Bloom Filter<br/>- Spending Counter"]
    end

    subgraph OFFLINE_PAYMENT["Offline Payment (No Connectivity)"]
        OFFLINE_TX["User Initiates Payment<br/>- Amount: MAD 500<br/>- Recipient: Merchant"]
        CNIE_VERIFY["SE Verifies:<br/>- CNIE is valid<br/>- Not Expired<br/>- Not Revoked"]
        AMOUNT_CHECK["SE Checks:<br/>- Amount <= MAD 500<br/>- Daily Total <= MAD 500<br/>- Offline Days < 72"]
        SIGN_TX["SE Signs Transaction<br/>- Token Hash<br/>- Amount<br/>- Nonce<br/>- Timestamp<br/>- (Ed25519 Signature)"]
        MARK_SPENT["SE Marks Token Spent<br/>(Hardware Enforced)<br/>- Physical Barrier<br/>to Re-spending"]
    end

    subgraph PAYMENT_CHANNEL["NFC Payment Channel"]
        NFC_TX["NFC Broadcast<br/>(ISO 18092)<br/>- Token Hash<br/>- Amount<br/>- Signature<br/>- Nonce"]
    end

    subgraph RECEIVER["Receiver (Merchant)"]
        RECV_VERIFY["Merchant SE Verifies:<br/>- Signature (Ed25519)<br/>- Amount (< limit)<br/>- Nonce (not replayed)"]
        RECV_BLOOM["Merchant SE Checks<br/>Bloom Filter:<br/>- Has this token been spent?<br/>- False positives may flag honest tx"]
        RECV_ACCEPT["SE Marks Received:<br/>- Token in Received Cache<br/>- Tx in Local Log<br/>- User Gets Receipt<br/>(Can Print QR)"]
    end

    subgraph RECONCILIATION["Later: Online Reconciliation"]
        USER_SYNC["User Comes Online<br/>(next day)"]
        UPLOAD_LOG["Wallet Uploads:<br/>- Offline Tx Log<br/>- Spent Tokens<br/>- Nonces"]
        SERVER_VERIFY["Settlement Server:<br/>- Checks Merkle Proof<br/>- Detects Double-Spend<br/>(if any)"]
        FINALIZE["Settlement Finalizes:<br/>- Ledger Updated<br/>- Balance Confirms<br/>- Tx is Immutable"]
    end

    subgraph DISPUTE_RESOLUTION["Dispute (if Double-Spend)"]
        DOUBLESPEND["Double-Spend Detected:<br/>- Two submissions<br/>of same token"]
        ANALYZE["BAM Analyzes:<br/>- Which submission first?<br/>- Timestamp comparison<br/>- Merkle proof order"]
        WINNER["First Submission Wins<br/>(Based on Merkle<br/>Block Height)"]
        LOSER["Second Tx Rejected:<br/>- Receiver's bank<br/>reverses transfer<br/>- User gets alert"]
    end

    USER_ONLINE -->|Automatic| SYNC_DL
    SYNC_DL -->|Cache Locally| SE_CACHE
    SE_CACHE -->|Ready for Offline| OFFLINE_TX

    OFFLINE_TX -->|SE Check| CNIE_VERIFY
    CNIE_VERIFY -->|All Valid| AMOUNT_CHECK
    AMOUNT_CHECK -->|Within Limit| SIGN_TX
    SIGN_TX -->|Hardware Op| MARK_SPENT
    MARK_SPENT -->|Token Locked| NFC_TX

    NFC_TX -->|ISO 18092| RECV_VERIFY
    RECV_VERIFY -->|Sig OK| RECV_BLOOM
    RECV_BLOOM -->|Not in Bloom| RECV_ACCEPT
    RECV_ACCEPT -->|Local Finality| RECV_ACCEPT

    RECV_ACCEPT -->|Offline Complete| RECONCILIATION
    USER_SYNC -->|Connect| UPLOAD_LOG
    UPLOAD_LOG -->|Cryptographic Proofs| SERVER_VERIFY
    SERVER_VERIFY -->|Merkle Verification| FINALIZE
    FINALIZE -->|T+1 Finality| FINALIZE

    SERVER_VERIFY -->|Clash Detected| DOUBLESPEND
    DOUBLESPEND -->|Investigate| ANALYZE
    ANALYZE -->|Determine Winner| WINNER
    WINNER -->|Reject Loser| LOSER
    LOSER -->|Reverse Transfer| RECV_ACCEPT

    style OFFLINE_SETUP fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style OFFLINE_PAYMENT fill:#fff9c4,stroke:#f57f17,stroke-width:3px
    style PAYMENT_CHANNEL fill:#ffccbc,stroke:#d84315,stroke-width:2px
    style RECONCILIATION fill:#bbdefb,stroke:#1565c0,stroke-width:2px
```

---

## Summary Table: Services, APIs, & Databases

| **Layer** | **Service** | **API Type** | **Database** | **Message Queue** |
|-----------|-----------|-----------|-----------|-----------|
| **1: Monetary** | Issuance Engine | gRPC | Fabric Ledger + PG | Kafka: supply-events |
| **1: Monetary** | Policy Engine | gRPC (Internal) | Git Repo + TimescaleDB | Kafka: policy-updates |
| **2: Settlement** | Transaction Validator | gRPC | Redis (Cache) | Kafka: settlement-events |
| **2: Settlement** | Consensus Engine | N/A (Network) | Fabric Ledger | Kafka: consensus-blocks |
| **2: Settlement** | Ledger Manager | gRPC | Fabric + PG | Kafka: ledger-updates |
| **3: Compliance** | Sanctions Engine | gRPC | Bloom Filter Cache + PG | Kafka: aml-alerts |
| **3: Compliance** | AML Detector | gRPC | ML State + PG | Kafka: aml-escalations |
| **3: Compliance** | SAR Generator | Webhook | SAR Database | Kafka: sar-filed |
| **4: Privacy** | Blind Signature Engine | gRPC | Token Store + Redis | N/A |
| **4: Privacy** | ZKP Engine | gRPC | ZKP Circuits + Redis | N/A |
| **5: Identity** | CNIE Engine | gRPC | CNIE Hash + PG | Kafka: kyc-verifications |
| **5: Identity** | KYC Tier | gRPC | PG (Tier Assignments) | Kafka: tier-changes |
| **6: Distribution** | Retail Wallet API | REST + gRPC | Redis (Sessions) + PG | Kafka: wallet-events |
| **6: Distribution** | Wholesale Interface | ISO 20022 + gRPC | PG (Omnibus) | Kafka: bank-settlements |
| **6: Distribution** | Open API | REST + OAuth | API Keys (Vault) | N/A |
| **7: Offline** | Sync Engine | REST (Upload) + gRPC | Offline Log + PG | Kafka: offline-sync-events |
| **7: Offline** | Double-Spend Detector | gRPC | Bloom Filter + PG | Kafka: doublespend-alerts |
| **8: Cross-Border** | mBridge Connector | mBridge Protocol | PG (Cross-Border Log) | Kafka: crossborder-tx |
| **8: Cross-Border** | Capital Controls | gRPC | PG (Monthly Counters) | Kafka: capctrl-violations |

---

## API Endpoint Examples

### Wallet Service (REST)
```
POST   /api/v1/wallet/create
GET    /api/v1/wallet/{wallet_id}/balance
POST   /api/v1/wallet/{wallet_id}/transfer
GET    /api/v1/wallet/{wallet_id}/history
POST   /api/v1/wallet/{wallet_id}/sync-offline
```

### Settlement Service (gRPC)
```
service Settlement {
  rpc Transfer(TransferRequest) returns (TransferResponse);
  rpc GetBalance(BalanceRequest) returns (BalanceResponse);
  rpc GetFinality(TxHashRequest) returns (FinalityReceipt);
}
```

### Compliance Service (gRPC)
```
service Compliance {
  rpc ScreenTransaction(ScreenRequest) returns (ScreenResponse);
  rpc GenerateSAR(SARRequest) returns (SARResponse);
  rpc CheckSanctions(NameRequest) returns (SanctionsResult);
}
```

### Identity Service (gRPC)
```
service Identity {
  rpc VerifyCNIE(CNIEVerificationRequest) returns (KYCResult);
  rpc IssueDID(DIDRequest) returns (DIDCredential);
  rpc GetVerifiableCredential(CredentialRequest) returns (VC);
}
```

### Cross-Border Service (mBridge Protocol)
```
POST   /mbridge/v1/initiate-transfer
GET    /mbridge/v1/exchange-rate/{corridor}
POST   /mbridge/v1/lock-rate
POST   /icebreaker/v1/query-other-cbdc
```

---

