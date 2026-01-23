# Incident Response Plan for Prediction Market Platform

> A comprehensive incident response framework designed for crypto/fintech platforms, based on industry best practices from PagerDuty, Google SRE, and cryptocurrency security standards.

---

## Table of Contents

1. [Incident Severity Levels (P1-P4)](#incident-severity-levels-p1-p4)
2. [On-Call Rotation Schedule](#on-call-rotation-schedule)
3. [Incident Commander Role](#incident-commander-role)
4. [Communication Protocols](#communication-protocols)
5. [Incident Response Flowchart](#incident-response-flowchart)
6. [Post-Incident Review Process](#post-incident-review-process)
7. [Blameless Postmortem Template](#blameless-postmortem-template)
8. [Incident Metrics to Track](#incident-metrics-to-track)
9. [Runbooks for Common Incidents](#runbooks-for-common-incidents)

---

## Incident Severity Levels (P1-P4)

### Severity Definitions

```
┌─────────────────────────────────────────────────────────────────┐
│                    INCIDENT SEVERITY LEVELS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  P1 - CRITICAL                                                  │
│  ─────────────                                                  │
│  • Complete service outage                                      │
│  • User funds at risk or lost                                   │
│  • Active security breach/exploit                               │
│  • Smart contract vulnerability being exploited                 │
│  • Data breach confirmed                                        │
│  • Regulatory compliance violation                              │
│                                                                  │
│  Response: IMMEDIATE (24/7)                                     │
│  Resolution Target: 2 hours                                     │
│  Escalation: Automatic to leadership                            │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  P2 - HIGH                                                      │
│  ─────────                                                      │
│  • Major functionality impaired                                 │
│  • Trading not working for significant users                    │
│  • Deposits/withdrawals failing                                 │
│  • Oracle data not updating                                     │
│  • Significant performance degradation (>3x normal latency)     │
│  • Market resolution errors                                     │
│                                                                  │
│  Response: 15 minutes (24/7)                                    │
│  Resolution Target: 4 hours                                     │
│  Escalation: Engineering Lead after 1 hour                      │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  P3 - MEDIUM                                                    │
│  ──────────                                                     │
│  • Partial functionality impaired                               │
│  • Workaround available                                         │
│  • Performance issues affecting minority of users               │
│  • Non-critical integrations failing                            │
│  • UI/UX bugs affecting usability                               │
│                                                                  │
│  Response: 1 hour (business hours)                              │
│  Resolution Target: 24 hours                                    │
│  Escalation: Team lead after 8 hours                            │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  P4 - LOW                                                       │
│  ────────                                                       │
│  • Minor issues with minimal impact                             │
│  • Cosmetic bugs                                                │
│  • Feature requests or improvements                             │
│  • Documentation issues                                         │
│  • Monitoring false positives                                   │
│                                                                  │
│  Response: 4 hours (business hours)                             │
│  Resolution Target: Best effort / Next sprint                   │
│  Escalation: N/A                                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Severity Assessment Matrix

| Factor | P1 | P2 | P3 | P4 |
|--------|-----|-----|-----|-----|
| Users Affected | All / Majority | Many (>20%) | Some (<20%) | Few (<5%) |
| Funds at Risk | Yes | Possible | No | No |
| Revenue Impact | Major | Significant | Minor | Negligible |
| Security Impact | Active exploit | Vulnerability found | Potential risk | None |
| Workaround Available | No | Limited | Yes | Yes |
| Reputational Risk | High | Medium | Low | None |

### Escalation Timelines

| Severity | First Response | Engineering On-Call | Engineering Lead | CTO/CEO |
|----------|---------------|---------------------|------------------|---------|
| P1 | Immediate | Immediate | 15 min | 30 min |
| P2 | 15 min | 15 min | 1 hour | 4 hours |
| P3 | 1 hour | Best effort | 8 hours | N/A |
| P4 | 4 hours | Best effort | N/A | N/A |

---

## On-Call Rotation Schedule

### On-Call Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    ON-CALL ROTATION STRUCTURE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PRIMARY ON-CALL                                                │
│  ───────────────                                                │
│  • First responder for all alerts                               │
│  • Available 24/7 during shift                                  │
│  • Response time: 5 minutes                                     │
│  • Rotation: Weekly                                             │
│                                                                  │
│  SECONDARY ON-CALL                                              │
│  ─────────────────                                              │
│  • Backup if primary unavailable                                │
│  • Escalation point for primary                                 │
│  • Available 24/7 during shift                                  │
│  • Response time: 15 minutes                                    │
│  • Rotation: Weekly (offset by one person)                      │
│                                                                  │
│  SPECIALIST ON-CALL (As Needed)                                 │
│  ──────────────────────────────                                 │
│  • Smart Contract Engineer                                      │
│  • Security Engineer                                            │
│  • DevOps/Infrastructure                                        │
│  • Database Administrator                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Rotation Schedule

| Week | Primary | Secondary | Smart Contract | Security |
|------|---------|-----------|----------------|----------|
| 1 | Engineer A | Engineer B | SC Engineer 1 | Security 1 |
| 2 | Engineer B | Engineer C | SC Engineer 2 | Security 1 |
| 3 | Engineer C | Engineer D | SC Engineer 1 | Security 2 |
| 4 | Engineer D | Engineer A | SC Engineer 2 | Security 2 |

### On-Call Expectations

| Requirement | Details |
|-------------|---------|
| Response Time | Acknowledge alert within 5 minutes |
| Availability | Must be reachable via phone and Slack |
| Location | Must have laptop and internet access |
| Sobriety | Must be in condition to respond effectively |
| Handoff | Formal handoff required at rotation change |
| Compensation | On-call stipend + incident response pay |

### On-Call Handoff Template

```markdown
## On-Call Handoff

**Date:** [Date]
**Outgoing:** [Name]
**Incoming:** [Name]

### Active Issues
| Issue | Status | Notes |
|-------|--------|-------|

### Recent Incidents (Last 7 Days)
| Incident | Severity | Status | Learnings |
|----------|----------|--------|-----------|

### Upcoming Events
| Event | Date | Risk Level |
|-------|------|------------|

### Known Issues to Watch
- [Issue 1]
- [Issue 2]

### Handoff Confirmed
- [ ] Incoming on-call has acknowledged
- [ ] PagerDuty rotation updated
- [ ] Slack status updated
```

---

## Incident Commander Role

### Incident Commander Responsibilities

```
┌─────────────────────────────────────────────────────────────────┐
│                    INCIDENT COMMANDER ROLE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PRIMARY RESPONSIBILITIES                                       │
│  ────────────────────────                                       │
│  1. Declare incident and set severity level                     │
│  2. Coordinate response team                                    │
│  3. Make decisions on response actions                          │
│  4. Manage communication (internal and external)                │
│  5. Track incident timeline                                     │
│  6. Escalate when necessary                                     │
│  7. Declare incident resolved                                   │
│  8. Initiate post-incident review                               │
│                                                                  │
│  INCIDENT COMMANDER IS NOT                                      │
│  ─────────────────────────                                      │
│  • Doing the technical work (that's the response team)          │
│  • Writing code or making changes                               │
│  • The person who caused the incident                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Supporting Roles

| Role | Responsibilities |
|------|-----------------|
| **Incident Commander (IC)** | Coordinates response, makes decisions, owns communication |
| **Technical Lead** | Leads technical investigation and resolution |
| **Communications Lead** | Manages external communication (status page, social) |
| **Scribe** | Documents timeline, actions, and decisions |
| **Subject Matter Experts** | Provide expertise on specific systems |

### Incident Commander Checklist

```markdown
## Incident Commander Checklist

### Upon Incident Declaration
- [ ] Acknowledge alert in PagerDuty
- [ ] Create incident channel (#incident-[date]-[brief-name])
- [ ] Post incident bridge link (Zoom/Google Meet)
- [ ] Announce: "I am IC for this incident"
- [ ] Assign roles (Tech Lead, Comms, Scribe)
- [ ] Set severity level

### During Incident
- [ ] Ensure technical team is investigating
- [ ] Provide status updates every [15/30/60] minutes based on severity
- [ ] Update status page if customer-facing
- [ ] Escalate if needed
- [ ] Document key decisions
- [ ] Manage responder fatigue (rotate if >2 hours)

### Resolution
- [ ] Confirm issue is resolved
- [ ] Verify with monitoring
- [ ] Update status page to resolved
- [ ] Announce resolution in incident channel
- [ ] Schedule post-incident review

### After Incident
- [ ] Ensure incident is logged
- [ ] Collect timeline from Scribe
- [ ] Schedule postmortem (within 48 hours for P1/P2)
- [ ] Thank the response team
```

---

## Communication Protocols

### Internal Communication

```
┌─────────────────────────────────────────────────────────────────┐
│                 INTERNAL COMMUNICATION FLOW                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ALERTING                                                       │
│  ────────                                                       │
│  PagerDuty ──► Primary On-Call (Push + Phone)                   │
│           ──► Secondary On-Call (Push, 5 min delay)             │
│           ──► Engineering Lead (Push, P1/P2 only)               │
│                                                                  │
│  COORDINATION                                                   │
│  ────────────                                                   │
│  Slack Channels:                                                │
│  • #incidents - All incidents logged                            │
│  • #incident-[id] - Per-incident discussion                     │
│  • #engineering - General awareness                             │
│  • #leadership - Executive updates (P1 only)                    │
│                                                                  │
│  VIDEO BRIDGE                                                   │
│  ────────────                                                   │
│  • Standing Zoom/Meet link for incidents                        │
│  • Auto-record for postmortem                                   │
│                                                                  │
│  UPDATE FREQUENCY                                               │
│  ────────────────                                               │
│  • P1: Every 15 minutes                                         │
│  • P2: Every 30 minutes                                         │
│  • P3: Every 2 hours                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### External Communication

| Channel | Purpose | Owner | Template |
|---------|---------|-------|----------|
| Status Page | Real-time system status | DevOps/IC | Automated + Manual |
| Twitter | Public updates, customer communication | Comms Lead | Pre-approved |
| Discord/Telegram | Community updates | Community Manager | Pre-approved |
| Email | Direct user notification | Comms Lead | Pre-approved |
| In-App Banner | User notification | Product | Pre-approved |

### Status Page Update Templates

**Investigating:**
```
We are investigating reports of [brief description]. Users may experience [symptoms]. We are actively working to identify the cause and will provide updates.

Started: [Time UTC]
```

**Identified:**
```
We have identified the cause of [brief description]. [Brief explanation without sensitive details]. We are working on a fix and expect resolution within [timeframe].

Started: [Time UTC]
Identified: [Time UTC]
```

**Monitoring:**
```
A fix has been deployed for [brief description]. We are monitoring to ensure the issue is fully resolved. Users should see normal service restored.

Started: [Time UTC]
Identified: [Time UTC]
Fix deployed: [Time UTC]
```

**Resolved:**
```
The incident affecting [brief description] has been resolved. Service has been fully restored. We apologize for any inconvenience caused.

Started: [Time UTC]
Identified: [Time UTC]
Resolved: [Time UTC]
Duration: [Duration]

A detailed postmortem will be published within [timeframe].
```

### Communication Approval Matrix

| Severity | Internal | Status Page | Social Media | Press |
|----------|----------|-------------|--------------|-------|
| P1 | IC auto-posts | IC approves | CEO approves | CEO + PR |
| P2 | IC auto-posts | IC approves | Comms Lead | N/A |
| P3 | IC auto-posts | IC approves | N/A | N/A |
| P4 | Auto-logged | N/A | N/A | N/A |

---

## Incident Response Flowchart

### Overall Response Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                 INCIDENT RESPONSE FLOWCHART                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                                               │
│  │   ALERT      │                                               │
│  │   TRIGGERED  │                                               │
│  └──────┬───────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────┐    NO     ┌──────────────┐                   │
│  │ Real Issue?  │─────────► │ Tune Alert   │                   │
│  │              │           │ Document FP  │                   │
│  └──────┬───────┘           └──────────────┘                   │
│         │ YES                                                   │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │ ACKNOWLEDGE  │                                               │
│  │ (5 min SLA)  │                                               │
│  └──────┬───────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │   ASSESS     │                                               │
│  │   SEVERITY   │                                               │
│  └──────┬───────┘                                               │
│         │                                                       │
│    ┌────┼────┬────┐                                             │
│    │    │    │    │                                             │
│    ▼    ▼    ▼    ▼                                             │
│   P1   P2   P3   P4                                             │
│    │    │    │    │                                             │
│    ▼    ▼    │    │                                             │
│  ┌─────────┐ │    │                                             │
│  │ CREATE  │ │    │                                             │
│  │ WAR     │ │    │                                             │
│  │ ROOM    │ │    │                                             │
│  └────┬────┘ │    │                                             │
│       │      │    │                                             │
│       └──────┴────┴──────────┐                                  │
│                              │                                  │
│                              ▼                                  │
│                       ┌──────────────┐                          │
│                       │ INVESTIGATE  │                          │
│                       │ & DIAGNOSE   │                          │
│                       └──────┬───────┘                          │
│                              │                                  │
│                              ▼                                  │
│                       ┌──────────────┐                          │
│                       │   CONTAIN    │◄─────┐                   │
│                       │   (if needed)│      │                   │
│                       └──────┬───────┘      │                   │
│                              │              │                   │
│                              ▼              │                   │
│                       ┌──────────────┐      │                   │
│                       │   RESOLVE    │      │                   │
│                       │              │      │                   │
│                       └──────┬───────┘      │                   │
│                              │              │                   │
│                              ▼              │                   │
│                       ┌──────────────┐  NO  │                   │
│                       │   VERIFY     │──────┘                   │
│                       │   FIXED?     │                          │
│                       └──────┬───────┘                          │
│                              │ YES                              │
│                              ▼                                  │
│                       ┌──────────────┐                          │
│                       │  CLOSE       │                          │
│                       │  INCIDENT    │                          │
│                       └──────┬───────┘                          │
│                              │                                  │
│                              ▼                                  │
│                       ┌──────────────┐                          │
│                       │  POSTMORTEM  │                          │
│                       └──────────────┘                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Security Incident Flow

```
┌─────────────────────────────────────────────────────────────────┐
│              SECURITY INCIDENT RESPONSE FLOW                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                                               │
│  │  SECURITY    │                                               │
│  │  ALERT       │                                               │
│  └──────┬───────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │ IMMEDIATE    │  • Page Security Team                         │
│  │ ACTIONS      │  • Page CTO                                   │
│  │              │  • Preserve evidence                          │
│  └──────┬───────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────┐                                               │
│  │ ASSESS SCOPE │  • What systems affected?                     │
│  │              │  • Are funds at risk?                         │
│  │              │  • Is attack ongoing?                         │
│  └──────┬───────┘                                               │
│         │                                                       │
│    ┌────┴────┐                                                  │
│    │         │                                                  │
│    ▼         ▼                                                  │
│  ACTIVE    CONTAINED                                            │
│  ATTACK    /PAST                                                │
│    │         │                                                  │
│    ▼         │                                                  │
│  ┌─────────┐ │                                                  │
│  │ CONTAIN │ │  • Pause smart contracts                         │
│  │         │ │  • Block suspicious addresses                    │
│  │         │ │  • Disable affected features                     │
│  └────┬────┘ │                                                  │
│       │      │                                                  │
│       └──────┴───────┐                                          │
│                      │                                          │
│                      ▼                                          │
│               ┌──────────────┐                                  │
│               │ INVESTIGATE  │  • Forensic analysis             │
│               │              │  • Identify attack vector        │
│               │              │  • Assess damage                 │
│               └──────┬───────┘                                  │
│                      │                                          │
│                      ▼                                          │
│               ┌──────────────┐                                  │
│               │ COMMUNICATE  │  • Internal: Leadership          │
│               │              │  • External: Users (if needed)   │
│               │              │  • Legal: If required            │
│               │              │  • Regulators: If required       │
│               └──────┬───────┘                                  │
│                      │                                          │
│                      ▼                                          │
│               ┌──────────────┐                                  │
│               │ REMEDIATE    │  • Patch vulnerability           │
│               │              │  • Rotate credentials            │
│               │              │  • Restore from backup           │
│               └──────┬───────┘                                  │
│                      │                                          │
│                      ▼                                          │
│               ┌──────────────┐                                  │
│               │ RECOVERY     │  • Resume operations             │
│               │              │  • Monitor for recurrence        │
│               │              │  • User communication            │
│               └──────────────┘                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Post-Incident Review Process

### Postmortem Culture Principles

```
┌─────────────────────────────────────────────────────────────────┐
│               BLAMELESS POSTMORTEM PRINCIPLES                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CORE BELIEFS                                                   │
│  ───────────                                                    │
│  • People acted with the best intentions given the information  │
│    they had at the time                                         │
│  • Humans make mistakes; systems should prevent them            │
│  • Blame prevents learning; curiosity enables it                │
│  • The goal is to fix systems, not find scapegoats              │
│  • Transparency leads to better outcomes                        │
│                                                                  │
│  WHAT WE DO                                                     │
│  ──────────                                                     │
│  ✓ Focus on what happened, not who did it                       │
│  ✓ Ask "why" not "who"                                          │
│  ✓ Identify systemic improvements                               │
│  ✓ Share learnings widely                                       │
│  ✓ Follow up on action items                                    │
│                                                                  │
│  WHAT WE DON'T DO                                               │
│  ────────────────                                               │
│  ✗ Name and shame individuals                                   │
│  ✗ Punish people for honest mistakes                            │
│  ✗ Accept "be more careful" as an action item                   │
│  ✗ Ignore systemic issues                                       │
│  ✗ Skip postmortems due to time pressure                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When to Write a Postmortem

| Criteria | Required |
|----------|----------|
| P1 incident | Always |
| P2 incident | Always |
| P3 incident with interesting learnings | Recommended |
| Near-miss (could have been serious) | Recommended |
| User-reported issue that took >4 hours to resolve | Recommended |
| Any incident where we want to share learnings | Optional |

### Postmortem Timeline

| Task | Timing | Owner |
|------|--------|-------|
| Create postmortem document | Within 24 hours | Incident Commander |
| Gather timeline and evidence | Within 48 hours | Scribe + IC |
| Initial draft complete | Within 72 hours | IC + Technical Lead |
| Review meeting scheduled | Within 5 business days | IC |
| Review meeting held | Within 7 business days | All stakeholders |
| Final document published | Within 10 business days | IC |
| Action items assigned | At review meeting | IC |
| Action items tracked | Ongoing | Engineering Lead |

---

## Blameless Postmortem Template

```markdown
# Postmortem: [Incident Title]

**Date:** [Date of incident]
**Authors:** [Names]
**Status:** Draft | In Review | Final
**Incident ID:** [ID from tracking system]

---

## Executive Summary

[2-3 sentence summary of what happened, impact, and resolution]

---

## Impact

**Duration:** [Start time] to [End time] ([total duration])
**Severity:** P[1/2/3/4]

### User Impact
- Users affected: [Number/percentage]
- Features affected: [List]
- Financial impact: [Amount, if applicable]

### Business Impact
- Revenue impact: [Estimated]
- Reputation impact: [Assessment]
- Regulatory implications: [If any]

---

## Timeline (All times UTC)

| Time | Event |
|------|-------|
| HH:MM | First alert triggered |
| HH:MM | On-call acknowledged |
| HH:MM | [Key event] |
| HH:MM | Root cause identified |
| HH:MM | Fix deployed |
| HH:MM | Issue resolved |
| HH:MM | Incident closed |

---

## Root Cause Analysis

### What Happened

[Detailed technical explanation of what went wrong]

### Five Whys Analysis

1. **Why did [symptom] occur?**
   Because [answer 1]

2. **Why did [answer 1] happen?**
   Because [answer 2]

3. **Why did [answer 2] happen?**
   Because [answer 3]

4. **Why did [answer 3] happen?**
   Because [answer 4]

5. **Why did [answer 4] happen?**
   Because [root cause]

### Root Cause

[Clear statement of the root cause]

### Contributing Factors

- [Factor 1]
- [Factor 2]
- [Factor 3]

---

## Detection

**How was the incident detected?**
[ ] Automated monitoring
[ ] User report
[ ] Internal discovery
[ ] Other: ___

**Detection time:** [How long after incident started was it detected?]

**Detection gap analysis:**
[What could have detected this sooner?]

---

## Response Assessment

### What Went Well

- [Positive 1]
- [Positive 2]
- [Positive 3]

### What Could Be Improved

- [Improvement 1]
- [Improvement 2]
- [Improvement 3]

### Where We Got Lucky

- [Lucky break 1]
- [Lucky break 2]

---

## Action Items

| Action | Type | Priority | Owner | Due Date | Status |
|--------|------|----------|-------|----------|--------|
| [Action 1] | Prevent | P1 | [Name] | [Date] | Open |
| [Action 2] | Detect | P2 | [Name] | [Date] | Open |
| [Action 3] | Mitigate | P2 | [Name] | [Date] | Open |
| [Action 4] | Process | P3 | [Name] | [Date] | Open |

**Action Types:**
- **Prevent:** Stops this from happening again
- **Detect:** Helps us find this faster
- **Mitigate:** Reduces impact when it happens
- **Process:** Improves response procedures

---

## Lessons Learned

### Key Takeaways

1. [Lesson 1]
2. [Lesson 2]
3. [Lesson 3]

### What Should Be Shared

- [ ] All-hands presentation
- [ ] Engineering blog post
- [ ] External blog post
- [ ] Training material

---

## Supporting Information

### Relevant Graphs/Dashboards
[Screenshots or links]

### Related Documents
- [Link to incident channel archive]
- [Link to relevant runbook]
- [Link to related previous postmortems]

---

## Approval

| Role | Name | Date |
|------|------|------|
| Author | | |
| Technical Reviewer | | |
| Engineering Lead | | |

---

*This postmortem follows the blameless postmortem principles. The goal is learning, not blaming.*
```

---

## Incident Metrics to Track

### Key Performance Indicators

```
┌─────────────────────────────────────────────────────────────────┐
│                    INCIDENT METRICS DASHBOARD                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DETECTION METRICS                                              │
│  ─────────────────                                              │
│  MTTD (Mean Time to Detect)                                     │
│  └── Target: < 5 minutes for P1/P2                              │
│  Alert Noise Ratio                                              │
│  └── Target: < 20% false positives                              │
│                                                                  │
│  RESPONSE METRICS                                               │
│  ────────────────                                               │
│  MTTA (Mean Time to Acknowledge)                                │
│  └── Target: < 5 minutes for P1/P2                              │
│  MTTR (Mean Time to Resolve)                                    │
│  └── Target: < 2 hours for P1, < 4 hours for P2                 │
│                                                                  │
│  FREQUENCY METRICS                                              │
│  ─────────────────                                              │
│  Incidents per week by severity                                 │
│  └── Trend: Decreasing                                          │
│  Repeat incidents (same root cause)                             │
│  └── Target: 0                                                  │
│                                                                  │
│  IMPACT METRICS                                                 │
│  ──────────────                                                 │
│  User-impacting minutes per month                               │
│  └── Target: < [defined threshold]                              │
│  Revenue impact per incident                                    │
│  └── Trend: Decreasing                                          │
│                                                                  │
│  PROCESS METRICS                                                │
│  ───────────────                                                │
│  Postmortem completion rate                                     │
│  └── Target: 100% for P1/P2                                     │
│  Action item completion rate                                    │
│  └── Target: 90% within due date                                │
│  Postmortem published within SLA                                │
│  └── Target: 100%                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Metric Definitions

| Metric | Definition | Calculation |
|--------|------------|-------------|
| MTTD | Mean Time to Detect | (Detection Time - Incident Start Time) averaged |
| MTTA | Mean Time to Acknowledge | (Ack Time - Alert Time) averaged |
| MTTR | Mean Time to Resolve | (Resolution Time - Incident Start Time) averaged |
| MTBF | Mean Time Between Failures | Total uptime / Number of incidents |
| Incident Rate | Incidents per time period | Count of incidents / Time period |
| SLA Compliance | % of incidents within SLA | (Incidents meeting SLA / Total incidents) * 100 |

### Monthly Incident Report Template

```markdown
# Monthly Incident Report - [Month Year]

## Summary

| Metric | This Month | Last Month | Trend |
|--------|------------|------------|-------|
| Total Incidents | | | |
| P1 Incidents | | | |
| P2 Incidents | | | |
| P3 Incidents | | | |
| Average MTTR | | | |
| SLA Compliance | | | |
| User-Impacting Minutes | | | |

## Incident Summary

| ID | Date | Severity | Duration | Root Cause | Status |
|----|------|----------|----------|------------|--------|

## Top Issues

1. [Most impactful issue]
2. [Second most impactful]
3. [Third most impactful]

## Action Item Progress

| Total Open | Closed This Month | Overdue |
|------------|-------------------|---------|

## Recommendations

[Recommendations for reducing incidents]
```

---

## Runbooks for Common Incidents

### Runbook 1: Smart Contract Exploit

```markdown
# Runbook: Smart Contract Exploit Response

## Severity: P1 - Critical
## Auto-page: Security Team, CTO, Engineering Lead

### Symptoms
- Unusual token transfers from contract
- Large withdrawals
- Abnormal contract state changes
- Community reports of lost funds

### Immediate Actions (First 5 Minutes)

1. **PAUSE ALL CONTRACTS**
   ```
   Command: [Pause command or admin panel action]
   Verification: Check contract state on explorer
   ```

2. **Alert Key Personnel**
   - [ ] Security Lead
   - [ ] CTO
   - [ ] CEO
   - [ ] Legal (if funds lost confirmed)

3. **Preserve Evidence**
   - [ ] Screenshot current contract state
   - [ ] Note all suspicious transaction hashes
   - [ ] Do NOT interact with attacker address

### Investigation Phase (5-60 Minutes)

1. **Identify Attack Vector**
   - Review recent transactions
   - Analyze contract interactions
   - Check for known vulnerability patterns

2. **Assess Damage**
   - Total funds at risk
   - Funds already lost
   - Affected users

3. **Trace Funds**
   - Track attacker addresses
   - Identify if funds bridged or swapped
   - Contact Chainalysis/TRM Labs if needed

### Containment Actions

| Action | Command/Process | Owner |
|--------|-----------------|-------|
| Pause contracts | [Command] | Engineering |
| Block addresses | [Process] | Engineering |
| Disable frontend | [Process] | DevOps |
| Notify exchanges | [Contact list] | Business Dev |

### Communication

**Internal:**
- Update #incident channel every 15 minutes
- Executive briefing within 1 hour

**External:**
- Status page: Update immediately
- Social media: Within 1 hour (CEO approval required)
- Affected users: Within 4 hours

### Recovery

1. Deploy patched contracts (after audit)
2. Migrate user funds to new contracts
3. Consider user compensation
4. Resume operations gradually

### Post-Incident

- [ ] Comprehensive postmortem required
- [ ] External audit of fix required
- [ ] Legal review of response
- [ ] Consider bug bounty reward if whitehat
```

### Runbook 2: Oracle Failure

```markdown
# Runbook: Oracle Failure Response

## Severity: P2 - High
## Auto-page: Engineering On-Call

### Symptoms
- Market prices not updating
- Resolution data missing or incorrect
- UMA oracle requests timing out
- Abnormal price discrepancies

### Immediate Actions (First 5 Minutes)

1. **Verify Oracle Status**
   - [ ] Check UMA oracle contract status
   - [ ] Verify data feed endpoints
   - [ ] Check for network-wide issues

2. **Assess Impact**
   - [ ] Which markets affected?
   - [ ] Any pending resolutions blocked?
   - [ ] Trading impacted?

### Diagnostic Steps

```
1. Check oracle contract health
   - Transaction history
   - Latest successful response
   - Error logs

2. Check data source
   - Primary source availability
   - Fallback source availability
   - Data freshness

3. Check network
   - Polygon network status
   - Gas prices
   - Block time
```

### Resolution Steps

| Issue Type | Resolution |
|------------|------------|
| Data source down | Switch to fallback source |
| Network congestion | Increase gas, wait |
| Contract paused | Contact UMA team |
| Data discrepancy | Manual verification + escalate |

### Communication

- Update status page if trading impacted
- Notify support team of potential user questions
- Alert market ops about resolution delays

### Recovery Verification

- [ ] Oracle responding normally
- [ ] Data freshness within tolerance
- [ ] No backlog of failed requests
- [ ] Test market creation/resolution

### Post-Incident

- [ ] Document root cause
- [ ] Review fallback procedures
- [ ] Consider additional redundancy
```

### Runbook 3: High Latency / Performance Degradation

```markdown
# Runbook: High Latency Response

## Severity: P2/P3 (based on impact)
## Auto-page: DevOps On-Call

### Symptoms
- API response times > 500ms (P95)
- Page load times > 3 seconds
- User complaints about slowness
- Increased error rates

### Immediate Diagnosis

```
1. Check Dashboard Metrics
   - API latency by endpoint
   - Database query times
   - Memory/CPU usage
   - Network throughput

2. Identify Bottleneck
   - [ ] Database (high query times)
   - [ ] Application servers (high CPU)
   - [ ] Network (high latency to users)
   - [ ] External services (API timeouts)
   - [ ] Traffic spike (unexpected load)
```

### Common Causes and Fixes

| Cause | Indicators | Fix |
|-------|------------|-----|
| Traffic spike | Sudden volume increase | Scale up instances |
| Slow database query | High DB CPU, slow queries | Identify + optimize query |
| Memory leak | Increasing memory usage | Restart affected services |
| External API slow | Timeout errors | Enable caching, fallbacks |
| CDN issues | High origin traffic | Check CDN config |

### Scaling Actions

```
# Scale application servers
[Command to scale]

# Scale database read replicas
[Command or process]

# Enable rate limiting
[Command or toggle]

# Activate CDN fallback
[Process]
```

### Monitoring During Incident

- [ ] API latency returning to normal
- [ ] Error rate decreasing
- [ ] No new user complaints
- [ ] Resource usage stabilizing

### Post-Incident

- [ ] Document cause and fix
- [ ] Review auto-scaling thresholds
- [ ] Add new monitoring if gap found
- [ ] Capacity planning review
```

### Runbook 4: Database Issues

```markdown
# Runbook: Database Issues

## Severity: P1/P2 (based on impact)
## Auto-page: DBA + Engineering On-Call

### Symptoms
- Connection pool exhausted
- Query timeouts
- Replication lag
- Database unresponsive

### Immediate Actions

1. **Check Database Status**
   ```
   - Connection count
   - Active queries
   - Replication status
   - Disk space
   - CPU/Memory
   ```

2. **If Connection Pool Exhausted**
   - Identify connections by application
   - Kill idle connections if safe
   - Check for connection leaks

3. **If Replication Lag**
   - Check replica health
   - Assess read traffic distribution
   - Consider failover if lag critical

### Emergency Procedures

| Issue | Action |
|-------|--------|
| Primary down | Failover to replica |
| Disk full | Emergency cleanup, add storage |
| Connections exhausted | Kill idle, increase pool |
| Corruption detected | Stop writes, assess backup restore |

### Data Protection

**CRITICAL: Never delete data without backup verification**

- [ ] Verify latest backup exists
- [ ] Confirm point-in-time recovery available
- [ ] Document all changes made

### Recovery Steps

1. Stabilize the database
2. Address root cause
3. Verify data integrity
4. Restore normal operations
5. Monitor for recurrence

### Post-Incident

- [ ] Review backup status
- [ ] Capacity planning
- [ ] Query optimization review
- [ ] Connection pool tuning
```

---

## Appendix: Quick Reference

### Emergency Contact List

| Role | Name | Phone | Slack |
|------|------|-------|-------|
| Engineering On-Call | [Rotation] | [PagerDuty] | @eng-oncall |
| Security Lead | [Name] | [Phone] | @security |
| CTO | [Name] | [Phone] | @cto |
| CEO | [Name] | [Phone] | @ceo |
| Legal Counsel | [Name/Firm] | [Phone] | N/A |
| PR Contact | [Name] | [Phone] | @pr |

### Critical System Links

| System | URL | Credentials |
|--------|-----|-------------|
| PagerDuty | [URL] | SSO |
| Status Page | [URL] | Shared credentials |
| AWS/GCP Console | [URL] | SSO |
| Database Admin | [URL] | Vault |
| Smart Contract Admin | [URL] | Multi-sig |

### Severity Quick Reference

| Severity | Response | Resolution | Page |
|----------|----------|------------|------|
| P1 | Immediate | 2 hours | All |
| P2 | 15 min | 4 hours | Engineering |
| P3 | 1 hour | 24 hours | On-call |
| P4 | 4 hours | Best effort | On-call |

---

*Document Version: 1.0*
*Last Updated: [Date]*
*Owner: Engineering Team*
*Review Cycle: Quarterly*
