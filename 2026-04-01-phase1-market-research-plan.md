# Phase 1: Market Research — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan step-by-step. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete comprehensive competitor audit and market landscape analysis for AI Robo-Advisor startup.

**Architecture:** This plan covers the first two phases of the 18-month learning journey — competitor research and user interviews. Output is a structured research document, not production code. Research is document-based (web fetch, analysis, writing).

**Tech Stack:** Claude Code (web research agent), Notion/Obsidian, Git for doc management.

**Time assumption:** Tasks estimate a 15-day sprint assuming full-time focus on Phase 1. If working part-time alongside other commitments, extend to 6-8 weeks. Interview recruitment should begin by Day 1 — scheduling is typically the longest lead time.

---

## Deliverables

| Week | Deliverable | File | Notes |
|---|---|---|---|
| Week 1 | Competitor Profiles (10 companies) | `docs/research/competitor-profiles.md` | Includes pricing analysis per competitor |
| Week 1 | Competitive Analysis Matrix | `docs/research/competitive-matrix.md` | SWOT section folded into gaps & opportunities |
| Week 2 | Regulation Map by Market | `docs/research/regulation-map.md` | |
| Week 2 | User Interview Synthesis | `docs/research/user-interviews.md` | |
| Week 3 | Market Sizing (TAM/SAM/SOM) | `docs/research/market-sizing.md` | |
| Week 3 | Phase 1 Market Research Report | `docs/research/phase1-market-research.md` | Includes SWOT, pricing analysis synthesis |

---

## Task 1: Competitor Audit — Identify 10 Competitors

**Files:**
- Create: `docs/research/competitor-profiles.md`

- [ ] **Step 1: Identify 10 competitors to research**

Research these 10 companies using web search:

| # | Name | Market | URL |
|---|---|---|---|
| 1 | Betterment | US | betterment.com |
| 2 | Wealthfront | US | wealthfront.com |
| 3 | Acorns | US | acorns.com |
| 4 | M1 Finance | US | m1.com |
| 5 | Nutmeg | UK | nutmeg.com |
| 6 | Roboadvisor.vn | Vietnam | roboadvisor.vn |
| 7 | StashAway | Singapore | stashaway.sg |
| 8 | Endowus | Singapore | endowus.com |
| 9 | YNAB | US | ynab.com |
| 10 | Personal Capital | US | personalcapital.com |

For each competitor, gather:
- **Business model:** Freemium/subscription/AUM fee
- **Target audience:** Retail/gen-z/mass-affluent/wealthy
- **Key features:** AI recommendations, automated rebalancing, tax optimization, crypto
- **Pricing:** Monthly fee or AUM percentage
- **Tech stack hints:** What AI/ML do they use?
- **Strengths:** What do they do well?
- **Weaknesses:** What do customers complain about?
- **Year founded:** Track maturity

- [ ] **Step 2: Deep-dive on top 5 competitors**

Use Claude Code web research (WebFetch) to analyze the 5 most relevant competitors deeply:
1. Betterment — largest US robo-advisor
2. Acorns — best UX, micro-investing model
3. StashAway — leading SEA competitor
4. M1 Finance — no-fee model
5. Roboadvisor.vn — Vietnam market

For each deep-dive, use:
```
Use WebFetch to get:
- Their landing page key messaging
- Their pricing page
- Their features list
- App store reviews (search for common complaints/praise)
- Any press about their AI/ML approach
```

- [ ] **Step 3: Write competitor profiles document**

Format: one section per competitor

```markdown
## [Competitor Name]

**HQ:** [Location]
**Founded:** [Year]
**Funding:** [Amount raised if available]
**Business Model:** [Description]
**Target Audience:** [Demographic]

### Key Features
- [Feature 1]
- [Feature 2]
- ...

### Pricing
- [Free tier details]
- [Paid tier details]

### Strengths
- [What they do well]

### Weaknesses
- [What customers complain about]

### AI/ML Approach
- [How they use AI, if documented]
```

- [ ] **Step 4: Commit**

```bash
git add docs/research/competitor-profiles.md
git commit -m "docs: add competitor profiles - 10 robo-advisor companies"
```

---

## Task 2: Competitive Analysis Matrix

**Files:**
- Create: `docs/research/competitive-matrix.md`

- [ ] **Step 1: Create comparison matrix**

Build a matrix comparing all 10 competitors across key dimensions:

| Dimension | Betterment | Wealthfront | Acorns | ... |
|---|---|---|---|---|
| **Business Model** | AUM fee | AUM fee | Subscription | ... |
| **Target Audience** | Mass affluent | Tech-savvy | Gen Z | ... |
| **Pricing** | 0.25% AUM | 0.25% AUM | $3-5/mo | ... |
| **AI Features** | Automated | Tax-loss | Spare change | ... |
| **Multi-market** | US only | US only | US | ... |
| **Crypto** | No | No | Yes | ... |
| **Mobile App** | ✅ | ✅ | ✅ | ... |
| **Social Features** | No | No | ✅ | ... |
| **Human Advisor** | ❌ | ❌ | ❌ | ... |
| **Tax Optimization** | ✅ | ✅ | ❌ | ... |
| **API Access** | ❌ | ❌ | ❌ | ... |
| **AI Chat** | ❌ | ❌ | ❌ | ... |

- [ ] **Step 2: Identify gaps and opportunities**

At the end of the matrix, add a section:

```markdown
## Gaps & Opportunities

### Underserved Segments
- [Segments that existing competitors don't serve well]

### Feature Gaps
- [Features nobody does well]

### Market Gaps
- [Markets that lack quality robo-advisors]

### AI/ML Gaps
- [Areas where AI could be dramatically better]

### Pricing Gaps
- [Pricing models that aren't common]
```

- [ ] **Step 3: Commit**

```bash
git add docs/research/competitive-matrix.md
git commit -m "docs: add competitive analysis matrix and gaps"
```

---

## Task 3: Regulation Map by Market

**Files:**
- Create: `docs/research/regulation-map.md`

- [ ] **Step 1: Research regulations for each target market**

Research regulatory requirements for offering robo-advisory services in:

| Market | Primary Regulator | Key License | Notes |
|---|---|---|---|
| Vietnam | VFSC | Securities license (pending sandbox) | New fintech sandbox announced 2025 |
| Singapore | MAS | CMS License (Capital Markets Services) | Well-defined sandbox |
| US | SEC | Investment Adviser (RIA) registration | Reg BI, fiduciary duty |
| EU | ESMA/NCA | MiCA / local NCA | PSD2 for payments |
| UK | FCA | FCA authorisation | FCA sandbox available |

For each market, research:
```
Using WebFetch and WebSearch:
- What license/registration is required?
- What activities are regulated?
- What's the application process and timeline?
- What's the estimated cost?
- Is there a sandbox or regulatory accelerator?
- What are the capital requirements?
- What investor protection rules apply?
```

- [ ] **Step 2: Build regulation matrix**

```markdown
## [Market] — [Regulator]

### License Required
- [Type of license]

### Regulated Activities
- [What activities require the license]

### Application Process
- [Steps to apply]

### Timeline
- [How long to get approved]

### Cost
- [Application fees, ongoing compliance costs]

### Capital Requirements
- [Minimum capital if any]

### Sandbox Available?
- [Yes/No + details]

### Key Rules to Comply
- [Consumer protection, reporting, etc.]

### Sources
- [Link to official guidance]
```

- [ ] **Step 3: Identify the easiest market to enter**

Add a conclusion section:

```markdown
## Entry Strategy Recommendation

### Easiest Market to Enter First
[Market] — [Reason]

### Recommended Approach
1. [Step 1: Start as insights/analytics only, no license needed]
2. [Step 2: Partner with licensed entity]
3. [Step 3: Apply for own license when scale justifies]

### Timeline for Each Market
- Vietnam: [X] months (if sandbox available)
- Singapore: [X] months
- US: [X] months
```

- [ ] **Step 4: Commit**

```bash
git add docs/research/regulation-map.md
git commit -m "docs: add regulation map for multi-market entry"
```

---

## Task 4: User Interviews

**Files:**
- Create: `docs/research/user-interviews.md`

- [ ] **Step 1: Define interview guide (5 questions)**

Design a structured interview guide for 10-20 potential users:

```markdown
# User Interview Guide — Robo-Advisor Research

## Screening Questions
1. Do you currently invest? (stocks, crypto, mutual funds, savings)
2. Do you use any investment apps? (Betterment, Acorns, etc.)
3. What's your age range? (18-25 / 26-35 / 36-50 / 50+)

## Core Questions

### Question 1: Investment Behavior
"Tell me about how you currently manage your money and investments."
- Probe: What apps do you use?
- Probe: What do you like/dislike about them?

### Question 2: Pain Points
"What's the most frustrating thing about investing or managing your money?"
- Probe: Have you ever felt lost or overwhelmed?
- Probe: What would make it easier?

### Question 3: Trust in AI
"Would you trust an AI to manage your investment decisions? Why or why not?"
- Probe: What would make you trust it more?
- Probe: Would you trust it more or less than a human advisor?

### Question 4: Pricing
"Would you prefer to pay a monthly fee, a percentage of your investments, or nothing (with ads or upsells)?"
- Probe: What price feels fair for good AI advice?

### Question 5: Dream Feature
"If you could have any feature in an investment app, what would it be?"
- Probe: Why is that important to you?
```

- [ ] **Step 2: Recruit interview participants**

Target: 10-20 people across segments:
- 5 Gen Z (18-25): Students, early career
- 5 Millennials (26-35): Working professionals
- 5 Mass affluent (36-50): Higher income
- 5 Expats/digital nomads: Cross-border investors

Sources:
- Friends and family network
- Reddit communities (r/personalfinance, r/VietNam, r/investing)
- LinkedIn outreach
- Facebook groups (Vietnamese finance communities)

- [ ] **Step 3: Conduct interviews and take notes**

For each interview:
- Get verbal consent to record/anonymize
- Take detailed notes (not full transcripts unless possible)
- Focus on: pain points, trust, willingness to pay, dream features

- [ ] **Step 4: Synthesize findings**

```markdown
# User Interview Synthesis

## Interview Participants
- Total: [X] interviews
- Demographics: [Breakdown]

## Key Themes

### Theme 1: [Name]
**Quote:** "[Representative quote]"
**Frequency:** [X out of Y] mentioned this
**Implication for product:** [What this means for our design]

### Theme 2: [Name]
...

## Pain Points (Ranked by Frequency)
1. [Most common pain point]
2. [Second most common]
3. [Third most common]

## Trust Signals
- [What makes users trust an AI advisor]
- [What breaks trust]

## Willingness to Pay
- [What price point is acceptable]
- [What pricing model is preferred]

## Dream Features (Ranked)
1. [Most requested feature]
2. [Second most requested]

## User Segments
### Segment 1: [Name]
- Characteristics: [Description]
- Biggest pain: [Pain point]
- Best price: [Price point]
- Dream feature: [Feature]

### Segment 2: [Name]
...
```

- [ ] **Step 5: Commit**

```bash
git add docs/research/user-interviews.md
git commit -m "docs: add user interview guide and synthesis"
```

---

## Task 5: Market Sizing

**Files:**
- Create: `docs/research/market-sizing.md`

- [ ] **Step 1: Research market sizes**

Use WebSearch and WebFetch to find:

```
For each market (Vietnam, Singapore, US, EU, UK):
- Total addressable market (TAM): global/regional fintech market size
- Serviceable addressable market (SAM): robo-advisory market in that region
- Serviceable obtainable market (SOM): realistic capture in 5 years
```

Key sources:
- Statista reports on robo-advisory
- McKinsey Global Fintech reports
- CB Insights fintech funding data
- Local fintech associations

- [ ] **Step 2: Build market sizing document**

```markdown
# Market Sizing — AI Robo-Advisor

## Global Context

**Global Fintech Market:** $[X]B (2025) — CAGR [Y]%
**Global Robo-Advisory Market:** $[X]B (2025) — CAGR [Y]%
**Source:** [Link]

## By Market

### Vietnam 🇻🇳
- **Population:** [X]M
- **Adult population:** [X]M (18-65)
- **Smartphone penetration:** [X]%
- **Digital banking users:** [X]M
- **Retail investors:** [X]M (stock market participants)
- **TAM:** $[X]B (total fintech opportunity)
- **SAM:** $[X]B (robo-advisory opportunity)
- **SOM (5yr):** $[X]M (1% capture)
- **Sources:** [Links]

### Singapore 🇸🇬
- **Population:** [X]M
- **Adult population:** [X]M
- **AUM in wealth management:** $[X]B
- **TAM:** $[X]B
- **SAM:** $[X]B
- **SOM (5yr):** $[X]M
- **Sources:** [Links]

### United States 🇺🇸
- [Same format]
- ...

## Summary Table

| Market | TAM | SAM | SOM (5yr) |
|---|---|---|---|
| Vietnam | $[X]B | $[X]B | $[X]M |
| Singapore | $[X]B | $[X]B | $[X]M |
| US | $[X]B | $[X]B | $[X]M |
| **Total** | **$[X]B** | **$[X]B** | **$[X]M** |

## Key Market Insights
- [Insight 1]
- [Insight 2]
```

- [ ] **Step 3: Commit**

```bash
git add docs/research/market-sizing.md
git commit -m "docs: add market sizing - TAM/SAM/SOM by market"
```

---

## Task 6: Phase 1 Report

**Files:**
- Create: `docs/research/phase1-market-research.md`

- [ ] **Step 1: Compile Phase 1 Research Report**

Combine all Phase 1 outputs into one comprehensive report:

```markdown
# Phase 1: Market Research Report
**Period:** Month 1-3
**Status:** Complete

---

## Executive Summary
[2-3 paragraphs: what we learned, key insights, implications]

## Market Opportunity
[Summarize market sizing findings]

## Competitive Landscape
[Summarize competitor analysis — top 3 takeaways]

## User Insights
[Summarize user interview synthesis — top 3 takeaways]

## Regulatory Pathway
[Summarize regulation map — easiest entry market]

## Strategic Recommendations

### Market Entry Recommendation
1. **First market:** [Market] — [Reason]
2. **Second market:** [Market] — [Timing]
3. **Third market:** [Market] — [Timing]

### Product Positioning
[How we position vs existing competitors]

### Key Differentiators to Build
1. [Differentiator 1]
2. [Differentiator 2]
3. [Differentiator 3]

### Risks Identified
- [Risk 1] — [Mitigation]
- [Risk 2] — [Mitigation]

## Next Steps
1. [What to do in Phase 2]
2. [Key questions to answer]
```

- [ ] **Step 2: Commit Phase 1 Report**

```bash
git add docs/research/phase1-market-research.md
git commit -m "docs: complete Phase 1 market research report"
```

- [ ] **Step 3: Tag Phase 1 as complete**

```bash
git tag phase1-complete -m "Phase 1: Market Research complete"
git push origin main --tags
```

---

## Phase 1 Complete — Summary

After completing all tasks:

```bash
# Verify all deliverables exist
ls docs/research/

# Expected output:
# competitor-profiles.md
# competitive-matrix.md
# regulation-map.md
# user-interviews.md
# market-sizing.md
# phase1-market-research.md
```

**Phase 1 deliverable:** 6 research documents covering:
- 10 competitor profiles
- Competitive matrix with gaps
- Regulation map for 5 markets
- User interview synthesis (10-20 interviews)
- Market sizing (TAM/SAM/SOM)
- Phase 1 summary report

---

## Phase 1 Timeline

| Day | Tasks |
|---|---|
| Day 1-2 | Task 1: Competitor audit |
| Day 3-4 | Task 2: Competitive matrix |
| Day 5-7 | Task 3: Regulation map |
| Day 8-12 | Task 4: User interviews |
| Day 13-14 | Task 5: Market sizing |
| Day 15 | Task 6: Compile Phase 1 Report |
