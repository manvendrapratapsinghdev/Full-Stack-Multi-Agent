# Requirement Traceability Matrix
## Maintained by: Root_Requirement_Agent

**Version:** 2.1 (Task IDs backfilled by Task_Breaking_Agent)
**Date:** 2026-03-27
**Source:** PRD_Analysis.md v2.0 -> Master_TRD.md v2.0.0 -> Sprint_Task_List.md v2.0

Traces: PRD Section -> Feature ID -> TRD Spec(s) -> Sprint Task(s) -> QA Task(s)

---

## Full Traceability Matrix

| PRD Section | Feature ID | Feature Title | TRD Spec(s) | Priority | State | Sprint Task(s) | QA Task(s) |
|-------------|-----------|---------------|-------------|----------|-------|----------------|------------|
| PRD-4 | F-001 | User Authentication (HT SSO) | TRD-AUTH-001, TRD-AUTH-002 | Must | Approved | S1-01, S1-02, S1-06, S1-07 | S1-10, S1-11 |
| PRD-4 | F-002 | Creator Level System | TRD-AUTH-003, TRD-AUTH-004 | Must | Approved | S1-03, S1-04, S1-08, S6-09 | S1-10 |
| PRD-6 | F-003 | News Submission Form | TRD-SUB-001, TRD-SUB-002, TRD-SUB-004 | Must | Approved | S2-01, S2-03, S2-05 | S2-09, S2-10 |
| PRD-6 | F-004 | Multi-Image Upload | TRD-SUB-003 | Should | Approved | S2-04, S2-06 | S2-09 |
| PRD-7.1 | F-005 | AI Language Rephrasing with Preview | TRD-AI-001, TRD-AI-002 | Must | Approved | S3-01, S3-02, S3-10 | S3-12 |
| PRD-7.2 | F-006 | AI Safety Guardrails | TRD-AI-003, TRD-AI-004 | Must | Approved | S3-04, S3-05 | S3-12 |
| PRD-7.3 | F-007 | AI Tag and Category Suggestion | TRD-AI-005 | Should | Approved | S3-06, S3-11 | S3-12 |
| PRD-7.4 | F-008 | AI Duplicate Detection | TRD-AI-006 | Should | Approved | S3-07 | S3-12 |
| PRD-7.5 | F-009 | AI Editorial Confidence Score | TRD-AI-007 | Must | Approved | S3-08, S3-09 | S3-12 |
| PRD-8 | F-010 | Editor Review Queue | TRD-ED-001, TRD-ED-002 | Must | Approved | S4-01, S4-05 | S4-09, S4-10 |
| PRD-8 | F-011 | Editor Actions (Approve/Edit/Reject/Correct) | TRD-ED-003 | Must | Approved | S4-02, S4-06 | S4-09, S4-10 |
| PRD-8 | F-012 | False News Retroactive Handling | TRD-ED-004 | Must | Approved | S4-03, S4-07 | S4-09 |
| PRD-5.A, 9 | F-013 | News Feed ("News By You") | TRD-FEED-001, TRD-FEED-002, TRD-FEED-003 | Must | Approved | S5-01, S5-06, S5-07 | S5-12, S5-13 |
| PRD-9 | F-014 | Story Detail Page | TRD-FEED-004 | Must | Approved | S5-02, S5-08 | S5-13 |
| PRD-9, 12 | F-015 | Community Confirmation | TRD-COM-001 | Must | Approved | S6-01, S6-06 | S6-10, S6-11 |
| PRD-9 | F-016 | Like Stories | TRD-COM-002 | Should | Approved | S6-02, S6-06 | S6-10, S6-11 |
| PRD-10 | F-017 | Creator Reputation System | TRD-REP-001 | Must | Approved | S6-03, S6-09 | S6-10 |
| PRD-11 | F-018 | Creator Public Profile | TRD-REP-002, TRD-REP-003 | Should | Approved | S6-04, S6-07 | S6-10 |
| PRD-5.B | F-019 | "Posted by You" Dashboard | TRD-SUB-006 | Must | Approved | S2-07, S2-08 | S2-09 |
| PRD-5.A | F-020 | Trending Local Stories | TRD-FEED-005 | Could | Approved | S9-03, S9-06 | S9-09 |
| GAP-003 | F-021 | Push Notification System | TRD-NOTIF-001, TRD-NOTIF-002 | Must | Approved | S7-01, S7-02, S7-03, S7-05, S7-06 | S7-09, S7-10 |
| GAP-004 | F-022 | KYC Verification Flow | TRD-AUTH-005 | Must | Approved | S1-05, S1-09 | S1-12 |
| GAP-006 | F-023 | Content Moderation Appeal | TRD-ED-005 | Should | Approved | S4-04, S4-08 | S4-09 |
| GAP-011 | F-024 | Onboarding Flow | TRD-CROSS-003 | Should | Approved | S7-07 | S7-10 |
| GAP-009 | F-025 | Search Functionality | TRD-FEED-006 | Should | Approved | S5-03, S5-09 | S5-12 |
| GAP-010 | F-026 | Creator Blocking and Reporting | TRD-COM-003 | Should | Approved | S6-05, S6-08 | S6-10 |
| GAP-002 | F-027 | Image and Media Constraints | TRD-SUB-005 | Must | Approved | S2-02 | S2-09 |
| GAP-001 | F-028 | Offline Draft Support | TRD-SUB-007 | Should | Approved | S2-08 | S2-10 |
| GAP-012 | F-029 | Analytics and Tracking Events | TRD-PLAT-003 | Must | Approved | S8-06, S8-09 | S8-11 |
| GAP-008 | F-030 | Accessibility (WCAG 2.1 AA) | TRD-CROSS-001 | Must | Approved | S0-08, S9-07 | S9-11 |
| GAP-005 | F-031 | Internationalization (i18n) | TRD-CROSS-002 | Should | Approved | S7-04, S7-08 | S7-10 |
| GAP-007 | F-032 | Data Privacy and Retention | TRD-PLAT-004 | Must | Approved | S8-03, S8-04, S8-05, S8-08 | S8-10 |
| GAP-013 | F-033 | Rate Limiting and Abuse Prevention | TRD-PLAT-005 | Must | Approved | S8-01, S8-02, S8-07 | S8-10 |
| GAP-017 | F-034 | Editor Dashboard Analytics | TRD-PLAT-002 | Could | Approved | S9-02, S9-05 | S9-08 |
| GAP-014 | F-035 | Story Sharing | TRD-FEED-008 | Should | Approved | S5-05, S5-11 | S5-13 |
| GAP-015 | F-036 | Location-Based Feed Personalization | TRD-FEED-007 | Should | Approved | S5-04, S5-10 | S5-12 |
| GAP-016 | F-037 | Admin Moderation Panel | TRD-PLAT-001 | Must | Approved | S9-01, S9-04 | S9-08 |
| PRD-7.1 | F-038 | Multilingual AI Processing | TRD-AI-008 | Must | Approved | S3-01, S3-03 | S3-12 |

---

## MoSCoW Summary

| Priority | Count | Features |
|----------|-------|----------|
| Must | 23 | F-001, F-002, F-003, F-005, F-006, F-009, F-010, F-011, F-012, F-013, F-014, F-015, F-017, F-019, F-021, F-022, F-027, F-029, F-030, F-032, F-033, F-037, F-038 |
| Should | 13 | F-004, F-007, F-008, F-016, F-018, F-023, F-024, F-025, F-026, F-028, F-031, F-035, F-036 |
| Could | 2 | F-020, F-034 |
| Won't | 0 | -- |

---

## TRD Spec Index (by Module)

| Module | TRD Specs | Feature Coverage |
|--------|-----------|-----------------|
| AUTH | TRD-AUTH-001 through TRD-AUTH-005 | F-001, F-002, F-022 |
| SUB | TRD-SUB-001 through TRD-SUB-007 | F-003, F-004, F-019, F-027, F-028 |
| AI | TRD-AI-001 through TRD-AI-008 | F-005, F-006, F-007, F-008, F-009, F-038 |
| ED | TRD-ED-001 through TRD-ED-005 | F-010, F-011, F-012, F-023 |
| FEED | TRD-FEED-001 through TRD-FEED-008 | F-013, F-014, F-020, F-025, F-035, F-036 |
| COM | TRD-COM-001 through TRD-COM-003 | F-015, F-016, F-026 |
| REP | TRD-REP-001 through TRD-REP-003 | F-017, F-018 |
| NOTIF | TRD-NOTIF-001 through TRD-NOTIF-002 | F-021 |
| PLAT | TRD-PLAT-001 through TRD-PLAT-005 | F-029, F-032, F-033, F-034, F-037 |
| CROSS | TRD-CROSS-001 through TRD-CROSS-003 | F-024, F-030, F-031 |

**Total: 32 TRD specs covering 38 features across 10 modules**

---

## Dependency Tiers (from PRD Dependency Graph)

| Tier | Features | Description |
|------|----------|-------------|
| Tier 0 | F-001, F-030, F-031 | Foundation -- no dependencies |
| Tier 1 | F-002, F-003, F-022, F-024 | Core creation -- depends on Auth |
| Tier 2 | F-004, F-005, F-006, F-007, F-008, F-019, F-027, F-028, F-033, F-038 | AI processing + submission extras -- depends on Submission |
| Tier 3 | F-009, F-010, F-011, F-023 | Editorial -- depends on AI |
| Tier 4 | F-012, F-013, F-017, F-021, F-029 | Publishing -- depends on Editorial |
| Tier 5 | F-014, F-015, F-016, F-018, F-025, F-026, F-035, F-036 | Consumption -- depends on Feed |
| Tier 6 | F-020, F-032, F-034, F-037 | Advanced -- depends on multiple tiers |

---

## Clarifications Pending (blocks partial development)

| CLR ID | Topic | Blocking Features | Default Assumption Used |
|--------|-------|-------------------|------------------------|
| CLR-001 | Story merging scope | F-008 (partial) | V2 scope |
| CLR-002 | Creator edit after submission | F-003 | No post-submission editing |
| CLR-003 | Editor assignment model | F-010 | Random with city filter |
| CLR-004 | Confirmation geo constraint | F-015 | No constraint |
| CLR-005 | Reputation point values | F-017 | publish=10, confirm=2, like=1 |
| CLR-006 | HT Gifts definition | F-017 | Boolean eligibility flag only |
| CLR-007 | Creator anonymity | F-018 | Real name only |
| CLR-008 | Submission limits per level | F-033 | Basic=5, Approved=10, Trusted=20 |
| CLR-009 | Editor SLA | F-010 | No enforced SLA in V1 |
| CLR-010 | Web vs. Mobile | All | Mobile only |

---

**END OF RTM v2.0**
