# Digital Dirham: Implementation Roadmap

**Status**: Planning Phase
**Project Type**: Personal study planning draft

> **Disclaimer**  
> This roadmap is an independent personal-study scenario derived from publicly available CBDC material.  
> It is not affiliated with, endorsed by, or owned by Bank Al-Maghrib, and all planning numbers below are illustrative hypotheses unless separately sourced.

## Executive Summary

24-month deployment plan:
- **Phase 1** (Q1-Q2 2025): Foundation & Pilots (\$2-3M)
- **Phase 2** (Q3-Q4 2025): Agent Network & Offline (\$4-5M)
- **Phase 3** (Q1-Q2 2026): Diaspora Remittances (\$3-4M)
- **Phase 4** (Q3-Q4 2026): Retail Expansion (\$2-3M)
- **Phase 5** (2027+): Financial Services (\$2-3M/year)

**Total 3-year**: \$13-18M (illustrative planning estimate)
**ROI (hypothesis)**: 15-30x (illustrative \$240-425M annual benefit estimate by Year 3)

## Critical Path Items (Do NOW)

### Week 1: Immediate Actions
- [ ] Governor approval (non-negotiable)
- [ ] Steering committee formed (weekly meetings)
- [ ] Parliament engagement begun
- [ ] Hiring postings live
- [ ] Cloud provider selected
- [ ] SE hardware ordered (lead time: 8-12 weeks)

### Week 2-3: Planning
- [ ] Detailed project plan completed
- [ ] Budget approved (\$2-3M Phase 1)
- [ ] CBDC legislation drafted
- [ ] Team structure defined

### Week 4-6: Execution
- [ ] 50% engineering team hired
- [ ] Infrastructure provisioning begun
- [ ] Development starts (Rust + Go)
- [ ] SE firmware development starts

## Detailed Timeline

### May 2025 (THIS MONTH)

**Steering Committee**
- [ ] Form committee (Governor, CTO, Compliance, Legal, Ops)
- [ ] First meeting held
- [ ] Weekly schedule established
- [ ] Decision authorities defined

**Governance & Legislation**
- [ ] Brief Parliament Finance Committee
- [ ] Engage legal counsel
- [ ] Draft CBDC law
- [ ] Timeline: Parliament approval by Q3 2025

**Team Assembly**
- [ ] Open 12-15 positions
- [ ] Identify lead architect (Rust expert)
- [ ] Contract with security auditors
- [ ] Target: 50% hired by June 30

**Infrastructure**
- [ ] Evaluate AWS vs. Azure
- [ ] Select 3-AZ provider
- [ ] Sign infrastructure contract
- [ ] Order Thales Luna HSM (2 units)

**Hardware Procurement**
- [ ] Contact Thales, G+D, Gemalto
- [ ] Order 100K SIM cards with applet
- [ ] Order 10K eSIM test units
- [ ] Lead time: 8-12 weeks (critical path)

**Budget & Approvals**
- [ ] \$2-3M Phase 1 funding approved
- [ ] Project charter signed by Governor
- [ ] Quarterly reporting established

### June 2025

**Team Hiring** (Target: 50% by end of month)
- [ ] Lead Architect: 1 person
- [ ] Rust Engineers: 3 people (50% may delay)
- [ ] Go Engineers: 2 people
- [ ] DevOps Engineer: 1 person
- [ ] Security Engineers: 2 people (1 of 2 hired)
- [ ] QA Engineers: 1 of 2 hired

**Technical Development Starts**
- [ ] Rust: Architecture phase (Week 1-2)
- [ ] Go: API design (Week 1-2)
- [ ] SE: Firmware specification (Week 1-2)

**Infrastructure Setup**
- [ ] Cloud account provisioning
- [ ] Initial network configuration
- [ ] HSM received, configured
- [ ] Monitoring systems (Prometheus)

**Legislation Progress**
- [ ] Draft law completed (sent to Parliament)
- [ ] Finance Committee review begun
- [ ] First reading scheduled

### July 2025

**Full Team Assembly** (Target: 100% by end)
- [ ] All 15 engineering positions filled
- [ ] Onboarding began (training, access setup)
- [ ] Team productivity ramps up

**Core Development**
- [ ] Rust: Settlement logic (Week 1-3)
- [ ] Go: API endpoints (Week 1-3)
- [ ] SE: Firmware development (Week 1-2)
- [ ] First builds available

**Infrastructure**
- [ ] PostgreSQL cluster provisioning begins
- [ ] Kubernetes cluster setup
- [ ] Cloud networking configured
- [ ] First staging environment ready

**Legislation**
- [ ] Second reading (Parliament)
- [ ] Committee amendments addressed
- [ ] Target: Passage by September 30

### August 2025

**Development Milestone**
- [ ] Rust: 80% feature-complete
- [ ] Go: 70% endpoints functional
- [ ] SE: Firmware 50% complete
- [ ] PostgreSQL cluster 90% ready
- [ ] Integration testing begins

**Security & Compliance**
- [ ] Security audit planning (external firm)
- [ ] Privacy impact assessment (PPIA)
- [ ] Threat modeling workshop

**Pilot Preparation**
- [ ] Select 10K pilot users (government employees)
- [ ] Training materials drafted
- [ ] Support infrastructure designed
- [ ] Comms plan drafted

### September 2025

**Code Complete (Feature Freeze)**
- [ ] Rust: 100% feature-complete
- [ ] Go: 100% endpoints ready
- [ ] SE: Firmware complete
- [ ] All systems integrated
- [ ] Feature freeze (bugfixes only)

**Testing Phase Begins**
- [ ] Unit tests running
- [ ] Integration tests ongoing
- [ ] Performance testing starts
- [ ] Security testing begins

**Legislation Passed**
- [ ] Parliament votes: CBDC Law PASSED ✓
- [ ] Publication in Official Gazette
- [ ] Regulatory rules drafted

**SE Hardware Arrives**
- [ ] 100K SIM cards with applet delivered
- [ ] Initial batch testing (1K units)
- [ ] Firmware loading verified
- [ ] Security audit complete

### October 2025

**Hardening Phase**
- [ ] All bugs from testing fixed
- [ ] Performance benchmarks met
- [ ] Security audit passed
- [ ] Production hardening completed

**Pilot Preparation**
- [ ] 10K pilot users recruited (salary email)
- [ ] Training videos completed
- [ ] Support team trained
- [ ] Pilot environment ready
- [ ] Soft-launch checklist complete

**Go/No-Go Decision**
- [ ] Governor final review
- [ ] All stakeholders sign off
- [ ] Launch approval given

### November 2025

**Pilot Environment Ready**
- [ ] All systems staged and tested
- [ ] Monitoring dashboards live
- [ ] Alerting configured
- [ ] On-call rotation established

**Final Testing & Training**
- [ ] Disaster recovery testing
- [ ] Failover testing (Primary → Standby)
- [ ] Operations team final training
- [ ] 24/7 support hotline operational

### December 2025

**Pre-Launch Checklist**
- [ ] All systems performing well
- [ ] SEs loaded with test balances
- [ ] Pilot users notified
- [ ] Media embargo (announcement scheduled Jan 15)

**Contingency Prep**
- [ ] Rollback procedure tested
- [ ] Emergency contact list established
- [ ] Crisis communication plan ready

### January 2026 – PILOT LAUNCH 🚀

**Soft Launch (January 15)**
- [ ] Government employees receive 50% salary in Dirham
- [ ] System monitoring intense (24/7)
- [ ] Early issues resolved quickly
- [ ] Target: 5K+ active users

**Full Rollout (January 20)**
- [ ] 100% of government salary in Dirham
- [ ] Scale to 8K+ users
- [ ] Daily monitoring of:
  - Settlement engine TPS
  - Latency (P99)
  - Error rates
  - Double-spend attempts

**Success Metrics (First 30 Days)**
- ✓ 8K+ users activated
- ✓ 0 security incidents
- ✓ 99.99% uptime
- ✓ <1 second settlement (online)
- ✓ <0.1% failed transactions
- ✓ Illustrative low double-spend-rate target

### February-June 2026 (AGENT NETWORK PHASE)

**Recruit Agents (500 initial)**
- [ ] Urban agents: 200 (pharmacies, shops)
- [ ] Rural agents: 300 (cooperatives, post offices)
- [ ] Compensation model: 0.25% per transaction
- [ ] Training: 2-day workshop per agent

**Scale Users**
- [ ] Feb: 50K users
- [ ] Mar: 200K users
- [ ] Apr: 500K users
- [ ] May: 800K users
- [ ] Jun: 1M users

**Offline Capability**
- [ ] NFC D2D payments live (Feb)
- [ ] USSD menu for feature phones (Mar)
- [ ] Agent batch reconciliation (Apr)
- [ ] Pilot: 20% of volume offline (Jun)

**Agent Metrics**
- ✓ 500 agents actively using
- ✓ 100K+ daily transactions
- ✓ Agent satisfaction: 80%+
- ✓ Revenue per agent: 500-1000 MAD/month

### July-December 2026 (DIASPORA PHASE)

**mBridge Integration (Jul-Aug)**
- [ ] Finalize Dirham node setup
- [ ] Test Dirham ↔ mBridge connectivity
- [ ] Regulatory clearances
- [ ] Go-live: August 2026

**Diaspora Corridor Launch (Sep)**
- [ ] mBridge to 3-4 countries (UAE, Saudi, France via mBridge)
- [ ] Sub-10-minute settlement
- [ ] <0.5% cost (vs. 5.5% Western Union)
- [ ] Target: 100K diaspora users registered by Dec

**Diaspora Metrics (Q4 2026)**
- ✓ 100K+ diaspora users
- ✓ 50K+ monthly remittance transactions
- ✓ \$50M+ monthly volume
- ✓ 2% market share (\$140M of annual \$7B)

**Retail Expansion (Q4 2026)**
- [ ] QR code payment support (static + dynamic)
- [ ] First 5K merchants onboarded
- [ ] Merchant training program
- [ ] Target: 50K merchants by end 2026

### 2027+ (SCALE & FINALIZE)

**Retail Phase (Q1-Q2 2027)**
- [ ] 50K+ merchants actively using
- [ ] 10% of daily payment volume
- [ ] 1M+ transactions/day (all types)
- [ ] 5M+ total wallets

**Financial Services (Q2-Q4 2027)**
- [ ] Microfinance partnerships
- [ ] Loan origination (via SE collateral)
- [ ] Savings products (interest-bearing)
- [ ] Insurance integration

**Final Metrics (Year 3)**
- ✓ 15M+ wallets (40% of population)
- ✓ \$500M+ daily transaction volume
- ✓ 25% diaspora remittance market share
- ✓ Regional recognition (African model)
- ✓ Post-quantum crypto migration begun

## Budget Breakdown

### Phase 1 (Q1-Q2 2025): \$2-3M
```
Engineering team (6 months)     : \$900K
├─ 6 engineers × \$150K/year
├─ + benefits + taxes
└─ Hiring + onboarding

Infrastructure (AWS/Azure)      : \$400K
├─ Cloud resources (\$50K/month × 8)
├─ HSM (\$50K)
└─ Networking + data

SE firmware & hardware          : \$150K
├─ 2 engineers × \$75K
├─ Testing hardware
└─ Security audit

Testing & QA                    : \$300K
├─ 2 QA engineers
├─ Testing tools
└─ Penetration testing

Management & overhead           : \$250K (contingency 10%)
```

### Phase 2 (Q3-Q4 2025): \$4-5M
```
Mobile app (iOS + Android)      : \$800K
USSD gateway & integration      : \$300K
Agent training & tools          : \$500K
Merchant POS                    : \$200K
Operations & support            : \$1.5M
Contingency (15%)               : \$700K
```

### Phase 3 (Q1-Q2 2026): \$3-4M
```
mBridge integration             : \$600K
International partnerships      : \$400K
Diaspora marketing              : \$300K
Compliance & regulatory         : \$200K
Contingency (20%)               : \$600K
```

### Total 3-Year: \$13-18M

## Success Metrics (Quarterly Tracking)

### Q1 2025
- [ ] Funding approved
- [ ] 50% team hired
- [ ] Development started
- [ ] Infrastructure provisioned

### Q2 2025
- [ ] 100% team hired
- [ ] Code feature-complete
- [ ] Legislation passed
- [ ] SE hardware arrives

### Q3 2025
- [ ] All systems tested
- [ ] Pilot environment ready
- [ ] 10K users recruited
- [ ] Training complete

### Q4 2025
- [ ] Pilot launches (10K government employees)
- [ ] 8K+ daily active users
- [ ] 0 security incidents
- [ ] 99.99% uptime

### Q1 2026
- [ ] 500K users
- [ ] 500+ agents operational
- [ ] NFC offline transactions live
- [ ] Agent network scaling

### Q2 2026
- [ ] 1M+ users
- [ ] Diaspora corridor planning
- [ ] Offline transactions: 20% of volume
- [ ] mBridge integration testing

### Q3 2026
- [ ] mBridge integration live
- [ ] Diaspora corridor launches
- [ ] 100K diaspora users
- [ ] Retail expansion begins

### Q4 2026
- [ ] \$50M+ diaspora monthly volume
- [ ] 5K merchants onboarded
- [ ] 5M+ total wallets
- [ ] Retail phase targets met

### 2027
- [ ] 15M+ wallets (40% of population)
- [ ] 25% diaspora market share
- [ ] Financial services integrated
- [ ] Regional leadership achieved

## Risk Management

### High-Priority Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Low adoption (eNaira repeat) | Medium | High | G2P use case, agent incentives |
| Technical delays | Low | High | Proven tech only; avoid custom |
| Regulatory delays | Medium | Medium | Parliamentary engagement early |
| Cybersecurity incident | Low | Critical | Multi-layer defense; insurance |
| Diaspora political pressure | Low | Medium | Competitive pricing; transparency |

### Contingency Plans

**If Governor delays approval:**
- Pivot to payment system modernization
- Reduce scope to pilot only
- Timeline: +6-12 months

**If legislation blocked:**
- Work with Parliament on amendments
- Seek compromise on privacy terms
- Alternative: Regulatory sandbox

**If SE hardware delayed:**
- Use eSIM-only for initial pilot
- Extend timeline by 4-8 weeks
- Source from alternative vendors

---

**Document Version**: 2.0
**Last Updated**: May 2025
**Project Type**: Personal study planning draft
