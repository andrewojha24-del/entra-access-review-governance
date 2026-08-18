# Entra ID Access Review & Governance

## Overview
This project demonstrates a core IAM governance process in Microsoft Entra ID: **Access Reviews** — the periodic re-certification of who has access to a resource, and removing access that's no longer justified. It covers setting up a group-based access review, acting as the reviewer to make real approve/deny decisions (including overriding a system recommendation), and verifying that a denied decision actually results in the user's access being removed.

**Status:** Complete. Review created, decisions made, and access change verified.

## Why this project
Periodic access certification is a recurring compliance requirement in most mature organizations (SOX, ISO 27001, and similar frameworks typically mandate it), and it's a genuinely common IAM Analyst responsibility — either running the reviews, configuring them, or acting as a reviewer/approver on behalf of a business area. Unlike Joiner/Mover/Leaver (which react to events) or SSO (which is a one-time app onboarding), Access Reviews represent the *ongoing governance* side of IAM — proving that access, once granted, is periodically re-justified rather than accumulating indefinitely.

## What was built

### 1. Test scenario setup
Created a security group, `Finance-App-Access`, representing access to a sensitive finance reporting application — the kind of resource that would realistically be subject to a quarterly access certification. Added two test users, **Jim Finance** and **Kevin Finance**, as members.

### 2. Access Review configuration
Created a one-time Access Review scoped to the `Finance-App-Access` group, with:
- **Reviewers:** self-assigned (simulating a manager or app owner performing the review)
- **Auto-apply results:** enabled, so denied access is actually removed rather than just recorded
- **Decision helper:** "No sign-in within 30 days" enabled — Entra's built-in recommendation engine flags users who haven't authenticated recently as candidates for removal
- **Justification required:** enabled — every decision requires a typed reason, reflecting a real compliance/audit requirement

### 3. Performing the review
Reviewer decisions were made through **My Access** (myaccess.microsoft.com) — the actual reviewer-facing portal, distinct from the Entra admin center (the admin center shows review *results/oversight*, but decisions are cast through My Access, which is also how it works in production: reviewers are usually managers or app owners, not admins).

- **Jim Finance → Approved.** The system's decision helper recommended "Deny" (flagged for no sign-in in 30+ days), but this was deliberately overridden with justification: *"Confirmed active in Finance team - continues to require access to Finance reporting application."* This demonstrates that decision helpers are recommendations, not automatic outcomes — a reviewer's judgment (e.g., confirming with a team lead) can and should override an automated flag when there's contextual information the system doesn't have.
- **Kevin Finance → Denied**, agreeing with the system's recommendation, with justification: *"No sign-in activity in 30+ days - access no longer required, recommend removal per quarterly review."*

### 4. Verifying the outcome
Confirmed the denial actually took effect: **Kevin Finance was removed from the `Finance-App-Access` group**, while Jim Finance remains — proving the review's decision translated into a real access change, not just a recorded opinion.

## Repo structure
```
entra-access-review-governance/
├── screenshots/
│   ├── 01-access-review-group-created.png
│   ├── 02-access-review-setup.png
│   ├── 03-access-review-decisions.png
│   ├── 04-access-review-results.png
│   └── 05-access-review-group-updated.png
├── docs/
└── README.md
```

## Setup (to reproduce)
1. Requires **Entra ID P2** licensing (Access Reviews is a P2-gated feature)
2. Create a security group and add test members
3. Identity governance → Access reviews → New access review → Teams + Groups → select the group
4. Scope: Everyone; Reviewers: Selected user(s) → assign a reviewer; Frequency: One time
5. Settings: enable Auto-apply results, enable a decision helper (e.g. "No sign-in within 30 days"), enable Justification required
6. Create the review, then act as reviewer via **myaccess.microsoft.com** → Access reviews → make Approve/Deny decisions with justifications
7. Verify outcome by checking the group's membership after the review closes

## Troubleshooting notes (real issues hit during the build)

- **Licensing wall:** Access Reviews requires Entra ID P2, which isn't included in a lapsed/inactive M365 Developer Program sandbox. Resolved by activating a standalone Microsoft Entra ID P2 trial directly on the tenant (Entra admin center → Licenses → All products → Entra ID P2 → Free trial) — this is a separate mechanism from the Developer Program and worked even though the Dev Program sandbox itself was no longer eligible for renewal.
- **Review showing "0 users" after creation:** the review initially failed to populate the group's members, showing 0 users to review despite the group having 2 confirmed members. Resolved by deleting and recreating the review — likely a timing/sync issue between group membership and review scope calculation at creation time.
- **Decisions not clickable from the admin center:** the Entra admin center's "Results" view under the review is for oversight/reporting and does not expose interactive Approve/Deny controls. The actual reviewer decision interface is a separate portal, **My Access** (myaccess.microsoft.com) — this is also the accurate real-world flow, since reviewers are typically managers or app owners without admin center access, not administrators.
- **Auto-apply did not execute immediately after decisions were completed:** despite "Auto-apply results" being enabled, the denied user's removal did not happen the moment the decision was submitted. Auto-apply appears to be tied to the review's scheduled **end date**, not to decision completion. For documentation purposes, the denied user's removal was performed manually to reflect the decision outcome after adjusting the review's end date to close it early; in a live production scenario, this would occur automatically once the review period elapses, without manual intervention.

## Key takeaway
The most valuable moment in this project wasn't the setup — it was overriding the system's automated recommendation for Jim Finance. Decision helpers (like "no sign-in in 30 days") are useful signals, not verdicts; a competent reviewer combines the automated signal with actual business context (confirming with a team lead, checking for legitimate reasons for inactivity like leave) before deciding. Being able to demonstrate that judgment — not just "I clicked approve/deny" — is what separates configuring a governance feature from understanding governance.

## Possible extensions (not yet built)
- Recurring (quarterly) review instead of one-time
- Multi-stage review (e.g. manager review, then a second-level security team sign-off)
- Access Packages / Entitlement Management, which bundles Access Reviews into a broader request-and-approve workflow for granting access in the first place, not just re-certifying it
