# Prompt Guide: Building the Unified POS + ERP Platform with Claude Opus 5.5

This guide contains the prompts to give **Claude Opus 5.5** (in Claude Code or a similar agentic coding environment) to build the system described in `POS_Concept_Note.md`: a cloud-native, offline-first POS with ERP-grade back office for supermarkets, bars, resto-bars and cafés.

> **Updated for concept note v0.6.** The first market is **Rwanda** (concept note Section 25): the Project facts in `CLAUDE.md` are prefilled for Rwanda, and a new **RW** milestone delivers RRA electronic invoicing (EBM through the VSDC), Rwandan tax, mobile money, payroll rules and data residency. Finance, Inventory and Sales are the three core pillars of the system, built in that order through the core prompts **C1 (Finance), C2 (Inventory) and C3 (Sales)**. They sit on a **document engine (C0)** that manages every business document: requisitions, RFQs, quotations, proformas, purchase and sales orders, delivery notes, GRNs, invoices, credit and debit notes, payment vouchers, receipts and statements (concept note Section 24). If you already started building from the earlier version, go to **Part H** for the upgrade path.

It has eight parts:

| Part | What it is | When you use it |
|---|---|---|
| **A** | How to run the build | Read once before starting |
| **B** | `CLAUDE.md`, the project rules file | Copy into the new repository before the first session |
| **C** | Prompt 0: architecture and plan | First session |
| **D** | Core prompts C0–C3 and milestone prompts M0–M14 | One milestone per session, in the order in A.3 |
| **E** | Reusable prompts (verify, review, resume, fix, UI) | Any time |
| **F** | Quality gates and invariants | At the end of every milestone |
| **G** | Tips for working with Opus 5.5 | Keep at hand |
| **H** | Upgrade path for a build already in progress | Now, if you started with the earlier version |

---

## Part A. How to run the build

### A.1 Setup

1. Create a **new, empty repository** for the product (for example `unified-pos`). Don't build it inside a portfolio or documents repository.
2. Copy `POS_Concept_Note.md` into it as **`docs/concept-note.md`**. This is the specification. Every prompt below refers to it by section number (S1–S15, B1–B13, R1–R15, E1–E9, the core IDs F1–F19, IN1–IN11 and SA1–SA12, the document IDs DOC1–DOC12, the Rwanda IDs RW1–RW14, Section 5 deployment modes, Section 7 roles and usability, Sections 19–23 core pillars, Section 24 business documents, Section 25 Rwanda).
3. Copy Part B of this guide into the repository root as **`CLAUDE.md`**. Claude Code reads it automatically at the start of every session, so the rules persist across sessions.
4. Check the **"Project facts"** block at the top of `CLAUDE.md`. It is prefilled for Rwanda; complete the pilot sites and providers, and leave anything unknown as "TBD". Prompt 0 will list the open questions.
5. Choose model **Claude Opus 5.5**. Use effort **`xhigh`** for Prompt 0, the core milestones C0, C1, C2 and C3, the Rwanda milestone RW, and milestones M1, M4, M10 and M14 (architecture-heavy, correctness-critical). Use **`high`** for the others. Lower effort only for small fixes.

### A.2 Working rhythm

- **One milestone per session.** Start each session with the milestone prompt from Part D. When the milestone is done, open a pull request, review it, merge, then start a fresh session for the next milestone.
- **Plan first, then build.** Every milestone prompt asks Claude to post a short plan and any blocking questions before writing code. Answer the questions; say "proceed" when the plan looks right.
- **Keep the memory in the repo.** Claude maintains `docs/PROGRESS.md` (what's done, what's next, known issues) and `docs/adr/` (architecture decision records). If a session ends mid-milestone, start the next one with the **Resume** prompt (E.3).
- **Verify with fresh eyes.** At the end of each milestone, run the **Verify** prompt (E.1). It has a separate sub-agent with a fresh context check the work against the concept note. That catches more than self-review.
- **You stay the product owner.** Claude makes routine engineering decisions itself and asks only when the choice changes the product (for example, a tax rule, or which ERP is the system of record).

### A.3 Order of milestones

```
Prompt 0  Architecture, domain model, milestone plan (no code)
M0   Repository foundation, CI, dev environment
M1   Platform core: tenants, locations, identity, roles, permissions, approvals, audit
C0   DOCUMENT ENGINE: document types, numbering, lifecycle, conversion, flows,
     matching, payment terms, templates, e-invoicing interface (DOC1–DOC12)
C1   FINANCE CORE: posting engine, accounting books, ledgers, reconciliations,
     books health checks and error correction, close, owner view (F1–F19)
M2   Catalogue: item master, units, barcodes, tax, modifiers, recipes, price books
C2   INVENTORY CORE: perpetual stock ledger posted to the books, valuation, GRNI,
     counts and variance, inventory–finance reconciliation (IN1–IN11)  [replaces M3]
C3   SALES CORE: sales documents posted to stock and books, tenders and clearing,
     Z-close, quick reports, insight-to-task, three-way reconciliation (SA1–SA12)
RW   RWANDA COMPLIANCE PACK: EBM receipts via VSDC, RRA tax types, VAT return,
     WHT, mobile money over eKash, payroll rules, data residency, languages (RW1–RW14)
M4   POS terminal apps: offline-first selling UI on top of the Sales core
M5   Payments: provider abstraction, pre-auth tabs, tips, refunds, settlement postings
M6   Purchasing, receiving, supplier invoices, three-way match, AP postings
M7   Supermarket mode (S1–S15)
M8   Bar mode (B1–B13)
M9   Resto-bar and café mode (R1–R15)
M10  ERP integration hub (Modes 2 and 3)  [finance core moved to C1]
M11  Management layer: role dashboards, routines, tasks, staff app, scheduling
M12  Reporting, analytics and AI features
M13  Intelligence layer over existing POS systems (Mode 3)
M14  Hardening: security, performance, offline chaos tests, accessibility, usability, release
```

**Why this order.**
- **The document engine (C0) comes just before the cores** because every event in them starts from a business document. It's a framework only; each core wires its own documents.
- **The Rwanda pack (RW) comes right after the cores** because no terminal may issue a receipt in Rwanda without an RRA-signed EBM receipt.
- **Finance comes first** among the cores because it is the reference every other module must reconcile to.
- **Inventory comes second** because it holds most of the value and produces COGS.
- **Sales comes third** because it depends on both.

Only M1 (identity and permissions) and M2 (the catalogue that inventory needs) come before or between the cores. Milestone numbers are kept from the earlier version so existing references stay valid. M3 is replaced by C2.

The vertical modes (M7–M9) can be reordered to match your first customer (for example, bar before supermarket).

---

## Part B. `CLAUDE.md`, the project rules file

Copy everything inside the block below into `CLAUDE.md` at the repository root.

````markdown
# Unified POS + ERP Platform: Project Rules

## Project facts (first market: Rwanda; complete the TBDs)
- Countries / tax jurisdictions: Rwanda (first market; see docs/concept-note.md Section 25). Other countries later, as country packs.
- Tax authority and fiscal system: Rwanda Revenue Authority (RRA); electronic invoicing through EBM using an RRA-certified VSDC integration. Every sale and refund needs an RRA-signed receipt (NS/NR).
- VAT: 18% standard; RRA tax types A (exempt), B (18%), C (zero-rated/export), D (not VAT-registered). Returns monthly, or quarterly for turnover ≤ RWF 200 million.
- Currencies: RWF (0 decimal places) primary; USD optional.
- Languages (staff UI and receipts): Kinyarwanda, English, French (Swahili optional).
- Time zone: CAT (UTC+2), no daylight saving.
- Accounting standard: IFRS for SMEs (IFRS for publicly accountable entities).
- Record retention: at least 10 years for books, records, documents and EBM data.
- Data residency: personal data of Rwandan tenants stored in Rwanda unless authorised otherwise (Law 058/2021).
- Payments: mobile money first (MTN MoMo, Airtel Money; eKash interoperability and unified merchant codes); cards through a local acquirer: TBD which acquirer.
- First pilot sites: TBD (e.g. 1 supermarket, 1 bar/lounge, 1 resto-bar with café in Kigali)
- Priority ERP / accounting connectors: TBD (e.g. QuickBooks, Xero, Odoo, Sage)
- Items marked "Confirm" in concept note Section 25 must be verified with RRA or a Rwandan tax adviser before go-live.

## What we are building
A cloud-native, offline-first POS with an ERP-grade back office, for supermarkets, bars, resto-bars and cafés.
It must beat typical cloud POS products on back-office control, and beat ERP POS modules on speed and hospitality/grocery depth.
It runs in three deployment modes: standalone (Mode 1); POS front end on an existing ERP (Mode 2); intelligence layer over an existing POS and ERP (Mode 3).

**The specification is `docs/concept-note.md`.** Process IDs (S1–S15 supermarket, B1–B13 bar, R1–R15 resto-bar/café, E1–E9 back office) and Section 7 (roles, permissions, management routines, usability) are the functional requirements. When code implements a process, reference its ID in the module README and in tests.

## Core pillars: Finance → Inventory → Sales (concept note Sections 19–23)
- Finance, Inventory and Sales are the three core modules, built in that order. Every other module (payments, purchasing, supermarket/bar/resto-bar modes, people, management, reporting, AI, integrations) is built on them.
- Only the cores write money and stock. Other modules use the core contracts: `postJournal` (Finance), `recordStockMovement` (Inventory) and `completeSale` (Sales). No other module writes journal lines, stock movements or balances directly. An architecture test enforces this.
- One event, three views: a sale, its stock movements and its journal are committed together (one transaction or an exactly-once outbox). None exists without the others.
- The books of original entry and ledgers (concept note 20.2) are views over one journal store, not separate copies.
- Every journal links to a source document. Posted entries are never edited or deleted; corrections go through the correction wizard (reversal, reclassification, amount correction) with reason, approval and audit.
- A posting that fails is never dropped: it is held in the finance issue inbox and shown as "unposted" until fixed.
- The three-way reconciliation (23.3) runs nightly and in the test suite. Any failure is a bug or a finance issue, never ignored.
- In Rwanda, no sale or refund is complete without an RRA-signed EBM receipt from the VSDC (concept note 25.2–25.4); tax per RRA tax type must equal the VSDC's figures to the franc.
- Every core event starts from a typed business document from the document engine (concept note Section 24). Only billing, payment and adjustment documents post to the books; only delivery and stock documents move stock; commercial documents (PR, RFQ, quotation, proforma, PO, SO) never do either. Issued documents are never edited or deleted.
- Process IDs: F1–F19 (finance), IN1–IN11 (inventory), SA1–SA12 (sales), DOC1–DOC12 (documents), RW1–RW14 (Rwanda). Reference them in module READMEs and tests.

## Memory in the repository
- `docs/PROGRESS.md`: milestone status, what's done, what's next, known issues. Update it at the end of every work session and whenever a milestone task finishes.
- `docs/adr/NNNN-title.md`: one architecture decision record per significant decision (context, decision, alternatives, consequences).
- `docs/domain/`: domain model, glossary, state machines, invariants.
Read `docs/PROGRESS.md` and the relevant ADRs before starting work.

## Default technology stack (change only through an ADR that explains why)
- Language: TypeScript everywhere (strict mode). Python allowed only for isolated ML/forecasting services.
- Monorepo: pnpm workspaces + Turborepo. Shared packages for domain types, validation (Zod), UI components and the sync protocol.
- Cloud backend: Node.js LTS, NestJS (modular monolith first; module boundaries enforced; split into services only via ADR), PostgreSQL 16+ (row-level security for tenant isolation), Redis, a job queue, transactional outbox for events, OpenAPI for every public API.
- Back office and owner web app: React + Next.js.
- Terminal and handheld apps: React Native (Expo) for Android/iOS devices; Tauri shell for Windows/Linux lane terminals; shared React UI packages. Local database: SQLite. Full offline operation.
- Store edge hub (optional per site): Node service on a local machine that bridges printers (ESC/POS), KDS screens, scales (serial/USB), cash drawers and payment terminals, and relays sync when a terminal is offline from the cloud but online on the LAN.
- Infrastructure as code; Docker for local development; one-command dev setup.

## Domain rules (non-negotiable)
- **Money:** integer minor units plus ISO 4217 currency code. Never floating point. Rounding rules (per line vs per invoice, cash rounding) are configurable per jurisdiction and tested.
- **Quantities:** decimal with explicit unit of measure and fixed scale; conversions (case→each, bottle→ml, kg→g, keg→pint) defined in data, never hard-coded.
- **Immutability:** completed sales, payments, stock movements, journal entries and audit events are append-only. Corrections are reversals or adjustments, never updates or deletes.
- **Stock** is a ledger of movements (sale, receipt, transfer, adjustment, waste, production, count variance). On-hand is derived from the ledger.
- **IDs:** UUIDv7 generated on the client, so offline devices can create records safely. Every command carries an idempotency key.
- **Offline:** every terminal must sell, open and close tabs, take cash and (within configured limits) store-and-forward card payments with no internet. Sales made offline are never discarded; conflicts resolve by the rules in the sync ADR.
- **Business day vs calendar day:** a trading day can cross midnight (bars). Reports use the business day defined per location.
- **Receipt and invoice numbering:** sequences per terminal/location as required by fiscal rules; gapless where the law requires it.
- **Multi-tenant, multi-entity, multi-location, multi-currency, multi-language** from the start.
- **Permissions:** role templates + per-person overrides + thresholds (amount/percentage/count limits) + remote approval. Every sensitive action writes an audit event with actor, approver, device, location, reason and before/after values.
- **Card data never enters our systems.** Payment terminals are P2PE/E2EE; we store only tokens, masked PAN and processor references. PCI DSS v4.0 scope must stay minimal.
- **Personal data:** consent tracked, minimal collection, export and deletion supported, retention configurable.

## Engineering standards
- Module structure follows the domain (catalogue, inventory, sales, payments, purchasing, finance, people, integration, reporting, ai). No cross-module database access; use module APIs or events.
- Validation at every boundary (Zod / class-validator). Clear, typed errors.
- Tests: unit tests for domain logic, property-based tests for money/tax/stock/sync invariants, integration tests against real PostgreSQL (Testcontainers), end-to-end tests for key flows per role, and contract tests for connectors.
- Keep tests focused: test behaviour and invariants that matter. Don't commit scratch or exploratory scripts as tests.
- Lint, type-check and tests must pass before a task is called done. CI runs them on every push.
- Seed data: a realistic demo tenant with one supermarket, one bar, one resto-bar with a café counter, and staff in every role from concept-note Section 7.
- Observability: structured logs, metrics, tracing; sync health and integration error queues visible to admins.

## UX standards (see concept-note Section 7.7 and 7.8)
- Role-based home screens; frontline users see only what their job needs.
- Speed targets: add item in 1 tap or scan; open a tab in 1 card tap; repeat a round in 1 tap; take payment in 2 taps; find any item in 3 keystrokes.
- Environment-specific design: dark high-contrast bar theme with large targets and no reliance on sound; KDS readable from 2 m with bump-bar support; scanner/keyboard-first checkout lanes; one-handed handheld flows; sunlight-readable floor handhelds.
- Customer-facing screens meet WCAG 2.2 AA. Staff UIs support per-user language and left/right-handed layouts.
- Avoid decorative styling: no gradients, glassmorphism, drop-shadow cards, pill-shaped everything, emoji icons or marketing-page layouts in operational screens. Operational screens are dense, calm, legible and fast.
- Every device must show online/offline/sync/printer status and tell the user what to do when something fails.

## How to work
- You are working autonomously on reversible engineering work. Make routine judgement calls yourself; ask only when different readings would lead to materially different product behaviour (tax, money, legal, data ownership, scope). Stop before destructive operations (dropping data, force-pushing, deleting branches).
- The milestone prompt sets the scope. Don't quietly narrow, widen or swap it. If you spot a real problem with the specification, say so briefly and propose an option.
- Don't add features, refactors or dependencies the milestone didn't ask for. Note good ideas in `docs/PROGRESS.md` under "Later".
- Prefer targeted edits over rewriting whole files.
- Establish a way to check your own work as you build (tests, seed-data scenarios, scripted end-to-end flows) and run it regularly, not only at the end.
- Before declaring a milestone done: all checks green, `docs/PROGRESS.md` updated, ADRs written for new decisions, and a short report listing what was built (by process ID), what was deferred, and known risks.
````

---

## Part C. Prompt 0: architecture and plan (no code)

Use effort `xhigh`. Paste this as the first message of the first session.

```text
Read CLAUDE.md and docs/concept-note.md in full. This is the specification for a unified POS + ERP platform for supermarkets, bars, resto-bars and cafés.

Your task in this session is to design the system and plan the build. Do not write application code yet.

Finance, Inventory and Sales are the three core pillars (concept note Sections 19–23). Design them, their contracts (postJournal, recordStockMovement, completeSale) and the document engine they rely on (Section 24) first. Every other module must be designed to use them rather than writing money or stock data itself.

Produce these files:

1. docs/architecture.md
   - System context (cloud, store edge hub, terminals, handhelds, KDS, customer devices, payment providers, ERPs, existing POS systems).
   - Module boundaries for the modular monolith and the events each module publishes and consumes.
   - Offline-first design: local storage on devices, the sync protocol (commands/events, ordering, idempotency, conflict rules per data type), what works offline and with what limits, and how the edge hub relays on the LAN.
   - The three deployment modes (concept note Section 5) and how the integration hub and the system-of-record matrix (5.3) are implemented.
   - Security architecture: tenancy isolation, authentication (staff PIN/NFC/biometric on shared devices; SSO/MFA for back office), permissions with thresholds and remote approval, audit trail, PCI scope minimisation, secrets.
   - Deployment topology and environments.

2. docs/domain/model.md
   - Entities, aggregates and their relationships for: tenant/entity/location/device; people/roles/permissions; catalogue (items, variants, units, barcodes, modifiers, recipes/BoMs, price books, promotions, taxes); inventory (stock ledger, batches, locations, counts, transfers, production, returnable containers); sales (orders, tabs, tables, courses, tickets, payments, refunds, shifts, cash); purchasing (suppliers, POs, receipts, invoices, claims); customers (profiles, loyalty, credit accounts); finance (chart of accounts, journals, AR/AP, tax, periods, settlements); integration (connectors, mappings, sync messages); tasks/checklists.
   - State machines for: order/ticket, tab, table, shift, PO, receipt, supplier invoice, stock count, approval request, sync message.
   - Invariants that must always hold (money, stock, ledger balance, numbering, permissions).

3. docs/adr/0001... : one ADR for each key decision (stack, modular monolith, sync approach, money/quantity representation, tenancy isolation, event/outbox approach, device platforms, costing method support, how fiscal connectors plug in).

4. docs/PROGRESS.md
   - The milestone plan (M0, M1, C0, C1, M2, C2, C3, RW, M4–M14; use the list in the prompt guide as a starting point and adjust if the design suggests a better order). For each milestone: goal, process IDs covered, acceptance criteria that can be verified, and the main risks.
   - A traceability table mapping every process ID in the concept note (S1–S15, B1–B13, R1–R15, E1–E9, F1–F19, IN1–IN11, SA1–SA12, DOC1–DOC12, RW1–RW14) and every Section 7 subsection to the milestone that delivers it. Nothing may be left unmapped; mark items explicitly "Later" if they are out of the first release.

5. docs/open-questions.md
   - Decisions only the product owner can make (for example: jurisdictions and fiscal rules, first ERP connector, payment providers, which costing methods to support, data residency). For each, give your recommended default so work can proceed if I agree.

When you finish, reply with a short summary: the key design decisions, the three biggest risks, and the questions that block M0 or M1 (if any).
```

---

## Part D. Milestone prompts

**Standard opening** (paste at the top of every milestone prompt):

```text
Read CLAUDE.md, docs/PROGRESS.md, docs/architecture.md, docs/domain/model.md and the relevant ADRs. Then read the concept-note sections listed below.
First reply with a short plan (modules, data changes, key tests, order of work) and any blocking questions. After I confirm, implement, verify, update docs/PROGRESS.md, and finish with the milestone report described in CLAUDE.md.
```

Each prompt below says what to build (**Build**), what to leave out (**Out of scope**) and how you will judge it (**Done when**).

### M0. Repository foundation

```text
[Standard opening]

Milestone M0: repository foundation. Concept note: Sections 8 and 13 (architecture, implementation approach).

Build:
- The monorepo layout from docs/architecture.md, with empty but wired modules and shared packages (domain types, validation, UI kit, sync protocol).
- Local development: one command starts PostgreSQL, Redis, the API, the back-office web app and a terminal app in a simulator/browser; a seed command loads the demo tenant skeleton.
- CI: install, lint, type-check, unit tests, integration tests with a real PostgreSQL, build for all apps. Fails on any error.
- Tooling: formatter, linter rules, commit hooks, OpenAPI generation, database migrations, test utilities (factories, Testcontainers setup, property-based test library).
- Observability basics: structured logging, request IDs, health checks.

Out of scope: any business features.

Done when: a fresh clone reaches a running stack with one documented command; CI is green; a sample end-to-end test (API health + web app loads + terminal app loads) passes.
```

### M1. Platform core: tenants, identity, roles, permissions, approvals, audit

```text
[Standard opening]

Milestone M1: platform core. Concept note: Section 7 (all of it, especially 7.1–7.5), Section 10 (security), 5.1 D9.

Build:
- Tenants, legal entities, locations (with business-day cut-off time, time zone, currency, tax jurisdiction), areas/stations (lanes, bar stations, kitchen stations, café counter), devices (registration, pairing, revocation).
- People: one person with one login across locations; role assignments per location; employment details needed for scheduling and payroll export later.
- Authentication: back office with email + MFA (SSO-ready); shared-device staff login by PIN, NFC card/wristband token and optional biometric; fast user switching on shared terminals; automatic lock.
- Role templates for every role in concept-note tables 7.2, 7.3 and 7.4, seeded as data.
- Permission engine: named permissions; allow / deny / threshold (amount, percentage, count per shift, value per period); per-person overrides; location scoping. The default matrix in 7.5 is seed data and fully tested.
- Remote approval: when an action exceeds a threshold, create an approval request, push it to eligible approvers (mobile/web), approve or deny with reason, and resume or cancel the original action on the device. It must work when the terminal is offline but on the LAN (edge hub) and time out safely.
- Segregation-of-duties rules (for example, receiver cannot approve the supplier invoice for the same receipt) as configurable policies.
- Audit log: append-only, tamper-evident (hash chain), queryable by person, action, location, time; exception reports by person.
- Back-office screens: locations, devices, people, roles, permissions matrix editor, approval inbox, audit viewer.

Out of scope: sales, catalogue, inventory.

Done when: property/integration tests prove every row of the 7.5 matrix; approval flow works end to end in the demo (bartender requests a comp over budget → manager approves on phone → action completes and is audited); tenant isolation tests show no cross-tenant access through any API.
```

### C0. DOCUMENT ENGINE (foundation for the three cores)

Use effort `xhigh`.

```text
[Standard opening]

Core milestone C0: DOCUMENT ENGINE. Concept note: Section 24 (all), 19.1, 7.5 (approvals), Section 10 (fiscal and e-invoicing).

Every event in the Finance, Inventory and Sales cores starts from a business document. Build the shared document engine now. C1, C2, C3, M6 and the vertical modes add their specific documents and core effects on top of it.

Build:
- Document framework (DOC1–DOC5, DOC8, DOC11):
  - Document types defined as configuration.
  - The document kinds in 24.1, with their allowed core effects enforced: a commercial document can never post to the books or move stock, and a billing document can never move stock.
  - Number series per type, entity, location, terminal and fiscal year; gapless where the law requires; terminal-scoped series so offline terminals never collide.
  - Lifecycle states and transitions per type, with cancellation rules: issued billing documents can only be corrected by credit or debit notes.
  - Convert/copy to successor documents.
  - Per-line quantity tracking (ordered, delivered/received, invoiced, returned, paid), with tolerances and back-orders.
  - The document flow graph and view.
  - Approvals through the M1 permission engine; revisions with version history; attachments; audit.
- Payment terms and flows (DOC7): prepaid, postpaid (net days, end of month, instalments), cash on delivery, direct cash, early-payment discounts, and advances with automatic allocation to the final invoice.
- Matching framework (DOC6): two-, three- and four-way matching with tolerances and exception routing (used by C1 and M6).
- Templates (DOC9): template engine with per-country and per-language layouts and mandatory legal fields; PDF rendering; a sample template for every document in 24.2.
- Sending (DOC10): email with PDF; interfaces for WhatsApp, portals, EDI and structured e-invoices (UBL/Peppol or the national format) behind the fiscal connector interface.
- Statements and reminders framework (DOC12); the data will come from the ledgers built in C1.
- Every document type in the catalogue in 24.2 registered as configuration, with its kind, allowed effects, numbering, lifecycle and template. Their core effects are wired in the milestones that own them: C1 (billing, payment and adjustment documents), C2 (delivery and stock documents), C3 (sales documents), M6 (purchasing documents), M8/M9 (hospitality documents).

Out of scope: posting and stock logic (C1–C3). Use test doubles for core effects.

Done when:
- Property-based tests prove numbers are unique, and gapless where required, under concurrency and with offline terminals.
- A commercial document cannot trigger any core effect.
- The full buying chain (PR → RFQ → quotation → PO → delivery note/GRN → invoice → payment voucher → receipt) and selling chain (quotation → sales order → delivery note → invoice → receipt, plus a credit note) convert end to end on test doubles, with correct per-line quantities; the flow view shows the whole chain.
- Prepaid and postpaid flows allocate advances correctly (on test doubles).
```

### C1. FINANCE CORE (first core pillar)

Use effort `xhigh`.

```text
[Standard opening]

Core milestone C1: FINANCE CORE. Concept note: Sections 19 and 20 (all), 23.3–23.5; E1–E9 (summaries, now mapped to F-IDs); 7.5 (approvals); owner and accountant roles in 7.2–7.4.

Finance is the first of the three cores. Inventory (C2) and Sales (C3) will post into it, so design its contracts for them now.

Build:
- Setup (F1): chart-of-accounts templates per business type (supermarket, bar, resto-bar, café, mixed) with account classes and control-account flags; dimensions (entity, location, department, cost centre, event) on every journal line; fiscal years and periods; multi-entity; multi-currency with revaluation; opening-balance import with validation.
- Posting engine (F2): the postJournal contract from concept note 19.3. Journals are balanced by construction and idempotent by source-document key; the engine applies open-period, account-class and dimension rules, links each journal to its source document and writes an audit event. Posting rules are configurable (document type and attributes → accounts), with a rules editor and a preview that shows the journal a document would produce. Failed postings go to the issue inbox and are never dropped.
- Books of original entry (F3): cash book, bank book, petty cash book, daily sales book, sales day book, sales returns day book, purchases day book, purchases returns day book, expense book, other income register, general journal. The books are views over the single journal store, not separate copies. Each book uses the familiar layout (date, reference, particulars, debit, credit, running balance), with filters, drill-down to the source document, and print/PDF/Excel export.
- Ledgers and registers (F4, F11–F14): general ledger; debtors and creditors ledgers with enforced control accounts; fixed asset register with depreciation; tax register; tips and service charge ledger; deposits, gift cards and vouchers ledger; capital, loans and drawings.
- Income and expense tracking for owners (F5): simple money-in/money-out view by category in plain language; expense capture by phone photo with OCR behind an interface (a mock is fine now; a real implementation comes in M12); recurring bills with reminders; budget alerts. Every number drills down to the accountant view and matches it to the cent.
- Petty cash (F6): imprest floats, vouchers with photos, approvals, top-ups.
- Bank and cash (F7): bank accounts, statement import (CSV and the standard formats used in the target countries, e.g. OFX/CAMT.053/MT940) and a bank-feed interface, deposits, transfers, safes and tills as cash accounts.
- Reconciliations (F8): bank (auto-match rules and suggestions), cash, clearing accounts, control accounts, tax, liabilities, inter-company. Leave hooks for the inventory reconciliation that C2 will add.
- Accounts receivable and payable (F9, F10): invoices, credit and debit notes, receipts, payments, payment runs, statements, ageing, reminders.
- Tax (F11): tax engine (rates, inclusive/exclusive, exemptions, rounding per jurisdiction), tax register, return-preparation report, and the fiscal/e-invoicing connector interface with a mock.
- Payroll and tips posting (F13): import payroll journals; tips and service charge liabilities.
- Books health and error correction (F15):
  - Every check in concept note 20.5 (omission, commission, principle, original entry, reversal, compensating, transposition, duplication, cut-off, unexplained differences, suspense) as a rule with a severity, a plain-language explanation and a suggested fix.
  - The issue inbox.
  - The correction wizard (reverse and re-enter, reclassify, correct amount, move period), which generates correcting journals with reason, link and attachment, with approvals above thresholds.
  - Handling for closed periods, and the "what changed" report.
- Period and year-end close (F16): guided checklist; blocking conditions (unreconciled items, suspense not zero, open high-severity issues); lock; controlled reopening with two approvals; year-end roll-forward to retained earnings.
- Statements and reports (F17): trial balance, P&L by any dimension, balance sheet, cash-flow statement (indirect), statement of changes in equity, aged debtors and creditors, tax report, daily flash P&L, and a one-click accountant pack export.
- Budgets and cash forecast (F18).
- Roles and collaboration (F19): bookkeeper, accountant, finance manager, owner, and external accountant/auditor (read-only) role templates added to the M1 permission seed; comments and document requests on entries.
- Documents (concept note Section 24, on the C0 engine): wire the billing, payment and adjustment documents to the posting engine:
  - sales and purchase tax/commercial invoices; advance payment invoices with automatic allocation;
  - credit notes; debit notes in both meanings, each labelled;
  - payment vouchers with approvals and remittance advice; receipts;
  - journal vouchers (from the correction wizard); petty cash vouchers; expense claims; bank deposit slips;
  - statements of account and payment reminders.
  Proforma invoices must never post. The laptop example in 24.6 must reproduce exactly once M6 supplies the PO and GRN (use test doubles until then).
- Seed data: a demo company for each business type with three months of finance activity, including deliberately planted examples of every error type in 20.5, so health checks and the correction wizard can be demonstrated.

Out of scope: stock movements (C2), sales documents (C3), external ERP connectors (M10). Build test doubles that call postJournal the way C2 and C3 will.

Done when:
- Property-based tests prove: every journal balances; the trial balance always balances; sub-ledgers equal their control accounts; closed periods reject postings; reposting the same source document creates no duplicate.
- Every planted error in the seed data is detected by a health check, explained in plain language, and corrected through the wizard with a full audit trail, and the "what changed" report lists the correction.
- A month-end close runs end to end on seed data, including bank reconciliation. The statements tie out: the balance sheet balances, and P&L profit equals the change in retained earnings.
- Every book in concept note 20.2 opens in the familiar layout, and the owner's simple view agrees with the accountant view to the cent.
- The milestone report reminds me that a qualified accountant should review the chart-of-accounts templates and posting rules before the pilot.
```

### M2. Catalogue and item master

```text
[Standard opening]

Milestone M2: catalogue. Concept note: S1, S4 (price book part only), B4, R6, R9 (recipe structure), R15 (drink builder structure), Section 5.3 (item master ownership).

Build:
- Items with departments/categories, brands, tax codes, attributes (weighed, age-restricted, shelf life, storage type, allergens, returnable container/deposit), images.
- Multiple units of measure and pack levels per item with conversions; multiple barcodes per item and per pack; GTIN validation; variable-weight / price-embedded barcode rule definitions (configurable prefixes and formats).
- Variants and modifier groups (required/optional, min/max, nested, priced), for drinks (double, mixer, ice), coffee (size, shots, milk, temperature, syrups) and food (cooking level, sides, allergy notes).
- Recipes and sub-recipes (BoMs) with ingredient quantities, yield and trim loss, pour sizes; items can be sold, made, or both.
- Price books: base prices, store price zones, channel prices (in-store, QR, delivery, online), scheduled price changes, time-based prices (happy hour) as data.
- New-item approval workflow and duplicate detection; import/export (CSV) with validation report.
- Master-data ownership setting per tenant (our platform vs ERP) that later milestones respect.
- Back-office screens for all of the above, designed for bulk editing.

Out of scope: promotions engine (M7), stock quantities (M3).

Done when: seed data contains a realistic supermarket catalogue (including weighed produce and deli items), a bar menu with recipes and pour sizes, a restaurant menu with sub-recipes and allergens, and a café menu with a full drink builder; unit conversion and price-resolution logic have property-based tests.
```

### M3. Inventory engine: replaced by C2

The old M3 is superseded by core milestone **C2** below. If your build already completed M3, run C2 as a refactor of that work (Part H).

### C2. INVENTORY CORE (second core pillar, replaces M3)

Use effort `xhigh`. If your build already completed M3, use this prompt as a refactor of that work (see Part H).

```text
[Standard opening]

Core milestone C2: INVENTORY CORE. This replaces the old M3. Concept note: Section 21 (all), 19.3, 23.2–23.4; S11, S12 (tracking), B1, B5 (engine), B12 (containers), R10, R11; Section 5.1 D3.

Inventory is perpetual: every movement records quantity and value, and posts to the books through the Finance core (C1).

Build:
- The recordStockMovement contract from 19.3. Each movement records quantity and value and calls postJournal using the mapping in concept note 21.1, configurable per tenant through C1 posting rules.
- Stock ledger with every movement type in 21.1: receipt, sale depletion by recipe, customer return, return to supplier, transfer out/in via in-transit, production (consume ingredients, create output), waste, comps/spills/staff consumption, count variance, revaluation, container issue/return.
- Stock locations per site (shop floor, back store, cold room, bar stations, cellar, kitchen stations, café); par levels; requisitions from store room to bar and kitchen (IN2, IN5).
- Valuation and costing (IN3): weighted average (default), FIFO (option per tenant), landed costs, recipe cost roll-up, and automatic revaluation entries when negative stock is later corrected.
- Receiving and GRNI (IN4): a receipt before its invoice creates the GRNI accrual. Provide the hooks M6 will use for invoice matching, price differences and debit notes.
- Production and yield variance (IN6); counts (full, cycle with ABC scheduling, blind, partial bottles by tenths or weight, recount rules, variance approval), with variance posted to the books (IN7).
- Classification of waste, comps, spills, staff consumption and shrink to separate expense accounts (IN8); batch, lot and expiry with FEFO (IN9).
- Theoretical vs actual usage and variance per item, category, location, station and period, in quantity and money.
- Inventory–finance reconciliation (IN10), nightly and on demand: stock valuation equals the GL inventory accounts; the GRNI listing equals the GRNI account; COGS in the stock ledger equals COGS in the GL. Differences become F15 issues with a suggested fix.
- Documents (Section 24, on the C0 engine): wire the delivery and stock documents to recordStockMovement: GRN (creates GRNI), outbound delivery note (stock out and COGS), return-to-vendor note, customer return note, transfer note, stock adjustment and count sheet, production order/batch sheet, waste note.
- Inventory insights (IN11): stock value by location and category, days of cover, stock turn, dead stock, reorder needs, variance hot spots (API plus simple screens on the relevant role home screens).

Out of scope: purchasing documents (M6), sales documents (C3).

Done when:
- Property-based tests prove that on-hand always equals the sum of movements, and that stock valuation equals the GL inventory accounts at any point in time.
- The GRNI listing always equals the GRNI account.
- Every movement type in 21.1 produces the expected journal.
- The worked supermarket example in concept note 23.2 reproduces exactly.
- A scripted bar week with seeded over-pouring shows correct variance in both quantity and money, and the variance appears in the P&L.
```

### C3. SALES CORE (third core pillar)

Use effort `xhigh`.

```text
[Standard opening]

Core milestone C3: SALES CORE. Concept note: Section 22 (all), 19.3, 23.1, 23.3, 23.4; S5, S8, S10, S13, B3, B8, B10, R7, R14.

Sales is the third core. It works with Inventory (C2) and Finance (C1) so every sale produces correct stock and correct books, and so owners get quick reports and simple tasks.

Build (server-side domain and APIs; the terminal apps come in M4):
- Sales document model (SA1): receipt, tab, table bill, credit invoice, online, delivery, QR and self-checkout orders.
- The completeSale contract from 19.3. It finalises the document, depletes stock through C2 and posts through C1 as in 22.1, all in one transaction or through an exactly-once outbox.
- Tenders and clearing accounts (SA2): cash, card, mobile money, customer account, gift card and deposit; settlement clears to the bank, with fees and chargebacks posted.
- Discounts and promotions accounting, including vendor-funded claims (SA3).
- Returns, refunds and credit notes (SA4): posted to the sales returns day book, with stock returned or written off.
- Credit sales to the debtors ledger (SA5).
- Deposits, gift cards and vouchers as liabilities (SA6).
- Tips and service charges (SA7).
- Daily sales close (SA8): Z-close per terminal and location, posted to the daily sales book, with cash over/short.
- Real-time margin per line, sale, category and shift (SA9).
- Quick reports (SA10): every report in 22.2, with the same plain-language names, each available as an API and as a screen on the home screens of the roles listed for it.
- Insight-to-task engine (SA11): rules that create tasks from findings in all three cores (examples in 22.3), assigned by role, with a due time, closing automatically when the underlying issue is resolved.
- Channel sales with commissions (SA12).
- Documents (Section 24, on the C0 engine):
  - The POS receipt as simplified tax invoice and proof of payment, and a full tax invoice on request that references the receipt without counting revenue twice.
  - Refund, deposit and gift card receipts.
  - For account customers: quotation, proforma, sales order (reserves stock), pick list, delivery note, tax invoice, credit and debit notes.
  - Prepaid selling with advance invoices and deposit receipts.
  - Z-report and cash-up sheet.
- The daily three-way reconciliation job (23.3) covering all six checks. Failures create F15 issues and SA11 tasks.

Out of scope: terminal apps (M4), real payment providers (M5), vertical-specific flows (M7–M9).

Done when:
- The worked example in concept note 23.1 (gin and tonic on a card tab with tip, settled the next day) reproduces exactly in sales, stock and journals.
- A seeded week of mixed trading (supermarket, bar, resto-bar and café) passes the three-way reconciliation every night.
- Deliberately broken cases (an unmapped tax code, a missing card settlement, an item sold without a recipe) are detected and appear as finance issues and tasks.
- Every quick report answers its question on seed data and agrees with the finance statements.
- The selling example in concept note 24.6 (50 cases to a restaurant on account, with a return) reproduces exactly in documents, stock and journals, and the document flow view shows the whole chain.
```

### RW. Rwanda compliance pack (EBM/VSDC, tax, payments, data residency)

Use effort `xhigh`. Run it after C3 and before M4, so the terminals print official EBM receipts from their first real use.

```text
[Standard opening]

Milestone RW: Rwanda compliance pack. Concept note: Section 25 (all), 24.5, 10 (regulatory), 20.8 (F11 tax), 22 (SA1, SA2, SA4), 8.2 (offline).

Rwanda is the first market. Deliver it as a country pack (configuration plus connectors) on top of the country-neutral cores, so later countries follow the same pattern.

Before writing code:
- Read the current RRA documents: the Technical Specification of CIS for VSDC, the VSDC specification, and the EBM user manual (links in the concept-note sources).
- If a newer version exists, or if a document can't be accessed, tell me which versions you used and what you couldn't verify.
- Build a checklist in docs/compliance/rwanda.md from concept note 25.1–25.9. Every rule has its source and a status: "confirmed from the specification", or "needs confirmation with RRA/tax adviser". Add the second kind to docs/open-questions.md.

Build:
- RW1: VSDC connector per the current specification: initialisation, code and reference data retrieval, item and branch registration, sales and refund transmission, purchase retrieval, import items, stock reporting where required, retries and idempotency. Include a VSDC simulator for tests and a certification test suite that mirrors RRA's integration checkpoints and runs in CI.
- RW2: completeSale cannot complete a sale or refund without a VSDC signature. Store the SDC ID, SDC receipt number, internal data, receipt signature and MRC (or equivalents in the current specification) on the sales document. Print the receipt with every mandatory field and the QR code in the specified format.
- RW3: tax types A, B (18%), C and D on items and services; tax totals per type calculated exactly as the VSDC does (rounding confirmed against the specification and simulator).
- RW4/RW5: RRA item classification, packaging and quantity unit codes on the catalogue; customer TIN on B2B invoices; purchase invoices from EBM flowing into supplier-invoice matching.
- RW6: mapping of our documents to EBM receipt types exactly as in 25.3, including:
  - NR refunds referencing the original receipt;
  - CS/CR copies on reprint;
  - TS/TR in training mode;
  - PS for proformas;
  - the TIN entered before sale completion;
  - the "Confirm" items implemented behind configuration switches.
- RW7: VSDC hosted on the store edge hub; connectivity monitoring with alerts at 12 and 18 hours; default blocking when no VSDC is reachable; daily reconciliation of platform receipts and tax per type with the VSDC, with differences raised as F15 issues.
- RW8–RW10: Rwanda chart-of-accounts template (IFRS for SMEs); VAT return report (monthly or quarterly); withholding-tax handling on payment vouchers and receipts; tax calendar reminders.
- RW11: mobile-money tenders: request-to-pay by phone number (MTN MoMo, Airtel Money), eKash unified merchant code/QR on the customer display, automatic confirmation, refunds where supported, settlement reconciliation. Use the providers' current official API documentation.
- RW12: Rwandan payroll rules (PAYE bands, RSSB pension and maternity, CBHI) as dated configuration for payroll calculation or journal validation.
- RW13: per-country data residency: Rwandan tenants' personal data stored in a Rwanda region/data centre (document the hosting option chosen in an ADR); processing register; consent and data-subject request tools; retention of at least 10 years for financial and EBM records.
- RW14: Kinyarwanda, English and French for staff UI, receipts, invoices and reports; RWF with no decimals; +250 phone numbers; TIN validation; CAT time zone; Rwandan seed data (Kigali supermarket, bar/lounge, resto-bar with café).
- VAT reward: one-tap capture of the customer's phone number on the customer display, printed on the EBM receipt.

Out of scope: going live with RRA (that needs the business's own VSDC approval and RRA certification). Prepare the certification application pack in docs/compliance/rwanda-certification/.

Done when:
- The certification test suite passes against the VSDC simulator (and against the RRA test environment if access is available).
- Every EBM receipt type in 25.3 is produced correctly.
- The Rwanda examples in 25.10 reproduce exactly (MoMo bar sale; B2B NS with TIN).
- Tax per type matches the VSDC to the franc on a seeded month.
- A 20-hour VSDC internet outage at the edge hub triggers the alerts without losing a sale.
- docs/compliance/rwanda.md lists every rule with its source and status.
```

### M4. POS terminal core (offline-first)

```text
[Standard opening]

Milestone M4: POS terminal apps on top of the Sales core (C3). The terminals create and complete sales documents only through the Sales core contracts; they never post to the books or deplete stock themselves. Concept note: S5, S10, S8 (basic refund), 22.1, Section 8 design principles (offline, conflict rules), Section 7.7 and 7.8 (usability), 7.5 (permissions at the till).

Build:
- Terminal apps (Expo for Android/iOS, Tauri for lane PCs) sharing one UI package; device pairing; staff login and fast switching from M1.
- Local database with catalogue, prices, permissions and open orders; full offline selling.
- Sync engine per docs/architecture.md: outbound command queue with idempotency keys, inbound change feed, ordering, retries, conflict rules, and a visible sync status. Includes the edge hub relay for LAN-only operation.
- Cart and order: scan/search/quick keys, modifiers, quantity, notes, line and order discounts (permission-checked), price override with remote approval, suspend/recall, void rules (before/after send).
- Tenders: cash (change calculation, rounding), card via a payment interface stub (real providers in M5), vouchers, customer account (placeholder), split tender.
- Receipts: print (ESC/POS via edge hub or direct), email/SMS/QR e-receipt; receipt numbering per fiscal ADR; reprint with audit.
- Shifts and cash: float issue, cash drops at threshold, paid-in/paid-out, no-sale with reason, blind close by denomination, X/Z reports, over/short by person.
- Customer-facing display.
- Environment themes: lane (scanner/keyboard-first), bar (dark, high-contrast, large targets), handheld (one-handed).
- Training mode that never affects sales, stock or reports.
- Documents: POS receipt, refund receipt, full tax invoice on request and Z-report printed or sent from the terminal, using the C0 templates and terminal-scoped number series (safe offline). In Rwanda, every receipt is the EBM receipt signed through the RW connector (NS/NR, with CS/CR on reprint and TS/TR in training mode); the till lets staff enter a buyer TIN or VAT-reward phone number before completing the sale.

Out of scope: tabs, tables, KDS, promotions, scales.

Done when: automated end-to-end tests cover selling, paying, refunding and closing a shift; an offline chaos test (network cut mid-sale, device restarts, two terminals offline for an hour, then reconnect) shows no lost or duplicated sales, correct stock and correct books (the three-way reconciliation passes after the test); speed targets from CLAUDE.md are measured by an automated UI test and reported.
```

### M5. Payments

```text
[Standard opening]

Milestone M5: payments. Concept note: Section 10 (payments, security), B3, B8, S10 (settlement part), E4, SA2, SA7, F8.

Build:
- Payment provider abstraction (ADR): authorise, capture, pre-authorise, incremental authorise, adjust tip, void, refund, store-and-forward offline with configured per-transaction and total limits, and settlement report import. Implement a full mock provider plus one real provider adapter chosen in docs/open-questions.md (or a sandbox-only adapter if none is chosen yet).
- Terminal integration patterns: semi-integrated P2PE terminals and SoftPOS (tap to phone) via the provider SDK. Card data must never reach our code; store tokens, masked PAN and references only.
- Tips: prompts, tip adjustment, auto-gratuity rules; data model for tip pooling (rules come in M8/M11).
- Mobile money and QR payment adapter interface with a mock implementation. For Rwanda, implement MTN MoMo and Airtel Money request-to-pay and the eKash unified merchant code, if not already done in RW.
- Settlement reconciliation: match provider settlements, fees and chargebacks to shifts and bank deposits; unmatched items queue.

Finance: post tenders, tips, settlements, fees and chargebacks through the Finance core clearing accounts (SA2, F8). Card-settlement reconciliation uses the C1 reconciliation framework.

Out of scope: vertical-specific payment flows (M7–M9).

Done when: contract tests pass for the provider interface; the mock provider simulates declines, timeouts, partial approvals, offline store-and-forward and late declines, and the POS handles each correctly; a documented PCI scope statement exists in docs/security/.
```

### M6. Purchasing, receiving and supplier invoices

```text
[Standard opening]

Milestone M6: purchasing. Concept note: S2, S3, B12 (purchasing part), E3, F10, IN4, 23.2, Section 5.3 (supplier and PO ownership in Mode 2).

Build:
- Suppliers with catalogues, pack sizes, MOQ, lead times, price history, delivery schedules (including DSD suppliers).
- Purchase orders with approval limits from the permission engine; send by email/PDF, with WhatsApp/EDI/portal adapters as interfaces plus at least email implemented.
- Suggested orders (rule-based in this milestone: par, min/max, sales velocity, lead time, MOQ, pack rounding), with an explanation per line. AI forecasting improves this in M12.
- Receiving on handheld: scan against PO, blind receive option, over/short/damaged/wrong item with photos, batch and expiry capture, supplier claim/debit note generation.
- Cost-change handling: new cost → margin impact → suggested retail price → approval → price change and label/ESL queue.
- Supplier invoices: capture by upload with OCR extraction (behind an interface; implementation may use the AI module later), three-way match with tolerances, exception routing for approval.
- Supplier scorecards.
- Documents and flows (concept note 24.3, on the C0 engine):
  - Purchase requisitions, created manually or from suggested orders.
  - RFQs to several suppliers, with a quotation comparison.
  - Supplier proformas and prepayments through advance invoices.
  - POs with revisions.
  - The supplier's delivery note captured at receipt, then the GRN and the supplier invoice.
  - A buyer's debit note for claims; payment vouchers; remittance advice.
  - The standard, prepaid, direct-cash and DSD flows, with two-, three- and four-way matching.
  - Fixed-asset purchases post to the fixed asset register (F12), not inventory.

Finance and inventory: receipts use the C2 receiving contract (GRNI accrual, IN4); matched supplier invoices post to the purchases day book and creditors ledger through the C1 AP functions (F10), clearing GRNI; debit notes post to the purchases returns day book; approved invoices flow into C1 payment runs.

Out of scope: ERP connectors (M10).

Done when: end-to-end tests cover order → receive with discrepancy → claim → invoice match → approval; stock, cost, GRNI and the creditors ledger update correctly, and the worked examples in concept note 23.2 (supermarket delivery) and 24.6 (laptops as a fixed asset, postpaid and prepaid) reproduce through the real purchasing screens.
```

### M7. Supermarket mode

```text
[Standard opening]

Milestone M7: supermarket mode. Concept note: S4–S9, S12–S15, plus the supermarket roles in 7.2 and environments in 7.7.

Build:
- Scales: checkout scanner-scale and label-scale integration through the edge hub (driver interface + simulator), tare library, price-embedded/weight-embedded barcode parsing, price and PLU push to label scales, visual PLU picker. Produce recognition is an interface with a stub (camera model later).
- Promotions engine: percentage/amount off, multi-buy, mix-and-match groups, buy X get Y, basket thresholds, member prices, coupons, time windows, stacking and priority rules, budget caps, vendor-funded promotions with claim report. Deterministic, explainable result on the receipt; property-based tests.
- Shelf labels: label templates (including unit price), print batches for changed prices, ESL adapter interface with a mock.
- In-store production (bakery/deli): production batches, suggested daily plan, end-of-day waste, labels with ingredients/allergens/best-before.
- Returns with receipt lookup, reason codes, fraud rules and disposition (restock, write-off, return to vendor).
- Age-restricted items: mandatory checks, ID scanner interface, restricted hours per licence, log; benefit/voucher tender eligibility per item.
- Expiry markdowns: rules, reduced-price labels with barcode, daily expiring-items task list.
- Customer credit accounts with limits, statements and payment on account; B2B price tiers; loyalty points and tiers.
- Self-checkout app with attendant console: guided flow, weight/security checks (interface), age approvals, help requests.
- Click & collect: order intake API, picking app with route by aisle and substitutions, weighed-item final price.
- Head-office functions: store assortment, price zones, inter-store transfers.
- Documents: B2B account-customer flow (quotation → sales order → pick list → delivery note → tax invoice → statement of account → receipt), customer return notes, full tax invoice on request at the till.
- Role home screens for cashier, front-end supervisor, department manager, receiving clerk, stock clerk, picker, self-checkout attendant.

Out of scope: real hardware drivers beyond the simulator and one reference device per type (document which).

Done when: scripted demo "Saturday rush" passes in automated tests: mixed basket with weighed produce, a mix-and-match promotion, a coupon, an age-restricted item, split tender, a return, a self-checkout alert, and a supervisor blind close.
```

### M8. Bar mode

```text
[Standard opening]

Milestone M8: bar mode. Concept note: B1–B13, plus bar roles in 7.3, bar environment in 7.7.

Build:
- Speed screen with auto-arranged top sellers by time of night; one-tap repeat round; NFC fast login per bartender on shared terminals.
- Tabs: open by card with pre-authorisation, incremental authorisation, tab limits and alerts, name/seat tabs, transfer/merge/split, guest view and pay by QR, auto-close rules, walk-out report.
- Recipes and pours on bar display screen; standard pour sizes.
- Liquor counts with bottle scale integration (Bluetooth interface + simulator) and tenths; variance by product, station, shift and bartender with thresholds (5% investigate, 10% audit); investigation task log. Optional smart spout / keg meter interface.
- Comps, spills, staff drinks with reason codes and budgets (permission thresholds from M1).
- Happy hour and event pricing with clear rules for open tabs.
- Tips: pooling rules by role, hours or points; tip-out to barbacks; cash tip declaration; export.
- Door: cover charges and tickets on handheld, guest-list scan, live capacity counter, ID scanner interface.
- Bottle service and minimum spend with deposits.
- Licensing: last-call lockout, licensed hours, incident log.
- Keg and container tracking with deposits; distributor ordering using M6.
- Closing: forced tab handling, bartender cash-out, closing count of high-value lines.
- Documents: bar order tickets (BOT), tabs as guest checks, deposit receipts for VIP tables and bottle service, container return notes for kegs and crates, distributor purchasing through the M6 flows.
- Role home screens for bartender, head bartender, barback, door staff, VIP host, bar manager, cellar person.

Done when: scripted demo "Friday night" passes: 200 drinks across three stations, tabs opened by card and closed, one walk-out prevented by pre-auth, happy-hour switch, comps within and over budget (remote approval), end-of-night cash-outs, next-day count with a seeded over-pour showing correct variance.
```

### M9. Resto-bar and café mode

```text
[Standard opening]

Milestone M9: resto-bar and café mode. Concept note: R1–R15, plus roles in 7.4, kitchen/floor/coffee environments in 7.7.

Build:
- Reservations with deposits and no-show fees, online booking widget API, waitlist with SMS/WhatsApp interface, guest notes.
- Floor plans per area; table states and timers; combine/split tables; server sections.
- Handheld ordering by seat and course; QR ordering linked to the open bill; suggestive selling by margin.
- Routing to kitchen/bar/café stations; KDS and BDS apps with coursing (hold/fire), timers and colour thresholds, expo view, all-day counts, recall/re-fire with reason, bump-bar keyboard support, readable at 2 m.
- Allergens on menus, highlighted on tickets with mandatory kitchen acknowledgement.
- 86/sold-out from kitchen or automatically from stock, synced to all channels.
- Bills: split by seat/item/equal/custom, merge, service charge rules, pay at table, hotel room charge via PMS interface.
- Delivery aggregators: order-injection interface with a mock aggregator and one real adapter if chosen; menu/availability sync; commission reporting.
- Recipe costing, theoretical vs actual food cost, menu-engineering matrix data.
- Prep lists from forecast/reservations; sub-recipe production with use-by labels; waste logging.
- Café: barista display queue (counter, table, QR, order-ahead) by promised time with names; cup-label printing; coffee recipes depleting beans/milk/cups; milk waste; order-ahead with load-based pickup times; digital stamp card via loyalty.
- Documents: kitchen and bar order tickets (KOT/BOT), guest checks, deposit receipts for reservations, the prepaid event and catering flow (quotation → advance payment invoice → deposit receipt → final tax invoice), full tax invoice on request for business meals.
- Role home screens for host, server, runner, floor manager, head chef, line cook, expeditor, head barista, barista, delivery coordinator.

Done when: scripted demo "Dinner service" passes: reservations and walk-ins, 40 covers, coursing with a delayed main, an allergy order, a sold-out item synced to QR and delivery, split bills, delivery orders, and a café rush with order-ahead; ticket times and food cost calculate correctly.
```

### M10. ERP integration hub (finance core moved to C1)

```text
[Standard opening]

Milestone M10: ERP integration hub. The finance core was built in C1; this milestone connects it, and the inventory and sales cores, to external systems. Concept note: Section 5 (5.2 deployment modes, 5.3 system of record, 5.4 integration principles), Section 11 (integrations), E1–E9 / F-IDs for what is exchanged.

Build:
- Connector framework: authentication, field mapping UI (items, taxes, tenders, GL accounts, cost centres, locations), sync schedules, idempotent messages, retries, dead-letter/error queue with fix-and-resend, reconciliation reports, full logging.
- Mode 2 (our POS on the customer's ERP): the customer's ERP can be the financial system of record. Our Finance core still produces the journals and books (so health checks and reconciliations keep working). It exports them to the ERP per transaction, per shift or as daily summaries, and reconciles that the ERP received exactly what was sent.
- System-of-record configuration per data object as in 5.3, enforced by the sync logic.
- Connectors in priority order from docs/open-questions.md. Default order: Odoo, ERPNext, QuickBooks Online, Xero; then Dynamics 365 Business Central and SAP Business One. Each connector ships with contract tests against a sandbox or recorded fixtures.
- Generic API/webhook and CSV/SFTP connectors.
- Export of the accountant pack (F17) in formats the customer's accountant can import.

Done when: the first ERP connector passes a round trip (items in; sales, journals, supplier invoices and POs out) with duplicate-delivery and failure-recovery tests; after a seeded month, the trial balance in the ERP equals the trial balance in our Finance core for the exported accounts.
```

### M11. Management layer and staff experience

```text
[Standard opening]

Milestone M11: management layer. Concept note: Section 7.2–7.4 (home screens), 7.6 (routines), 7.9 (staff experience), R12, R14.

Build:
- Role-based home screens and dashboards for every management role in 7.2–7.4 (owner, store manager, duty manager, front-end supervisor, department manager, buyer, bar manager, GM, floor manager, head chef, head barista, cost controller, accountant), showing exactly the data listed in the "Home screen on login" column.
- Owner/manager mobile app: live summary, alerts, one-tap approvals, data-light mode.
- Routines from 7.6 as configurable checklists (opening, during shift, closing, weekly, monthly) with the needed data embedded and sign-off.
- Tasks with assignment, due time and photo proof; shift handover notes; staff announcements and pre-shift briefing.
- Scheduling: rota built against forecast sales/covers with labour-cost targets, availability, swaps, break/overtime rules; time clock on terminals; payroll export (hours, tips, service charge).
- Staff app: schedule, swaps, earnings and tips, announcements, training modules, and the person's own performance data.
- Automatic daily sales report at close to owner/managers.

Done when: each management role can complete its 7.6 daily routine in the demo without leaving its home screen; scheduling and time clock produce a correct payroll export for the seeded week.
```

### M12. Reporting, analytics and AI features

```text
[Standard opening]

Milestone M12: reporting and AI. Concept note: Section 12 (KPIs), Section 8.4 (AI features), S2 (forecast-based ordering), 5.1 D8.

Build:
- Reporting layer: every KPI listed in concept-note Section 12, by location/department/station/person/period, with drill-down to transactions; scheduled report delivery; exports; a read-optimised store (materialised views or a warehouse, per ADR).
- Forecasting service: demand by item/location/day (and by hour for staffing), with seasonality, holidays, promotions and events; backtesting with accuracy metrics; feeds suggested orders (M6), production plans (M7), prep lists (M9) and rotas (M11). Explanations shown to users.
- Anomaly detection: voids, refunds, discounts, comps, no-sales, cash over/short and variance patterns per person/terminal; alert rules and weekly exceptions summary; false-positive feedback loop.
- Markdown optimisation suggestions for expiring items.
- Natural-language questions over reports ("What were last Friday's top cocktails?") using the Claude API through the official Anthropic TypeScript SDK. The model ID comes from configuration. Consult the current Anthropic documentation for request shapes instead of relying on memory. The assistant may only call read-only, permission-checked reporting tools; it must never see card data or data outside the user's permissions; log every question and answer.
- Supplier invoice OCR/extraction implementation behind the M6 interface.

Done when: forecasts beat a naive baseline (same day last week) on seeded data, measured and reported; anomaly detection finds seeded fraud patterns with an acceptable false-positive rate; NL questions respect permissions in tests (a bartender cannot see margins).
```

### M13. Intelligence layer over existing POS (Mode 3)

```text
[Standard opening]

Milestone M13: Mode 3. Concept note: Section 5.2 Mode 3, 5.3, 5.4, Section 11 (existing POS connectors), Section 14 (commercial model: intelligence layer).

Build:
- Import connectors for existing POS data: official APIs where available (start with the two chosen in docs/open-questions.md, e.g. Square and Toast) and a generic CSV/database-export importer with mapping UI.
- Mapping of imported items to our recipes and catalogue, so variance, food/pour cost and forecasting work on imported sales.
- Mode 3 product surface: dashboards, variance, suggested orders pushed as drafts to the ERP, anomaly alerts, consolidated multi-system reporting.
- Clear data-freshness indicators and gaps reporting when the source system limits access.

Done when: a demo tenant fed only by imported data from a mock external POS and a mock ERP shows correct variance, forecasts and suggested orders, and pushes draft POs to the ERP mock.
```

### M14. Hardening and release

```text
[Standard opening]

Milestone M14: hardening. Concept note: Sections 10, 16 and 17 (security, risks, success measures), Section 7.8 (accessibility), 7.9 (usability measurement).

Do:
- Security review of the whole system (authentication, authorisation, tenant isolation, injection, secrets, device loss, audit tamper-evidence, dependency vulnerabilities). Fix findings. Write docs/security/threat-model.md and the PCI DSS v4.0 scope statement.
- Performance and load tests: lane item-add latency, 50 terminals per site, 500 sites per region, peak-hour sync backlog, report queries. Set budgets and fix regressions.
- Offline chaos suite: network partitions, clock skew, device crash mid-transaction, duplicate delivery, edge-hub failure; prove invariants in Part F hold.
- Accessibility audit of customer-facing screens against WCAG 2.2 AA; fix issues.
- Usability test kit for the pilot: task scripts per role (from 7.2–7.4), timing capture, SUS questionnaire, and a report template.
- Operations: backups and restore drill, disaster recovery plan, monitoring dashboards and alerts, runbooks, release and rollback process, device fleet update process.
- Documentation: admin guide, role-based user guides, API reference, connector guides.

Done when: no open high-severity findings; all budgets met or documented with a plan; chaos suite green; restore drill documented; pilot usability kit ready.
```

---

## Part E. Reusable prompts

### E.1 Verify (end of every milestone)

```text
Use a sub-agent with a fresh context to verify milestone [Mx] independently. Give it only: CLAUDE.md, docs/concept-note.md, docs/PROGRESS.md, the milestone prompt, and access to the code and tests.

The verifier must:
1. For each process ID and acceptance criterion in scope, find the implementing code and the test that proves it. List anything missing, partial or untested.
2. Check the domain invariants in the prompt guide Part F that this milestone touches, and try to break them with new edge-case tests (don't commit tests that duplicate existing ones).
3. Check the role and usability requirements (Section 7) for any screens built: right home screen, permissions enforced, speed targets, environment theme, offline status.
4. Report findings ranked by severity with file:line references.

Then fix every confirmed finding of medium severity or higher, re-run all checks, and update docs/PROGRESS.md with the outcome.
```

### E.2 Code review before merging

```text
Review the diff of this branch against main as a senior engineer on a payments and accounting system. Look for: money or quantity errors, non-idempotent commands, lost offline data, permission bypasses, tenant leaks, missing audit events, mutable ledger records, unhandled provider failures, race conditions in sync, N+1 queries, and UX that breaks the speed targets. Report only real problems with a concrete failure scenario, most severe first. Then fix them.
```

### E.3 Resume a milestone in a new session

```text
Read CLAUDE.md, docs/PROGRESS.md and the git log for this branch. Summarise in five lines where milestone [Mx] stands and what remains, then continue the work. Keep the original milestone scope.
```

### E.4 Fix a bug

```text
Bug: [what happened, steps, expected vs actual, device/location/role, logs if any].
Reproduce it first with a failing test. Find the root cause (not just the symptom), fix it with the smallest correct change, and check whether the same cause affects other modules. Report the cause, the fix and the test.
```

### E.5 Change request

```text
Change request: [description]. Before changing code, update docs/concept-note.md (or add docs/changes/NNNN.md) with the new requirement and its process ID, update the traceability table in docs/PROGRESS.md, then give me a short impact plan (modules, data migration, risks). Implement after I confirm.
```

### E.6 Build or refine a screen for a role

```text
Build/refine the [screen] for the [role] (concept note 7.2/7.3/7.4) used in the [environment] (7.7).
Requirements: show only what this role needs; meet the CLAUDE.md speed targets; use the environment theme; support offline status; permission-check every action.
Avoid: gradients, glassmorphism, card grids with drop shadows, pill-shaped buttons everywhere, emoji icons, marketing-style hero headers, tiny touch targets, low-contrast grey text, and animations that delay input.
After building, run the screen in the simulator, take screenshots at the target device sizes, and check them against these requirements before reporting.
```

### E.7 Add a connector

```text
Add a [ERP/POS/payment/aggregator] connector for [system]. Follow the connector framework and CLAUDE.md. Read that system's current official API documentation before writing code; don't rely on memory. Implement authentication, mappings, the data flows in concept-note 5.3 for the deployment mode(s) [1/2/3], idempotency, retries and the error queue. Add contract tests with recorded fixtures and document setup in docs/connectors/[system].md.
```

### E.8 Add a country (tax and fiscal)

```text
Add support for [country], following the pattern of the Rwanda country pack (concept note Section 25, milestone RW). Research the current VAT/GST/sales-tax rules, rounding rules, receipt and invoice requirements, and fiscal device or e-invoicing obligations from official sources, and list the sources in docs/compliance/[country].md. Implement tax configuration and a fiscal connector plug-in behind the existing interface. Ask me before any legal interpretation that is ambiguous.
```

### E.9 Books accuracy drill (after C3, then monthly)

```text
Using a copy of the demo tenant, plant one example of each error type in concept note 20.5, plus one broken case for each check in the three-way reconciliation (23.3). Plant them through the normal APIs and screens, not by editing the database. Then run the books health checks and the nightly reconciliation.
Report, for each planted problem: whether it was detected, how it was explained to the user, the suggested fix, and whether applying the fix corrects the books completely (trial balance, sub-ledgers, stock valuation and reconciliation all clean afterwards). Fix any gaps you find, then run the drill again.
```

### E.10 Document flow walk-through (after C3 and after M6)

```text
On the demo tenant, walk through every flow in concept note 24.3 and 24.4 using the real screens and APIs, as the roles listed in 24.8:
- buying: standard postpaid, prepaid, direct cash and DSD;
- selling: walk-in sale, bar tab, table bill, account customer, prepaid event;
- returns: customer returns, and supplier returns with debit and credit notes.
For each flow, check that:
- each document has the right number series, status, template and legal fields;
- the flow view shows the full chain;
- per-line quantities (ordered, received, invoiced, paid) are right;
- stock and journals match the concept note's rules and examples;
- the three-way reconciliation passes afterwards.
Report gaps with file:line references and fix them.
```

---

## Part F. Quality gates and invariants

**Every milestone must pass these gates before merge:**

1. Lint, type-check, unit, integration and end-to-end tests green in CI.
2. Verify prompt (E.1) run; all medium+ findings fixed.
3. `docs/PROGRESS.md` updated, including traceability by process ID; ADRs for new decisions.
4. Demo seed data updated so the new features can be shown.
5. No new dependency without a stated reason; no secrets in the repository.
6. The three-way reconciliation (concept note 23.3) passes on the demo data, and the architecture test confirms no module outside the cores writes journal, stock or balance data.

**Invariants that must be covered by property-based or scenario tests:**

| Area | Invariant |
|---|---|
| Money | All amounts are integers in minor units; totals equal the sum of lines, tax and rounding exactly; refunds never exceed the original tender. |
| Tax | Tax per jurisdiction matches configured rules for inclusive and exclusive prices across all rounding modes. |
| Stock | On-hand equals the sum of ledger movements; every sale depletes the recipe quantities exactly once; transfers balance out and in. |
| Costing | Inventory value equals the sum of cost layers; COGS plus closing value equals opening value plus purchases (per period). |
| Ledger | Every journal balances; closed periods cannot change; every posting traces to a source document. |
| Sync | Offline commands apply exactly once; no sale is lost or duplicated after any sequence of disconnects, restarts and retries; ordering conflicts resolve deterministically. |
| Numbering | Receipt/invoice numbers are unique, and gapless where required. |
| Permissions | No action above a threshold completes without a recorded approval; no cross-tenant or cross-location data leakage. |
| Audit | Every sensitive action has an audit event; the audit hash chain verifies. |
| Core boundaries | Only the Finance, Inventory and Sales cores write journals, stock movements and balances; every other module goes through `postJournal`, `recordStockMovement` or `completeSale`. |
| Three-way reconciliation | Sales ↔ Finance, Sales ↔ Inventory, Inventory ↔ Finance, Tenders ↔ Finance, Settlements ↔ Bank and Sub-ledgers ↔ GL all agree (concept note 23.3), per day and location. |
| Books | Every book of original entry and ledger is a view over the one journal store; sub-ledgers equal control accounts; suspense is zero at every closed period; corrections are journals, never edits. |
| Rwanda EBM | Every completed sale and refund in a Rwandan location has an RRA-signed NS or NR receipt; copies, training and proformas are never official and never post; platform totals and tax per RRA tax type equal the VSDC's figures daily. |
| Documents | Issued documents are never edited or deleted; numbers are unique, and gapless where required; per line, invoiced ≤ delivered/received ≤ ordered (within tolerance); every stock movement and journal traces to a document of an allowed kind; commercial documents never post or move stock; every advance is allocated or shown as open. |
| One event, three views | No completed sale exists without its stock movements and journal, and no journal from a sale exists without its sale. |
| Payments | No card data stored; every authorisation is captured, voided or expired; tips adjust only within provider rules. |

---

## Part G. Tips for working with Claude Opus 5.5

- **Give goals and limits, not step-by-step instructions.** The prompts above say what to build, what not to build and how success is measured, and leave the engineering steps to the model. Over-prescribed prompts tend to lower the quality of work from recent Claude models.
- **Set effort deliberately.** Opus 5.5 defaults to a lower effort level than earlier Opus models. Choose `xhigh` for design and correctness-critical milestones and `high` for the rest.
- **Start with the hardest part.** Prompt 0 and M1, M3, M4 and M10 carry the most risk (permissions, stock, offline sync, finance). Give them the most attention and review time.
- **Let it ask, then let it work.** Each prompt asks for a plan and blocking questions first. After you confirm, let it work without interruption; it will stop only for product decisions or destructive actions.
- **Use fresh-context verification.** A separate verifier sub-agent (E.1) catches more than self-review. Run it every milestone.
- **Hold the scope.** If it proposes extras, have them added to "Later" in `docs/PROGRESS.md` instead of building them now.
- **For UI, name what to avoid.** A general request like "make it look professional" tends to produce generic styling. Name the specific patterns to avoid (as in E.6) and review screenshots.
- **Tell it to check live documentation** for any external API (payments, ERPs, delivery platforms, fiscal systems, the Claude API). Those APIs change faster than any model's training data.
- **Automate the checks.** Configure Claude Code hooks or CI so lint, type-check and tests run automatically after edits. The model then gets immediate feedback.
- **Build the cores before the screens people see.** Finance and inventory are less visible than the till, but getting them right first is what makes every later report and screen trustworthy. Resist moving the vertical modes ahead of C1–C3.
- **Review what matters most yourself:** money, tax, permissions, and anything that touches real card terminals or real customer data. Get a qualified accountant to review posting rules and tax, and a security professional to review M14 findings before going live.

---

## Part H. Upgrade path for a build already in progress

Use this part if you have already started building from the earlier version of the concept note (v0.3) and this guide. It re-plans the existing code around the three core pillars, **Finance, then Inventory, then Sales**, without discarding working code.

### H.1 What changed

| Area | Before | Now |
|---|---|---|
| Specification | Concept note v0.3 | Concept note **v0.5**: new Sections 19–23 (core pillars) and Section 24 (business documents), with process IDs F1–F19, IN1–IN11, SA1–SA12 and DOC1–DOC12. Sections 1–18 and IDs S, B, R and E are unchanged. |
| Market | Country-neutral, facts "TBD" | **Rwanda first**: concept note Section 25, prefilled Project facts, milestone **RW** (EBM/VSDC, tax, mobile money, payroll, data residency, languages) after C3 |
| Documents | Created ad hoc inside each milestone | **C0 document engine** before C1 (numbering, lifecycle, conversion, flow view, matching, payment terms, templates); each core and M6/M8/M9 wire their own documents (Section 24) |
| Finance | Built late (M10), together with ERP connectors | Built **first** as core C1, with accounting books, ledgers, reconciliations, health checks, error correction, close and an owner view |
| Inventory | M3 stock engine, with cost kept "for later finance posting" | Core **C2** (replaces M3): perpetual inventory that posts every movement to the books, with GRNI and inventory–finance reconciliation |
| Sales | Sales model created inside the POS terminal milestone (M4) | Core **C3**: server-side sales core that posts to stock and books, with Z-close, quick reports and insight-to-task. M4 becomes the terminal apps on top of it |
| Payments (M5) | Settlement matching only | Also posts tenders, tips, settlements, fees and chargebacks through clearing accounts |
| Purchasing (M6) | Invoices matched; AP "later" | Invoices post to the purchases day book and creditors ledger; GRNI clears on matching |
| M10 | Finance core + ERP hub | **ERP integration hub only**; it maps the finance core to external ERPs |
| Rules | `CLAUDE.md` without core rules | `CLAUDE.md` gains the "Core pillars" section (H.2) |

### H.2 Update the repository files

1. Replace `docs/concept-note.md` in your product repository with the new `POS_Concept_Note.md` (v0.6). Replace the "Project facts" block in your `CLAUDE.md` with the Rwanda version from Part B.
2. Add this section to your existing `CLAUDE.md`, directly after "What we are building":

````markdown
## Core pillars: Finance → Inventory → Sales (concept note Sections 19–23)
- Finance, Inventory and Sales are the three core modules, built in that order. Every other module (payments, purchasing, supermarket/bar/resto-bar modes, people, management, reporting, AI, integrations) is built on them.
- Only the cores write money and stock. Other modules use the core contracts: `postJournal` (Finance), `recordStockMovement` (Inventory) and `completeSale` (Sales). No other module writes journal lines, stock movements or balances directly. An architecture test enforces this.
- One event, three views: a sale, its stock movements and its journal are committed together (one transaction or an exactly-once outbox). None exists without the others.
- The books of original entry and ledgers (concept note 20.2) are views over one journal store, not separate copies.
- Every journal links to a source document. Posted entries are never edited or deleted; corrections go through the correction wizard (reversal, reclassification, amount correction) with reason, approval and audit.
- A posting that fails is never dropped: it is held in the finance issue inbox and shown as "unposted" until fixed.
- The three-way reconciliation (23.3) runs nightly and in the test suite. Any failure is a bug or a finance issue, never ignored.
- In Rwanda, no sale or refund is complete without an RRA-signed EBM receipt from the VSDC (concept note 25.2–25.4); tax per RRA tax type must equal the VSDC's figures to the franc.
- Every core event starts from a typed business document from the document engine (concept note Section 24). Only billing, payment and adjustment documents post to the books; only delivery and stock documents move stock; commercial documents (PR, RFQ, quotation, proforma, PO, SO) never do either. Issued documents are never edited or deleted.
- Process IDs: F1–F19 (finance), IN1–IN11 (inventory), SA1–SA12 (sales), DOC1–DOC12 (documents), RW1–RW14 (Rwanda). Reference them in module READMEs and tests.
````

3. Commit these two changes on a new branch (for example `upgrade/core-pillars`) before running U0.

### H.3 Prompt U0: assess and re-plan (no application code)

Use effort `xhigh`.

```text
The specification has been upgraded. docs/concept-note.md is now v0.6 (first market: Rwanda, Section 25): Finance, Inventory and Sales are the three core pillars (Sections 19–23), to be built in that order, with new process IDs F1–F19, IN1–IN11 and SA1–SA12. They rest on a document engine and complete buying and selling document flows (Section 24, DOC1–DOC12). CLAUDE.md has a new "Core pillars" section.

Your task in this session is to assess the existing code against the upgraded specification and re-plan the work. Do not change application code yet.

1. Read CLAUDE.md, docs/concept-note.md (especially Sections 19–23), docs/PROGRESS.md, docs/architecture.md, docs/domain/model.md and all ADRs. Then read the code.

2. Write docs/upgrade/core-pillars-assessment.md containing:
   - What exists today for finance, inventory and sales (modules, tables, APIs, tests), with file references.
   - A gap analysis against F1–F19, IN1–IN11, SA1–SA12, DOC1–DOC12, RW1–RW14 (Rwanda), the document catalogue and flows in Section 24, and the core contracts in 19.3. For each ID give a status (done / partial / missing / conflicts with the spec) and the evidence.
   - Boundary violations: every place where code outside the cores writes money, journal or stock data, or calculates totals that should come from the cores.
   - The data model changes needed and a migration approach for existing data, including backfilling journals and stock values for existing sales and movements, with a reconciliation proof that the backfill is complete and correct.
   - What to keep as is, what to refactor and what (if anything) to replace, with reasons. Prefer refactoring working code over rewriting it.

3. Update docs/architecture.md and docs/domain/model.md for the three cores and their contracts. Add ADRs for: core boundaries; the posting engine; books as views over one journal store; perpetual inventory valuation; the three-way reconciliation; the migration and backfill approach.

4. Update docs/PROGRESS.md: the new milestone order from the prompt guide (A.3, including C0), adapted to where this build actually is; the traceability table extended with the new IDs; and, for work finished under old milestones, what still needs to change.

5. Reply with a short summary: the current state, the plan for C0 → C1 → C2 → C3 → RW in this codebase, the main risks, and any product decisions I need to make.
```

### H.4 Existing-build addendum for C0, C1, C2 and C3

When you run the C0, C1, C2 and C3 prompts on an existing codebase, paste this paragraph directly after the standard opening:

```text
This codebase already contains work from earlier milestones; see docs/upgrade/core-pillars-assessment.md. Keep what already meets the specification, refactor what partly meets it, and migrate existing data safely: reversible migrations, backfills with a reconciliation proof, and no data loss. Where existing modules write money or stock directly, route them through the core contracts in this milestone and add an architecture test that stops it from happening again. Keep existing features working; run the full existing test suite before and after, and report any behaviour that changed on purpose.
```

### H.5 What to do depending on where your build is

| Where you are | What to run next |
|---|---|
| Any stage (Rwanda) | Run **RW** after C3 and before any terminal is used for real sales. If terminals already exist, RW adds EBM signing to them. Money handling must use whole Rwandan francs (0 decimals). If your code assumed 2 decimals for every currency, fix that in C1 with a migration. |
| Any stage | Run **C0** before C1. If your code already has documents (orders, invoices, receipts), C0 becomes a refactor that moves them onto the engine and migrates their numbering and history. |
| Prompt 0 and/or M0 done | U0 (short), then M1 → C0 → C1 → M2 → C2 → C3 → M4 → … |
| M1 and/or M2 done | U0, then C0 → C1 → C2 → C3 (M2 is kept; C2 builds on it) |
| Old M3 (inventory engine) done | U0, then C1, then C2 as a **refactor** of M3: add valuation posting, GRNI, movement-to-journal mapping, inventory–finance reconciliation, backfill journals for existing movements |
| Old M4 (POS terminal) done | U0, C1, C2, then C3: move sale completion from the terminal/server code into completeSale; terminals send sales documents; backfill sales, stock and journals for existing data; keep the offline sync guarantees |
| M5 and/or M6 done | As above, and in C1–C3 also rewire payments to clearing accounts and purchasing to AP/GRNI (use the addendum) |
| Old M10 (finance) done | U0 compares the existing finance module with F1–F19; C1 becomes gap-closing: books as views, health checks, correction wizard, close checklist, owner view, reconciliations. M10 continues as the ERP hub only |
| Vertical modes (M7–M9) done | As above; then run E.1 (Verify) for each mode with a focus on the three-way reconciliation, and fix any mode that bypasses the cores |

### H.6 After the upgrade

- Continue with the remaining milestones in the new order (A.3).
- Run **E.1 Verify** at the end of C0, C1, C2 and C3; then **E.9 Books accuracy drill** and **E.10 Document flow walk-through** after C3.
- From now on, every milestone's "Done when" implicitly includes: the three-way reconciliation passes on the demo data, and no module bypasses the core contracts.
