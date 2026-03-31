# AI Robo-Advisor Startup Plan

**Date:** 2026-04-01
**Status:** Approved
**Author:** Generated via Claude Code brainstorming
**Last Updated:** 2026-04-01

---

## 1. Concept & Vision

**Tên dự kiến:** InvestAI / WealthMind / *(open for naming)*

**One-liner:** AI-powered Robo-Advisor for everyone — smarter than traditional advisors, cheaper than human managers.

**Mission:** Democratize wealth management bằng AI, giúp người bình thường tiếp cận investment strategies từng chỉ dành cho wealthy investors.

**Giá trị khác biệt:**
- AI insights tốt hơn traditional rules-based robo-advisors
- Phí thấp hơn (freemium model)
- Multi-market (stocks, crypto, savings)
- Personalized financial education

---

## 2. Market Opportunity

**TAM:** $500B+ global robo-advisory market (2026)

**SAM:** $50B+ addressable market (retail investors in US, SEA, EU)

**SOM:** $5B+ initial focus (mass affluent + emerging retail investors)

**Key trends:**
- 60% Gen Z prefer digital-first financial services
- AI adoption in finance growing 30% YoY
- Micro-investing trend (spare change investing)
- Regulatory sandbox opening in SEA (Singapore, Vietnam)

---

## 3. Learning Framework (12-18 tháng)

### 3.1 Phase 1: Market Research (Months 1-3)

**Mục tiêu:** Hiểu thị trường, competitors, regulation

| Week | Activity | Deliverable |
|---|---|---|
| 1-2 | Competitor audit | List 10 competitors, analyze 5 deeply |
| 3-4 | Regulation map | Matrix of regulations by market |
| 5-6 | User interviews | 10-20 interviews, pain points identified |
| 7-8 | Market sizing | TAM/SAM/SOM validation |
| 9-10 | Pricing analysis | Competitive pricing model |
| 11-12 | SWOT analysis | Internal capabilities vs market needs |
| 13 | Phase 1 Report | Comprehensive market research doc |

**Output:** `docs/research/phase1-market-research.md`

### 3.2 Phase 2: Technical Learning (Months 4-6)

**Mục tiêu:** Hiểu technical landscape, build technical foundation

| Week | Topic | Deliverable |
|---|---|---|
| 14-15 | Portfolio theory | Summary: MPT, risk-parity, factor investing |
| 16-17 | AI/ML for finance | POC: simple portfolio recommendation model |
| 18-19 | Data infrastructure | Architecture design for data pipeline |
| 20-21 | Security & compliance | Tech requirements checklist |
| 22-23 | Prototype tech stack | Chosen stack + reasoning |
| 24 | Phase 2 Report | Technical feasibility assessment |

**Output:** `docs/research/phase2-technical-feasibility.md`

### 3.3 Phase 3: Business Model Validation (Months 7-9)

**Mục tiêu:** Validate business model và go-to-market strategy

| Week | Topic | Deliverable |
|---|---|---|
| 25-26 | Unit economics | CAC, LTV, churn model |
| 27-28 | Go-to-market | Launch strategy per market |
| 29-30 | Partnership strategy | List 10 potential BaaS partners |
| 31-32 | Financial modeling | 5-year projections |
| 33-34 | Regulatory prep | License pathway for each market |
| 35-36 | Phase 3 Report | Business model validation doc |

**Output:** `docs/research/phase3-business-validation.md`

### 3.4 Phase 4: Build Prototype (Months 10-12)

**Mục tiêu:** Có working MVP để validate ý tưởng

| Week | Feature | Status |
|---|---|---|
| 37-38 | Auth + Onboarding | Complete |
| 39-40 | Portfolio Dashboard | MVP |
| 41-42 | Risk Assessment Quiz | MVP |
| 43-44 | AI Recommendations | MVP |
| 45-46 | Paper Trading | MVP |
| 47-48 | Basic Analytics | MVP |
| 49 | Internal Beta | 10 beta users |

**Output:** `app/` — working prototype codebase

### 3.5 Phase 5: Validate & Iterate (Months 13-18)

**Mục tiêu:** Gather feedback, iterate, prepare cho next steps

| Week | Activity | Metric |
|---|---|---|
| 50-52 | User feedback loop | NPS > 40 |
| 53-56 | Partnership outreach | 3 LOIs signed |
| 57-60 | Regulatory consultation | Pre-application meetings |
| 61-64 | Iterate product | Retention > 60% |
| 65-68 | Investment thesis | Finalize market focus |
| 72 | Decision Point | Raise seed / Apply license / Pivot |

---

## 4. Technical Architecture

### 4.1 Tech Stack

```
Frontend:        Next.js 16 + shadcn/ui + Geist
Backend:        FastAPI / Node.js + Vercel Functions
AI:             Claude API (AI SDK v6) + AI Gateway
Data:           Neon Postgres + Upstash Redis + Vercel Blob
Market Data:    Alpha Vantage / Polygon.io / Yahoo Finance API
Auth:           Clerk (multi-market ready)
Infrastructure: Vercel (Edge, Fluid Compute)
```

**Lựa chọn thay thế:**
- Frontend: React Native (mobile-first) hoặc Flutter (cross-platform)
- Backend: Python FastAPI (better for AI/ML), Node.js (faster iteration)
- AI: Vercel AI SDK với AI Gateway (recommended), direct provider SDKs

### 4.2 MVP Features (Priority Order)

1. **Risk Assessment Quiz** — Onboarding flow: questions → risk tolerance → recommended allocation
2. **Portfolio Dashboard** — View holdings, performance chart, asset allocation pie chart
3. **AI Recommendations** — "Based on your profile, consider X" (powered by Claude)
4. **Paper Trading** — Simulate trades without real money
5. **Basic Analytics** — Returns, volatility, Sharpe ratio, drawdown

### 4.3 Data Architecture

```
Market Data APIs
       ↓
  Data Pipeline (Kafka / Redis queues)
       ↓
  Neon Postgres (structured data)
       ↓
  Analytics Layer
       ↓
  AI Recommendations Engine
       ↓
  User Dashboard
```

---

## 5. Revenue Model

### 5.1 Freemium Tiers

| Tier | Price | Features |
|---|---|---|
| **Free** | $0 | Portfolio tracking, basic analytics, 1 watchlist |
| **Pro** | $5/tháng | AI recommendations, unlimited watchlists, advanced analytics |
| **Premium** | $15/tháng | + Tax optimization, + Priority support, + Multi-account |

### 5.2 Long-term Revenue

Khi đã có license:
- **AUM-based fee:** 0.5% trên tổng tài sản quản lý
- **Transaction fee:** $0.50-$2 per trade
- **Premium subscriptions:** $15-50/tháng cho advanced features

### 5.3 Unit Economics Target

| Metric | Target |
|---|---|
| CAC (Customer Acquisition Cost) | < $20 |
| LTV (Lifetime Value) | > $200 |
| LTV:CAC Ratio | > 10:1 |
| Monthly Churn | < 5% |
| Break-even | 500 paid users |

---

## 6. Competitive Landscape

### 6.1 Direct Competitors

| Competitor | Strengths | Weaknesses | Pricing |
|---|---|---|---|
| **Betterment** | Largest, most established | Rules-based AI, high fees | 0.25% AUM |
| **Wealthfront** | Strong technical | US only | 0.25% AUM |
| **Acorns** | Great UX, micro-investing | Limited features | $3-5/tháng |
| **M1 Finance** | No fees, good UX | Limited AI | Free |
| **Roboadvisor.vn** | Vietnam market | Limited features | ~1% AUM |

### 6.2 Our Positioning

- **Khác biệt chính:** AI thông minh hơn (Claude-powered) + phí thấp hơn
- **Focus market đầu tiên:** SEA (Singapore + Vietnam) — underserved market
- **Go-to-market:** Content marketing + referrals (low CAC)

---

## 7. Regulatory Strategy (Hybrid Approach)

### Phase 1: Aggregator/Insights Only (No License Required)

- Portfolio tracking và analytics (không quản lý tiền)
- AI recommendations (khuyến nghị, không phải advice có trách nhiệm)
- Paper trading

### Phase 2: Partner with Licensed Entities

- Làm việc với broker-dealer hoặc bank có license
- Revenue share model
- Leverage existing compliance infrastructure

### Phase 3: Apply for License (When Scale Justifies)

- US: SEC registration as RIA
- Singapore: MAS license
- Vietnam: VFSC license

### Regulation by Market

| Market | License Required | Timeline | Cost |
|---|---|---|---|
| Vietnam | VFSC (pending sandbox) | 6-12 months | $10K-50K |
| Singapore | MAS CMS License | 12-18 months | $50K-100K |
| US | SEC RIA | 6-12 months | $10K-25K |
| EU | MiCA passport | 18-24 months | $100K+ |

---

## 8. Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Regulation changes | High | High | Start as aggregator-only, monitor sandbox programs |
| AI hallucinations | Medium | High | Human-in-the-loop for financial advice, clear disclaimers |
| Competition | High | Medium | Focus on underserved SEA market first |
| User trust | Medium | High | Transparent about limitations, show track record |
| Data quality | Medium | Medium | Use reputable data providers, validate with multiple sources |
| Market volatility | High | Medium | Diversified portfolio recommendations, clear risk disclosures |

---

## 9. Key Deliverables & Timeline

| When | Deliverable | File/Location |
|---|---|---|
| Month 3 | Market Research Report | `docs/research/phase1-market-research.md` |
| Month 6 | Technical Feasibility Report | `docs/research/phase2-technical-feasibility.md` |
| Month 9 | Business Model Validation Report | `docs/research/phase3-business-validation.md` |
| Month 12 | Working MVP | `app/` directory |
| Month 18 | Decision Point | `docs/decisions/raise-seed-or-pivot.md` |

---

## 10. Success Metrics

### Product Metrics

| Metric | Month 12 Target | Month 18 Target |
|---|---|---|
| Beta Users | 10 | 100 |
| DAU/MAU | 30% | 40% |
| NPS | > 30 | > 50 |
| Retention (Month 1) | 60% | 70% |

### Business Metrics

| Metric | Month 18 Target |
|---|---|
| Paid Users | 50 |
| MRR | $500 |
| CAC | < $30 |
| LTV | > $150 |

### Learning Metrics

| Metric | Description |
|---|---|
| Market Validation | Confirmed underserved segment with pain points |
| Technical Feasibility | POC working, AI model accurarte > 60% |
| Business Model | Unit economics work on paper |
| Regulatory Path | Clear license pathway identified |

---

## 11. Next Steps (After This Plan)

1. **Immediate (Week 1-2):** Start competitor audit — analyze 10 competitors
2. **Week 3-4:** Begin user interviews — target 10-20 potential users
3. **Week 5-8:** Continue market research — build comprehensive market map
4. **Week 9-12:** Compile Phase 1 Report
5. **Month 4+:** Begin technical learning — portfolio theory, AI/ML

---

## 12. Open Questions

- [ ] Tên sản phẩm: InvestAI / WealthMind / ?
- [ ] Focus market đầu tiên: Vietnam vs Singapore vs US?
- [ ] Founding team: có ai join chưa? Cần什么人?
- [ ] Budget: có funding chưa hay cần bootstrap?
- [ ] Technical skills: background về fintech/AI không?

---

## Appendix: Resources

### Competitors to Study

1. Betterment (US) — largest robo-advisor
2. Wealthfront (US) — technical, tax-loss harvesting
3. Acorns (US) — micro-investing, great UX
4. M1 Finance (US) — no fees, self-directed
5. Nutmeg (UK) — UK market leader
6. Roboadvisor.vn (Vietnam) — local market
7. StashAway (Singapore) — SEA market
8. Endowus (Singapore) — institutional focus
9. YNAB (US) — personal finance
10. Personal Capital (US) — wealth management

### Data Sources

- **Market data:** Alpha Vantage, Polygon.io, Yahoo Finance API, CoinGecko
- **Regulatory:** SEC.gov, MAS.gov, VFSC.gov.vn, ESMA.europa.eu
- **Industry reports:** McKinsey Global Banking, Deloitte Fintech
- **User research:** Reddit (r/personalfinance), Quora, interviews

### Learning Resources

- **Portfolio theory:** AQR Capital Management blog, Quantopian lectures
- **AI/ML:** Andrew Ng's Machine Learning course, fast.ai
- **Fintech:** LendIt Fintech conference, Singapore Fintech Festival
- **Regulation:** CFA Institute, FINRA guidance
