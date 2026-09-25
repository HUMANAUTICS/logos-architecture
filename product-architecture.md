# Logos Tax Systems — Product Architecture
_Last updated: 2026-08-13_

## Brand and Positioning

**Logos Tax Systems** is a portable tax operating system for business owners — powered by AI, executed by CPAs, owned by the business owner forever.

The core problem: tax history and financial context live inside a CPA's systems. When a business owner switches advisors, they start from zero. Logos flips this — the owner owns their data; CPAs get scoped access; when an engagement ends, the owner keeps everything.

Full business context: [`projects/logos-platform/context/business.md`](../projects/logos-platform/context/business.md)

---

## Value Chain

```mermaid
flowchart LR
    Citadel --> Logos
    Vault --> Logos
    Logos --> Enchiridion
    Enchiridion --> Marketplace
    Marketplace --> CPAPlatform
    CPAPlatform --> Ledger
    Ledger -->|"portable forever"| BO[BusinessOwner]
```

1. **Citadel** — Connect bank accounts, QuickBooks, payroll, and other financial data sources.
2. **Vault** — Upload tax documents and save AI sessions (source material).
3. **Logos** — AI analyzes connected data and documents; conversational advisor surfaces insights.
4. **Enchiridion** — Strategy playbook: synthesized, actionable tax plan with savings, steps, and legal refs. The value-delivery moment.
5. **CPA Marketplace** — Find and engage a CPA to implement strategies.
6. **Ledger** — Permanent, portable tax event timeline — the spine of the product.

---

## Five Product Pillars

| Pillar | What it is | Route(s) | Repo | Maturity |
|--------|-----------|----------|------|----------|
| **Citadel** | Integrations hub — bank (Plaid), QBO, payroll, tax software, IRS. Data intake layer. | `/citadel` | logos-app | UI shipped; Plaid inline connect not yet wired (links out to `/dashboard/quickbooks`) |
| **Logos** | Conversational AI tax advisor. Analyzes documents + financial data. | `LogosWidget` (global), `/logos`, `/logos/history` | logos-app + logos-backend + logos-agents | Widget + full page shipped |
| **Enchiridion** | Strategy playbook — recommended strategies, savings, implementation steps, legal references. | `/enchiridion` | logos-app | Metered gating: first strategy shown fully; remaining blurred with lock icon (strategyPreviewLimit) |
| **Ledger** | Portable tax event timeline. Business events + CPA implementation events. | `/ledger` | logos-app | Exists; CPA co-write implementation events partial |
| **Vault** | Saved AI sessions + uploaded documents. Feeds Logos. | `/vault`, `/dashboard/documents` | logos-app | Validated ingestion shipped — `validateClaim.ts` content-claim check, human approve/reject gate, fact reconciliation |

### Pillar details

**Citadel** (`components/citadel/CitadelPage.tsx`) — Shows integration cards for Plaid, QuickBooks, payroll, and tax software. Explains how connected data flows into Logos and Enchiridion for precise savings calculations.

**Logos** — Two surfaces:
- **Floating widget** (`components/LogosWidget.tsx`) — Pill → inline chat panel on most pages. "Full page" link opens `/logos`.
- **Full page** (`/logos`) — Split layout: session history sidebar + conversation.

**Enchiridion** (`components/enchiridion/EnchiridionPage.tsx`) — Tax position summary, strategy cards with estimated savings, implementation steps, legal references, and strategy timeline preview. `TaxPositionSummary` routes users here from Logos.

**Ledger** — Permanent record of tax life events. New CPA reads the Ledger and picks up where the last one left off.

**Vault** — Document storage and saved Logos sessions. Not generic file storage — exists to feed AI and give CPAs context.

---

## Three Surfaces (+ Platform Admin)

| Surface | Domain | Repo | Local port |
|---------|--------|------|------------|
| Marketing site | logostaxsystems.com | logos-website | 3002 |
| Business owner app | app.logostaxsystems.com | logos-app | 3003 |
| CPA portal | cpa.logostaxsystems.com | logos-cpa | 3001 |
| Platform admin | admin.logostaxsystems.com | logos-admin | 3004 |

All four share **logos-backend** (NestJS backend) and **Supabase** (`yhrbwyarzrjwgkthojzu`). Business owner data is readable by matched CPAs with explicit permission. Platform admin is cross-tenant ops for Logos staff only.

---

## UX Architecture (logos-app)

### Global navigation

`AppHeader` (`components/AppHeader.tsx`) — fixed top nav on authenticated pages:

| Link | Route |
|------|-------|
| Dashboard | `/dashboard` |
| Citadel | `/citadel` |
| Ledger | `/ledger` |
| Enchiridion | `/enchiridion` |
| Vault | `/vault` |
| Advisors | `/dashboard/my-cpas` |

Tagline under logo: *Citadel · Ledger · Enchiridion · Vault*

### Logos widget rules

`LogosWidget` is mounted in `app/layout.tsx` and shown on all pages **except**:

- `/strategies` (and sub-routes)
- `/logos` (and sub-routes)
- `/onboarding` (and sub-routes)

Rationale: Strategy agent and full Logos page are dedicated surfaces; onboarding should not compete with floating chat.

### SMS Channel (Logos on SMS)

Added 2026-07-31. Business owners can text Logos directly from any phone:

| Component | Function |
|-----------|----------|
| `/api/sms/link-phone` | Link E.164 phone + TCPA consent to account |
| `/api/sms/inbound` | Twilio webhook — signature validation, identity lookup, chunked Logos response |
| `ProfileEditForm` SMS Connect card | Phone input, TCPA consent checkbox, link/unlink |
| `sms_identities` table | Maps phone → user; enforces TCPA opt-in |

Logos SMS shares history with web Logos (`LogosChatMessage`) — one conversation across channels.

### Find-Advisor Marketplace (`/dashboard/find-advisor`)

Added 2026-07-21. Business owners browse and engage CPAs directly from the app:

| Route | Function |
|-------|----------|
| `/dashboard/find-advisor` | Browse marketplace — CPA cards with specializations |
| `/dashboard/find-advisor/[cpaId]` | CPA detail — profile, rates, reviews; send engagement request |
| `POST /api/engagement/request` | Create engagement request |
| `GET /api/engagement/advisor/[cpaId]` | Fetch CPA detail for engagement flow |

Replaces the old flow where engagement was initiated only from `/dashboard/my-cpas`.

CPA advisor ranking (added 2026-08-03): matching score algorithm + AI Recommended Match badge shown on browse page.

### Strategy agent (separate from Logos)

The **Strategy agent** is the recommendation engine on `/strategies` only. No Logos widget on that page. Strategies are AI-generated recommendations based on the owner's financial profile — not generic advice.

### Dashboard

`/dashboard` (`components/dashboard/BusinessOwnerDashboard.tsx`) — Journey-oriented home with:
- **Estimated Opportunity hero** — total savings potential from all recommended strategies
- **Contextual Next Step CTA** — guides the owner through: onboarding → Logos → Enchiridion → CPA
- **5 pillar cards** (Citadel, Logos, Enchiridion, Ledger, Vault) with live stats
- **Recent Activity feed** — strategy recommendations + Ledger events combined
- `AccountGuidanceStrip` — contextual guidance bar

### Account Settings (`dashboard/(account)/`)

Restructured under a Next.js route group with shared layout:

| Sub-route | Function |
|-----------|----------|
| `profile` | `ProfileEditForm` — personal + tax profile fields |
| `subscription` | Subscription management; Stripe cancel auth-gated to session owner |
| `settings/notifications` | `NotificationsPanel` |
| `settings/security` | `SecurityPanel` |
| `settings/privacy` | Privacy controls |
| `settings/data-sharing` | Data-sharing preferences |
| `settings/usage` | `UsagePanel` — quota display |
| `settings/support` | `SupportPanel` → `POST /api/support` → Supabase `support_requests` table |

---

## End-to-End User Journey

A first-time business owner's path:

1. **Sign up + Onboarding** — Three-step flow (`/onboarding/step1`, `step2`, `step3`). Cream/navy/gold branding. Logos generates a seed insight on completion.
2. **Citadel** — Connect bank accounts (Plaid), QuickBooks, payroll. Real financial data flows in.
3. **Vault** — Upload tax documents (prior returns, formation docs, IRS letters).
4. **Logos** — AI analyzes connected data + documents. Chat surfaces tax position and optimization signals.
5. **Enchiridion** — Synthesized strategy playbook: savings, implementation steps, legal refs. What the owner takes to a CPA.
6. **CPA Marketplace** — Find a CPA filtered by strategy needs. Send engagement request with context.
7. **CPA implements** — CPA uses Enchiridion as the brief, implements strategies, files returns, adds Ledger entries.
8. **Ledger** — Everything recorded. Owner switches CPAs without losing history.

**Key insight:** Enchiridion is the value-delivery moment. Without it, users have conversations (Logos) and timelines (Ledger) but no actionable plan.

---

## CPA Engagement Flow (MVP)

### Shipped (2026-06-17)

| Surface | Implementation |
|---------|----------------|
| BO engagement list | `/dashboard/my-cpas` |
| List engagements | `GET /api/engagement/requests` — merges `member_requests` + `cpa_clients` + `cpa_profiles` |
| End engagement | `POST /api/engagement/end` — body: `{ cpa_id }`, marks `cpa_clients.status = "ended"` |
| Database | `supabase/migrations/20260617_initial_schema.sql` (11 tables) |

Engagement statuses: `request_status` (pending / accepted / declined) and `engagement_status` (active / paused / ended).

### Not yet shipped

- Engagement letter upload (CPA uploads PDF → Vault → BO can view)
- CPA-side end-engagement UI + engagement letter upload
- Data bridge: CPA seeing BO strategies/ledger in logos-cpa
- Enchiridion → marketplace deep link filtered by specialization
- RLS: BO read policy on `cpa_clients`

---

## AI Agents (Product View)

| Agent | User-facing role | Where |
|-------|-----------------|-------|
| **Logos** | Conversational tax advisor for authenticated users | `LogosWidget` + `/logos` + SMS (`/api/sms/inbound`) |
| **Strategy agent** | Tax strategy recommendation engine | `/strategies` only |
| **IRC Tax Advisor** | Anonymous pre-sales tax Q&A (no login) | logos-website public pages; backed by `tax-advisor` module in logos-backend |
| **logos-tax-advisor** | ADK agent (IRC RAG + strategy tools) | logos-agents (`agents/logos-tax-advisor/`) — Vercel/pgvector, not GCP Cloud Run |
| **Community Agent** | Public demo chat for outreach events | `/community` on logos-website; 2-turn limit; lead capture; proxies to logos-backend community-leads API |

Logos and IRC Tax Advisor are intentionally separate: Logos is in-app and profile-aware; IRC Tax Advisor is public pre-sales with anonymous Redis sessions (24h TTL, transfer to 30d on auth). Community Agent is a third public surface, lighter than IRC Tax Advisor, focused on event demo + lead gen.

---

## Naming Conventions

| Name | Meaning |
|------|---------|
| Logos Tax Systems | Brand / product family |
| Citadel | Integrations hub |
| Logos | AI tax advisor (chat) |
| Enchiridion | Strategy playbook (Greek: "handbook") |
| Ledger | Portable tax event timeline |
| Vault | Documents + saved AI sessions |
| logos-app | Internal repo name for the business owner app |
| logos-cpa | CPA-facing application |

---

## Open Product Gaps (2026-08-13)

| Gap | Status |
|-----|--------|
| CPA access bridge — matched CPA accessing BO data in main app | Shipped — `context_access_granted` toggle gates Tax Profile + Ask Logos; CPA "Ask Logos" tab live |
| Engagement letter flow | Shipped in CPA platform — `ClientDocuments` with upload + private storage |
| Strategy-to-Marketplace CTA from Enchiridion | Not built |
| Ledger CPA implementation events | Partial — BO events exist; CPA co-write not complete |
| Enchiridion real profile data | Pending — connect to Supabase profile (filing posture, income) |
| Citadel inline Plaid connect | Partially addressed — Plaid BFF layer added; inline connect widget still pending |
| Enchiridion → marketplace deep link | Not built (find-advisor marketplace exists but no direct CTA from Enchiridion) |
| SMS Phase 2 | Shipped: Phase 1 (inbound/outbound). Phase 2: security hardening (brute-force lockout, OTP rate limiting, idle re-auth — also shipped Aug 2026) |
| IRS forms (2848, 8821) | Out of scope for MVP |

---

## What We Are NOT

- Not a licensed tax advisor — AI generates strategies to discuss with a CPA
- Not a CPA firm — marketplace CPAs are independent professionals
- Not generic document storage — Vault feeds AI and gives CPAs context

---

## Monetization (four-tier gating)

Org-scoped subscriptions in logos-backend `PLAN_CATALOG`: `free` → `professional` → `business_pro` → `enterprise`.

| Tier | Highlights |
|------|------------|
| **Free** | Logos Q&A, transaction categorization, audit alerts; no Plaid/QBO; browse marketplace |
| **Professional** | Plaid + QBO, deductions, Enchiridion (`advisor_brief`), quarterly estimates, CPA engagement requests |
| **Business Pro** | Compliance checklist, anomaly detection, multi-state, soft cap / overage |
| **Enterprise** | `api_access`, `white_label`, firm integrations, 25 members |

Enforcement: `EntitlementsService` on API routes; therapy `FeatureGate` UI; CPA `PlanGate` on branding/integrations/transfers.

OpenSpec: [`projects/logos-platform/design/changes/subscription-gating/`](../projects/logos-platform/design/changes/subscription-gating/)

---

## Related Docs

- [Architecture Viewer](./viewer.html) — HTML artifact: full system map + all diagrams
- [Platform System Map](./platform-system-map.md) — source Mermaid for the full repo diagram
- [Technical Architecture](./technical-architecture.md) — repos, APIs, AI layer, infrastructure
- [Business Context](../projects/logos-platform/context/business.md) — pitch, GTM, engagement process detail
- [Active Projects](../context/active-projects.md) — current phase and shipped work
