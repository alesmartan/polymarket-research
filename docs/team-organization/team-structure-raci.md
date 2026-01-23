# Team Structure & RACI Matrix

> Organizational structure, decision-making frameworks, and governance for a prediction market platform

## Table of Contents

1. [Organizational Chart](#organizational-chart)
2. [Team Structure by Function](#team-structure-by-function)
3. [RACI Matrix](#raci-matrix)
4. [Decision-Making Framework](#decision-making-framework)
5. [Meeting Cadence](#meeting-cadence)
6. [Communication Channels](#communication-channels)
7. [Reporting Structure](#reporting-structure)
8. [Career Progression Framework](#career-progression-framework)
9. [Performance Review Process](#performance-review-process)

---

## Organizational Chart

### Pre-Seed / Seed Stage (5-10 People)

```
                           ┌─────────────┐
                           │    CEO      │
                           │ Co-Founder  │
                           └──────┬──────┘
                                  │
            ┌─────────────────────┼─────────────────────┐
            │                     │                     │
     ┌──────┴──────┐       ┌──────┴──────┐       ┌──────┴──────┐
     │    CTO      │       │   Product   │       │   Growth    │
     │ Co-Founder  │       │    Lead     │       │    Lead     │
     └──────┬──────┘       └──────┬──────┘       └──────┬──────┘
            │                     │                     │
   ┌────────┼────────┐           │              ┌──────┴──────┐
   │        │        │           │              │             │
┌──┴──┐  ┌──┴──┐  ┌──┴──┐    ┌──┴──┐      ┌──┴──┐      ┌──┴──┐
│ SC  │  │Full │  │DevOp│    │UI/UX│      │Comm-│      │Mktg │
│ Eng │  │Stack│  │  s  │    │     │      │unity│      │     │
└─────┘  └─────┘  └─────┘    └─────┘      └─────┘      └─────┘
```

### Series A Stage (20-30 People)

```
                                    ┌─────────────────┐
                                    │       CEO       │
                                    │                 │
                                    └────────┬────────┘
                                             │
        ┌────────────────┬───────────────────┼───────────────────┬────────────────┐
        │                │                   │                   │                │
┌───────┴───────┐ ┌──────┴──────┐    ┌──────┴──────┐    ┌──────┴──────┐  ┌──────┴──────┐
│      CTO      │ │Head of Prod │    │    COO      │    │Head of Mktg │  │General      │
│               │ │             │    │             │    │             │  │Counsel      │
└───────┬───────┘ └──────┬──────┘    └──────┬──────┘    └──────┬──────┘  └──────┬──────┘
        │                │                   │                   │                │
   ┌────┴────┐      ┌────┴────┐        ┌────┴────┐         ┌────┴────┐      ┌────┴────┐
   │         │      │         │        │         │         │         │      │         │
┌──┴──┐   ┌──┴──┐ ┌─┴─┐    ┌──┴──┐  ┌──┴──┐  ┌──┴──┐   ┌──┴──┐  ┌──┴──┐ ┌──┴──┐  ┌──┴──┐
│Eng  │   │Eng  │ │PM │    │PM   │  │Ops  │  │Fin  │   │Cont-│  │Comm-│ │Compl│  │Legal│
│Mgr  │   │Mgr  │ │(2)│    │     │  │Mgr  │  │Mgr  │   │ent  │  │unity│ │iance│  │     │
│(SC) │   │(App)│ └───┘    └─────┘  └──┬──┘  └─────┘   └──┬──┘  └──┬──┘ └─────┘  └─────┘
└──┬──┘   └──┬──┘                      │                  │        │
   │         │                    ┌────┴────┐        ┌────┴────────┴────┐
┌──┴──┐   ┌──┴──┐              ┌──┴──┐   ┌──┴──┐  ┌──┴──┐  ┌──┴──┐  ┌──┴──┐
│SC   │   │Full │              │Supp-│   │HR/  │  │PR   │  │Social│ │Growth│
│Eng  │   │Stack│              │ort  │   │Peop │  │     │  │     │  │      │
│(4)  │   │(4)  │              │(3)  │   │le   │  └─────┘  └─────┘  └──────┘
└─────┘   └──┬──┘              └─────┘   └─────┘
             │
          ┌──┴──┐
          │DevOp│
          │(2)  │
          └─────┘
```

### Detailed Team Structure (Series A)

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                                    EXECUTIVE TEAM                                   │
├────────────────────────────────────────────────────────────────────────────────────┤
│  CEO           │  CTO           │  COO          │  Head of Prod  │  Head of Mktg   │
│  - Strategy    │  - Technical   │  - Operations │  - Product     │  - Brand        │
│  - Fundraising │    Vision      │  - Finance    │    Roadmap     │  - Growth       │
│  - Board       │  - Security    │  - Legal      │  - User        │  - Comms        │
└────────────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────────────┐
│                                    ENGINEERING                                      │
├────────────────────────────────────────────────────────────────────────────────────┤
│  Protocol Team (5)                │  Application Team (6)                          │
│  ├─ Engineering Manager           │  ├─ Engineering Manager                        │
│  ├─ Sr. Smart Contract Eng (2)    │  ├─ Sr. Full-Stack Engineer (2)               │
│  ├─ Smart Contract Engineer (2)   │  ├─ Full-Stack Engineer (2)                   │
│  └─ Security Engineer             │  └─ DevOps/SRE (2)                            │
└────────────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────────────┐
│                                    PRODUCT                                          │
├────────────────────────────────────────────────────────────────────────────────────┤
│  Product Managers (2)             │  Design (2)                                    │
│  ├─ PM, Core Trading              │  ├─ Sr. Product Designer                       │
│  └─ PM, Growth & Discovery        │  └─ UI/UX Designer                            │
└────────────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────────────┐
│                                    OPERATIONS                                       │
├────────────────────────────────────────────────────────────────────────────────────┤
│  Finance (2)                      │  People (2)          │  Support (3)            │
│  ├─ Finance Manager               │  ├─ HR Manager       │  ├─ Support Lead        │
│  └─ Accountant (fractional)       │  └─ Recruiter        │  └─ Support Agents (2)  │
└────────────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────────────┐
│                                    GROWTH                                           │
├────────────────────────────────────────────────────────────────────────────────────┤
│  Marketing (4)                    │  Community (2)       │  BD (1)                 │
│  ├─ Content Lead                  │  ├─ Community Mgr    │  └─ Partnerships Lead   │
│  ├─ Growth Lead                   │  └─ Community Mod    │                         │
│  ├─ PR Manager                    │                      │                         │
│  └─ Social Media                  │                      │                         │
└────────────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────────────┐
│                                    LEGAL & COMPLIANCE                               │
├────────────────────────────────────────────────────────────────────────────────────┤
│  Legal (2)                        │  Compliance (1)                                │
│  ├─ General Counsel               │  └─ Compliance Manager                         │
│  └─ Legal Counsel                 │                                                │
└────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Team Structure by Function

### Founding Team

| Role | Responsibilities | Key Metrics |
|------|------------------|-------------|
| CEO | Vision, strategy, fundraising, external relations, board management | Funding raised, partnerships, regulatory progress |
| CTO | Technical architecture, engineering leadership, security oversight | Platform reliability, security posture, tech debt |
| COO | Operations, finance, legal, HR, day-to-day execution | Operational efficiency, compliance, team health |

### Engineering

**Mission:** Build a secure, scalable, and delightful prediction market platform.

| Team | Focus | Size (Series A) |
|------|-------|-----------------|
| Protocol | Smart contracts, security, on-chain mechanics | 4-5 engineers |
| Application | Web app, mobile, APIs, backend services | 4-5 engineers |
| Platform | Infrastructure, DevOps, reliability, tooling | 2 engineers |

**Engineering Principles:**
1. Security first - all code is security-critical
2. Ship fast, iterate faster
3. Write tests for everything
4. Document as you build
5. Open source by default

### Product

**Mission:** Understand users deeply and build products that make prediction markets accessible to everyone.

| Role | Focus |
|------|-------|
| Head of Product | Product strategy, roadmap, team leadership |
| PM - Core Trading | Trading experience, market mechanics, power users |
| PM - Growth & Discovery | Onboarding, discovery, casual users |
| Product Designer (Sr.) | Design system, core flows, research |
| UI/UX Designer | Feature design, prototyping, testing |

### Operations

**Mission:** Enable the team to do their best work while maintaining compliance and financial health.

| Function | Responsibilities |
|----------|------------------|
| Finance | Treasury management, accounting, financial planning |
| HR/People | Recruiting, onboarding, culture, compensation |
| Legal | Contracts, regulatory compliance, IP protection |
| Support | User support, issue resolution, feedback collection |

### Growth

**Mission:** Acquire, activate, and retain users through compelling marketing and community engagement.

| Function | Responsibilities |
|----------|------------------|
| Marketing | Brand, content, paid acquisition, PR |
| Community | Discord/Telegram management, events, engagement |
| Partnerships | Exchange listings, integrations, strategic partnerships |

---

## RACI Matrix

### Understanding RACI

| Letter | Role | Definition |
|--------|------|------------|
| **R** | Responsible | Does the work to complete the task |
| **A** | Accountable | Ultimately answerable for the task (only one per task) |
| **C** | Consulted | Provides input before decisions are made |
| **I** | Informed | Kept up-to-date on progress and decisions |

### Product Decisions

| Decision | CEO | CTO | Head of Product | PM | Eng Manager | Designer | Legal |
|----------|-----|-----|-----------------|-----|-------------|----------|-------|
| Product Vision & Strategy | A | C | R | C | I | C | I |
| Feature Prioritization | I | C | A | R | C | C | I |
| Roadmap Planning | I | C | A | R | C | I | I |
| Feature Specifications | I | I | A | R | C | R | C |
| UX/UI Design Decisions | I | I | C | C | I | R/A | I |
| User Research Initiatives | I | I | A | R | I | C | I |
| Launch Decisions | C | C | A | R | R | I | C |
| Feature Deprecation | I | C | A | R | C | I | C |

### Technical Decisions

| Decision | CEO | CTO | Eng Manager | Sr. Engineer | DevOps | Security | Legal |
|----------|-----|-----|-------------|--------------|--------|----------|-------|
| Technical Architecture | I | A | R | R | C | C | I |
| Technology Stack | I | A | R | C | C | C | I |
| Smart Contract Design | I | A | R | R | I | R | C |
| Security Policies | C | A | C | C | C | R | C |
| Infrastructure Decisions | I | A | C | I | R | C | I |
| Code Review Standards | I | A | R | C | I | C | I |
| Deployment Process | I | C | A | I | R | C | I |
| Incident Response | C | A | R | R | R | C | I |
| Audit Scope & Firm | C | A | C | C | I | R | C |

### Financial Decisions

| Decision | CEO | COO | Finance | CTO | Head of Mktg | Legal | Board |
|----------|-----|-----|---------|-----|--------------|-------|-------|
| Annual Budget | A | R | R | C | C | I | C |
| Quarterly Spending | C | A | R | C | C | I | I |
| Compensation Decisions | A | R | C | C | I | C | I |
| Vendor Contracts (>$50K) | A | R | C | C | C | C | I |
| Vendor Contracts (<$50K) | I | A | R | C | C | I | I |
| Treasury Management | A | R | R | I | I | C | C |
| Token Economics | A | C | C | C | C | R | C |
| Fundraising | A | C | R | C | I | R | C |

### Legal Decisions

| Decision | CEO | COO | Legal/GC | Compliance | CTO | Head of Prod | External Counsel |
|----------|-----|-----|----------|------------|-----|--------------|------------------|
| Regulatory Strategy | A | C | R | C | C | I | C |
| Terms of Service | C | C | A | C | I | C | R |
| Privacy Policy | C | C | A | R | C | C | R |
| Jurisdiction Decisions | A | C | R | C | I | I | C |
| Litigation Response | A | C | R | C | I | I | R |
| Patent/IP Decisions | A | I | R | I | C | I | C |
| KYC/AML Implementation | I | C | A | R | C | C | C |
| Compliance Reporting | I | C | A | R | I | I | C |

### Marketing Decisions

| Decision | CEO | COO | Head of Mktg | Content | Community | PR | Legal |
|----------|-----|-----|--------------|---------|-----------|-----|-------|
| Brand Strategy | A | I | R | C | C | C | I |
| Marketing Budget | C | A | R | I | I | I | I |
| Campaign Strategy | I | I | A | R | C | C | C |
| Content Calendar | I | I | A | R | C | I | I |
| PR & Media Relations | C | I | A | C | I | R | C |
| Community Guidelines | I | I | A | I | R | I | C |
| Social Media Strategy | I | I | A | R | C | C | I |
| Partnership Announcements | C | I | A | C | C | R | C |

### Cross-Functional Decisions

| Decision | CEO | CTO | COO | Head of Product | Head of Mktg | Legal |
|----------|-----|-----|-----|-----------------|--------------|-------|
| Product Launch | A | C | C | R | C | C |
| Market Expansion | A | C | C | R | C | R |
| Pricing Changes | A | I | C | R | C | C |
| Major Partnerships | A | C | C | C | R | C |
| Crisis Response | A | R | R | C | R | R |
| Hiring Decisions (Director+) | A | C | R | C | C | I |
| Company Policies | A | C | R | I | I | C |

---

## Decision-Making Framework

### Decision Categories

| Category | Timeline | Approvers | Documentation |
|----------|----------|-----------|---------------|
| **Trivial** | Immediate | Individual | None required |
| **Reversible** | < 1 week | Team lead | Slack message |
| **Significant** | 1-2 weeks | Department head | Brief doc |
| **Strategic** | 2-4 weeks | Executive team | Full proposal |
| **Transformational** | 4+ weeks | CEO + Board | Board presentation |

### Decision Escalation Matrix

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DECISION ESCALATION                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Impact Level     │  Financial Impact  │  Decision Maker   │  Approval     │
│  ───────────────────────────────────────────────────────────────────────── │
│  Trivial          │  < $1K             │  Individual       │  Self         │
│  Minor            │  $1K - $10K        │  Team Lead        │  Manager      │
│  Moderate         │  $10K - $50K       │  Dept. Head       │  COO          │
│  Major            │  $50K - $250K      │  Executive Team   │  CEO          │
│  Transformational │  > $250K           │  CEO + Board      │  Board        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### DARE Framework (Alternative to RACI for Major Decisions)

For complex decisions, use the DARE framework:

| Role | Definition | Example |
|------|------------|---------|
| **D**ecider | Has final authority, makes the call | CEO for strategic decisions |
| **A**dvisor | Provides expert input, has significant influence | CTO for technical decisions |
| **R**ecommender | Does analysis, presents options with recommendation | PM for product decisions |
| **E**xecution | Implements the decision once made | Engineering team |

### Decision Documentation Template

```markdown
## Decision Record: [Title]

**Date:** YYYY-MM-DD
**Decider:** [Name, Role]
**Status:** [Proposed | Approved | Rejected | Superseded]

### Context
[Why is this decision needed? What problem are we solving?]

### Options Considered
1. **Option A:** [Description]
   - Pros: [...]
   - Cons: [...]

2. **Option B:** [Description]
   - Pros: [...]
   - Cons: [...]

### Decision
[What was decided and why]

### Consequences
[Expected impact, risks, and how we'll measure success]

### Stakeholders Consulted
- [Name, Role] - [Input summary]
```

---

## Meeting Cadence

### Daily Meetings

| Meeting | Participants | Duration | Time | Purpose |
|---------|--------------|----------|------|---------|
| Engineering Standup | All engineers | 15 min | 10:00 AM PT | Progress, blockers, coordination |
| Product Sync | Product + Design | 15 min | 10:15 AM PT | Daily priorities, quick decisions |

**Standup Format:**
1. What I completed yesterday
2. What I'm working on today
3. Blockers or help needed

### Weekly Meetings

| Meeting | Participants | Duration | Day/Time | Purpose |
|---------|--------------|----------|----------|---------|
| Executive Sync | CEO, CTO, COO, Heads | 60 min | Monday 9 AM | Strategic alignment, key decisions |
| All Hands | Everyone | 30 min | Monday 11 AM | Company updates, wins, priorities |
| Engineering Review | Engineering team | 60 min | Tuesday 2 PM | Technical deep dives, architecture |
| Product Review | Product, Design, Eng leads | 60 min | Wednesday 2 PM | Feature reviews, roadmap |
| Growth Review | Marketing, Community, BD | 45 min | Thursday 2 PM | Metrics, campaigns, community |
| 1:1s | Manager + Report | 30 min | Various | Personal development, feedback |

**All Hands Agenda:**
1. Wins of the week (5 min)
2. Key metrics update (5 min)
3. Priorities for the week (10 min)
4. Q&A (10 min)

### Monthly Meetings

| Meeting | Participants | Duration | Purpose |
|---------|--------------|----------|---------|
| Monthly Business Review | Exec team + leads | 90 min | Full metrics review, course correction |
| Team Retrospectives | Individual teams | 60 min | Process improvement |
| Security Review | CTO, Security, Eng leads | 60 min | Security posture, vulnerabilities |
| Finance Review | CEO, COO, Finance | 60 min | Budget tracking, projections |

**Monthly Business Review Agenda:**
1. Metrics dashboard review (20 min)
2. Product update & roadmap (20 min)
3. Growth & marketing update (15 min)
4. Operations & finance (15 min)
5. Key decisions & action items (20 min)

### Quarterly Meetings

| Meeting | Participants | Duration | Purpose |
|---------|--------------|----------|---------|
| Quarterly Planning | All teams | Half day | OKR setting, roadmap planning |
| Board Meeting | Exec team + Board | 2-3 hours | Investor update, governance |
| Strategy Offsite | Leadership team | 1-2 days | Long-term strategy, team building |
| Performance Reviews | All employees | 1 hour each | Formal feedback, growth plans |

**Quarterly Planning Process:**
1. Week -2: Teams draft proposed OKRs
2. Week -1: Cross-team alignment discussions
3. Planning Day: Finalize OKRs, dependencies
4. Week +1: Communicate to company, start execution

---

## Communication Channels

### Channel Overview

| Channel | Purpose | Response Time | Examples |
|---------|---------|---------------|----------|
| **Slack** | Day-to-day communication | < 4 hours (business hours) | Questions, updates, quick decisions |
| **Email** | External communication, formal records | < 24 hours | Investor updates, vendor contracts |
| **Notion** | Documentation, async decisions | N/A | Specs, policies, meeting notes |
| **Linear** | Task tracking, engineering work | N/A | Bugs, features, sprints |
| **GitHub** | Code, technical discussions | PR-dependent | Code reviews, technical RFCs |

### Slack Channel Structure

| Channel | Purpose | Members |
|---------|---------|---------|
| `#general` | Company-wide announcements | Everyone |
| `#random` | Non-work chat, social | Everyone |
| `#engineering` | Engineering discussions | Engineering team |
| `#product` | Product discussions | Product, Design, Eng leads |
| `#growth` | Marketing, community updates | Growth team |
| `#support` | User issues, escalations | Support, Engineering |
| `#alerts` | System alerts, monitoring | Engineering, On-call |
| `#security` | Security-sensitive discussions | Security team, leadership |
| `#legal-compliance` | Legal and compliance matters | Legal, Compliance, Exec |
| `#wins` | Celebrate achievements | Everyone |
| `#feedback` | User feedback, insights | Product, Support, Community |

### Communication Guidelines

#### Async-First Principles
1. **Default to async:** Most communication should not require immediate response
2. **Document decisions:** Write it down in Notion, not just Slack
3. **Respect time zones:** Don't expect immediate responses outside business hours
4. **Batch notifications:** Check Slack 3-4x/day, not continuously

#### When to Use Each Channel

| Scenario | Channel |
|----------|---------|
| Quick question for teammate | Slack DM or relevant channel |
| Needs input from multiple people | Slack channel thread |
| Decision that needs documentation | Notion doc with Slack link |
| Urgent production issue | Slack `#alerts` + page on-call |
| Formal feedback or HR matter | Email or scheduled call |
| Technical design discussion | GitHub RFC or Notion doc |
| Sensitive/confidential matter | Direct message or call |

#### Escalation Path

```
Level 1: Slack channel post
    ↓ (No response in 2 hours)
Level 2: Direct Slack message to relevant person
    ↓ (No response in 1 hour during business)
Level 3: Text message or phone call
    ↓ (Urgent production issue)
Level 4: Page on-call via PagerDuty
```

---

## Reporting Structure

### Direct Reports by Role

| Leader | Direct Reports |
|--------|----------------|
| CEO | CTO, COO, Head of Product, Head of Marketing, General Counsel |
| CTO | Engineering Managers, Security Lead |
| COO | Finance Manager, HR Manager, Operations Manager |
| Head of Product | Product Managers, Design Lead |
| Head of Marketing | Content Lead, Community Lead, PR Manager, Growth Lead |
| Engineering Manager (Protocol) | Smart Contract Engineers, Security Engineer |
| Engineering Manager (App) | Full-Stack Engineers, DevOps Engineers |

### Span of Control Guidelines

| Level | Recommended Direct Reports | Notes |
|-------|---------------------------|-------|
| Executive | 4-7 | Strategic focus, delegation important |
| Director | 5-8 | Balance strategic and tactical |
| Manager | 5-10 | Primarily people management |
| Tech Lead | 3-6 | Technical leadership + IC work |

### Matrix Relationships

Some roles have dotted-line reporting relationships:

| Role | Solid Line | Dotted Line |
|------|------------|-------------|
| Product Designer | Head of Product | Engineering Manager |
| DevOps Engineer | Eng Manager (App) | CTO (for security) |
| Community Manager | Head of Marketing | Head of Product |
| Compliance Manager | General Counsel | COO |

---

## Career Progression Framework

### Engineering Levels

| Level | Title | Years Experience | Scope | Compensation Band |
|-------|-------|------------------|-------|-------------------|
| E1 | Junior Engineer | 0-2 | Tasks | $100K-$140K |
| E2 | Engineer | 2-4 | Features | $130K-$170K |
| E3 | Senior Engineer | 4-6 | Projects | $160K-$220K |
| E4 | Staff Engineer | 6-10 | Team/Domain | $200K-$280K |
| E5 | Principal Engineer | 10+ | Organization | $250K-$350K |
| M3 | Engineering Manager | 4-6 | Team (5-8) | $180K-$250K |
| M4 | Senior Engineering Manager | 6-10 | Multiple Teams | $220K-$300K |
| M5 | Director of Engineering | 10+ | Department | $280K-$380K |

### Engineering Level Expectations

#### E1 - Junior Engineer
- Completes well-defined tasks with guidance
- Writes clean, tested code
- Asks questions and learns actively
- Participates in code reviews

#### E2 - Engineer
- Independently completes features
- Contributes to technical discussions
- Mentors junior engineers
- Improves team processes

#### E3 - Senior Engineer
- Leads project delivery
- Makes significant technical decisions
- Identifies and solves ambiguous problems
- Drives cross-team initiatives

#### E4 - Staff Engineer
- Defines technical strategy for domain
- Influences engineering culture
- Mentors senior engineers
- Represents engineering externally

#### E5 - Principal Engineer
- Shapes company-wide technical direction
- Solves hardest technical challenges
- Industry thought leadership
- Reports to CTO

### Product & Design Levels

| Level | Title | Scope |
|-------|-------|-------|
| P1 | Associate PM / Junior Designer | Features with guidance |
| P2 | Product Manager / Designer | Full features independently |
| P3 | Senior PM / Senior Designer | Product areas |
| P4 | Group PM / Design Lead | Multiple areas or team |
| P5 | Head of Product / Head of Design | Department |

### Promotion Criteria

| Factor | Weight | Description |
|--------|--------|-------------|
| Impact | 40% | Business results, technical contributions |
| Scope | 25% | Complexity and breadth of work |
| Leadership | 20% | Influence, mentorship, culture |
| Expertise | 15% | Technical depth, domain knowledge |

### Promotion Process

1. **Self-Assessment:** Employee documents achievements against level criteria
2. **Manager Assessment:** Manager provides independent evaluation
3. **Calibration:** Leadership reviews all candidates at level boundary
4. **Decision:** Promote, develop further, or maintain
5. **Communication:** Manager delivers feedback within 1 week

---

## Performance Review Process

### Review Cadence

| Review Type | Frequency | Purpose |
|-------------|-----------|---------|
| Continuous Feedback | Ongoing | Real-time guidance and recognition |
| 1:1 Discussions | Weekly | Progress, blockers, development |
| Quarterly Check-in | Every 3 months | OKR progress, trajectory |
| Annual Review | Yearly | Comprehensive evaluation, compensation |

### Annual Review Timeline

| Week | Activity |
|------|----------|
| Week 1 | Self-reviews distributed |
| Week 2 | Self-reviews due, peer feedback requested |
| Week 3 | Manager reviews written |
| Week 4 | Calibration meetings |
| Week 5 | Reviews finalized, compensation decisions |
| Week 6 | Review delivery meetings |

### Review Components

#### Self-Review Questions
1. What were your key accomplishments this period?
2. What goals did you not achieve and why?
3. What feedback have you received and acted on?
4. What are your development goals for next period?
5. What support do you need from your manager?

#### Manager Review Sections
1. **Summary:** Overall performance assessment
2. **Accomplishments:** Key achievements and impact
3. **Strengths:** Areas of excellence
4. **Development Areas:** Opportunities for growth
5. **Goals:** Expectations for next period
6. **Rating:** Performance level (1-5 scale)

### Performance Ratings

| Rating | Description | Distribution Target |
|--------|-------------|---------------------|
| 5 - Exceptional | Consistently exceeds expectations, top performer | 5-10% |
| 4 - Exceeds | Regularly exceeds expectations | 20-30% |
| 3 - Meets | Consistently meets expectations | 50-60% |
| 2 - Below | Sometimes meets expectations, needs improvement | 10-15% |
| 1 - Unsatisfactory | Does not meet expectations | 0-5% |

### Performance Improvement Plans (PIP)

For employees rated 2 or below:

| Phase | Duration | Activities |
|-------|----------|------------|
| Warning | Week 1 | Clear feedback, documented expectations |
| PIP Start | Week 2-4 | Weekly check-ins, specific goals |
| Mid-Point Review | Week 4 | Progress assessment, adjust if needed |
| Final Review | Week 6-8 | Success or transition discussion |

### Compensation Adjustments

| Rating | Merit Increase | Bonus Eligibility |
|--------|----------------|-------------------|
| 5 - Exceptional | 8-15% | 150% of target |
| 4 - Exceeds | 5-8% | 125% of target |
| 3 - Meets | 3-5% | 100% of target |
| 2 - Below | 0-2% | 50% of target |
| 1 - Unsatisfactory | 0% | 0% |

---

## Appendix: Templates

### RACI Template (Blank)

| Decision/Task | CEO | CTO | COO | Product | Eng | Design | Marketing | Legal |
|---------------|-----|-----|-----|---------|-----|--------|-----------|-------|
| [Task 1] | | | | | | | | |
| [Task 2] | | | | | | | | |

### Meeting Agenda Template

```markdown
# [Meeting Name] - [Date]

## Attendees
- [Name 1]
- [Name 2]

## Agenda
1. [Topic 1] - [Owner] - [Time]
2. [Topic 2] - [Owner] - [Time]
3. [Topic 3] - [Owner] - [Time]

## Pre-Read Materials
- [Link to doc 1]
- [Link to doc 2]

## Notes
[To be filled during meeting]

## Action Items
- [ ] [Action] - [Owner] - [Due Date]
```

### 1:1 Template

```markdown
# 1:1: [Employee Name] & [Manager Name]
**Date:** YYYY-MM-DD

## Employee Updates
-

## Manager Updates
-

## Discussion Topics
1.
2.

## Feedback
- For employee:
- For manager:

## Action Items
- [ ]

## Next Meeting Topics
-
```

---

*Document Version: 1.0*
*Last Updated: January 2025*
*Owner: People Operations*
