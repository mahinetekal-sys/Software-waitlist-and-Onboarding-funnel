# Software-waitlist-and-Onboarding-funnel
OJT project sem 1 ( product dev)
# LaunchGate

**Software waitlist and onboarding funnel** with admin approval, launch capacity and role-based access.

> Join the waitlist → get reviewed → get approved → finish onboarding, with a clear status at every step and approvals that only the right people can make.

| | |
|---|---|
| **Track** | PST_PD_112 / Product Development (2026 Core) |
| **Duration** | 4 weeks (one-month OJT) |
| **Team** | Mahesh Netekal & Divya Mohanty |
| **Mentor** | Suhana Jamadar |
| **PRD version** | v0.1 |

---

## Overview

LaunchGate replaces spreadsheet-and-email waitlist handling for early-stage SaaS teams. Prospects sign up, Operations Approvers review them against a launch capacity, Founders watch the funnel, and approved prospects complete a guided onboarding checklist.

It is a deliberately small **front-end only** app (HTML, CSS, JavaScript, Bootstrap 5) running on sample data and browser LocalStorage. Funnel and approval rules live in separate, DOM-free JavaScript modules so automated tests can prove that only authorized roles can move a prospect through the funnel, and only in the allowed order.

**Guiding rule:** don't overbuild. Finish join → review → approve → onboard and the role-check tests first; stretch items only after that.

## Problem

- Founders can't see where prospects drop off between signup and first use.
- Approvers have no consistent way to decide who gets access first.
- Approved prospects get ad-hoc emails with no guided path to first use.

**Hypothesis:** one status-transition function that validates every change and role, plus a visible funnel status and a capacity-based approval queue, will remove out-of-order or unauthorized approvals and cut the effort of admitting prospects.

## Roles

| Role | Can | Cannot |
|---|---|---|
| **Prospect** | Join waitlist, view own status and queue position, redeem invite code, complete onboarding | Act on anyone else's entry |
| **Operations Approver** | View queue, filter, approve / reject / put on hold (with notes), reissue expired codes, view audit trail | Set capacity |
| **Founder** | Set launch capacity, view funnel dashboard and audit trail | Approve, reject or hold (attempts are blocked and logged) |

## Funnel and status rules

```
Waitlisted → Under Review → Approved → Onboarding → Activated
     │            │            │
     └────────────┴────────────┴──→ On Hold ⇄ Approved / Rejected
                                  Rejected (final)      Activated (final)
```

1. Email must be valid and unique (case-insensitive); required fields present.
2. Status changes **only** through one transition function. Allowed:
   - Waitlisted → Under Review
   - Waitlisted / Under Review / On Hold → Approved, On Hold or Rejected
   - Approved → Onboarding (invite redeemed)
   - Onboarding → Activated
   - Rejected and Activated are final; everything else is refused.
3. Only an Approver can approve/reject/hold; only the Founder sets capacity; a Prospect acts only on their own entry.
4. Approval needs a free seat. Seats used = entries in the batch that are Approved, Onboarding or Activated.
5. Reject and Hold require a note of at least 10 characters; approval note is optional.
6. Approval issues exactly one invite code, valid 7 days, single use. An Approver can reissue an expired, unused code (voids the old one).
7. Redeeming requires a code that belongs to that prospect, is unused and unexpired.
8. Onboarding steps (create workspace profile → choose template → invite teammates → finish product tour) are completed in order and can't be undone. Activated is set only when all are done.
9. Capacity is a positive whole number (max 500) and can't go below the number already approved.
10. Every decision, status change and blocked attempt writes an immutable audit event.

## Features

### MVP (graded)

- Demo role selection with role-specific navigation
- Landing page and waitlist form with validation and duplicate-email rejection
- Prospect status page (stage, queue position, next step)
- Approval queue with filters, prospect detail and decisions with required notes
- Founder-set launch capacity that approvals can never exceed
- Single-use, expiring invite codes
- Guided four-step onboarding ending in Activated
- Founder dashboard: counts and conversion per stage, derived from records
- Audit trail of decisions, status changes and blocked attempts
- One role guard function for every action
- Inline form validation with clear messages
- Sample data: every stage, a full batch, an empty batch, an expired code
- LocalStorage persistence with reset-sample-data option
- Responsive Bootstrap 5 layout (about 360 px / 768 px / 1280 px)
- Automated critical-path and role-check tests (or scripted demo checklist)

### Stretch (only if the demo already runs)

Bulk approve with capacity check · referral link to move up the queue · invite email preview · CSV export · Chart.js funnel chart · dark theme · React + Vite port (self-study)

### Out of scope

Real backend, database or API · real authentication · real email/SMS · payments · the actual SaaS product · third-party analytics · compliance certification

## Architecture

Logic modules are DOM-free and unit-tested in Node. UI components only call services.

| Module | Responsibility |
|---|---|
| `storage.js` | Versioned LocalStorage read/write, sample data load, reset |
| `funnelService.js` | Stage definitions, the single transition function, queue position |
| `approvalService.js` | Approve / reject / hold, capacity enforcement, audit event, invite trigger |
| `inviteService.js` | Generate unique single-use codes with expiry; redeem |
| `onboardingService.js` | Ordered checklist progress; activation |
| `metrics.js` | Funnel counts, conversion and drop-off from records |
| `validators.js` | Form validation (required, email, company size, duplicate, capacity) |
| `roleGuard.js` | Role → allowed actions; logs blocked attempts |

**UI components:** Navbar, WaitlistForm, StatusTracker, ApprovalTable, DecisionModal, FunnelChart, Toast.

**Design rules**
- Derived values (queue position, seats used, counts, conversion) are calculated, never stored.
- Role checks run **inside** services, not only in the UI.
- One replaceable clock function for expiry logic, so tests can control time.
- Decision functions re-read storage right before writing; UI refreshes on the `storage` event.

## Data model

`WaitlistEntry` · `Decision` · `LaunchBatch` · `InviteCode` · `OnboardingStep` · `AuditEvent` · `User (demo)`

**Error codes:** `ERR_FORBIDDEN`, `ERR_DUPLICATE_EMAIL`, `ERR_CAPACITY_FULL`, `ERR_INVALID_TRANSITION`, `ERR_CODE_USED`, `ERR_CODE_EXPIRED`, `ERR_STEP_ORDER`

## Tech stack

- HTML5, CSS3, JavaScript (ES6 modules), Bootstrap 5
- LocalStorage with JSON sample data and a version key
- Tests: Node.js built-in runner (`node:test`) or Jest
- Git and GitHub; deploy on GitHub Pages / Netlify / Vercel
- Tool versions: `<fill in exact versions>`

## Getting started

ES modules don't load from `file://`, so use a static server:

```bash
# Option 1
python -m http.server 8000

# Option 2: VS Code Live Server extension
```

Then open `http://localhost:8000`, pick a role on the demo login screen, and use **Settings → Reset sample data** if needed.

Run tests:

```bash
node --test
```

## Testing

Automated tests should cover at least:

- Duplicate email signup
- Invalid transition (e.g. Waitlisted → Activated)
- Prospect or Founder attempting to approve
- Approval beyond batch capacity
- Invite code used twice, expired code, another prospect's code
- Skipping an onboarding step
- A decision on a Rejected prospect
- A blocked attempt written to the audit trail

**Critical-path script:** join → approve → redeem → complete four steps → Activated.

## Acceptance criteria

1. Prospect can join and see status and queue position
2. Never two entries for the same email
3. Only Approvers can approve/reject/hold; other attempts are blocked and logged
4. Approvals never exceed batch capacity
5. One single-use, expiring invite code per approval; reused or expired codes rejected
6. Prospect can complete onboarding in order and reach Activated
7. Founder sets capacity; dashboard counts match records
8. Clear validation and success/error feedback
9. Usable on phone and laptop widths (no horizontal scroll at ~360 px)
10. Tests or checklist pass and results are documented

## Accessibility and UX

- Semantic HTML, labelled fields, keyboard-operable queue and filters, visible focus, 44 px touch targets
- Target WCAG 2.2 AA; statuses distinguished by text or icon, not color alone
- Always confirm Reject; show a summary (prospect, decision, seats left) before Approve
- Fictional sample data only, with a notice not to enter real personal data

## Milestones

| Week | Focus |
|---|---|
| 1 | Requirements, personas, journeys, wireframes, repo, Bootstrap shell, sample data design |
| 2 | Landing and signup, status page, approval queue and decisions with LocalStorage |
| 3 | Invite codes, onboarding, founder dashboard, role guard, audit trail, validation, tests; stretch only if demo runs |
| 4 | Responsive and accessibility pass, bug fixes, more tests, README, screenshots, deployment, demo |

## Evaluation rubric

| Area | Weight |
|---|---|
| Funnel logic, approvals and role checks (with tests) | 25% |
| Role flows | 20% |
| UI quality, responsiveness, accessibility | 20% |
| Code structure and data handling | 15% |
| Testing and validation | 10% |
| Documentation and demo | 10% |

## Known limitations

- LocalStorage is per-browser and not a secure database; a real product needs a server enforcing roles and uniqueness.
- Front-end role checks are for demonstration, not security.
- Invitations are simulated on screen only.
- Two tabs approving at once can rarely over-approve; mitigated by re-reading storage before writes.

## Deliverables

PRD and journey diagrams · wireframes · this repo with clean commits · role-based app · sample dataset · tests with results · README with screenshots · deployed or local demo · demo video or live presentation
