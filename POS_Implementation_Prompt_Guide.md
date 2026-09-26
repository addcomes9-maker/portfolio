# Prompt Guide: Building the Unified POS + ERP Platform with Claude Opus 5.5

This guide contains the prompts to give **Claude Opus 5.5** (in Claude Code or a similar agentic coding environment) to build the system described in `POS_Concept_Note.md`: a cloud-native, offline-first POS with ERP-grade back office for supermarkets, bars, resto-bars and cafés.

It has seven parts:

| Part | What it is | When you use it |
|---|---|---|
| **A** | How to run the build | Read once before starting |
| **B** | `CLAUDE.md`, the project rules file | Copy into the new repository before the first session |
| **C** | Prompt 0: architecture and plan | First session |
| **D** | Milestone prompts M0–M14 | One milestone per session, in order |
| **E** | Reusable prompts (verify, review, resume, fix, UI) | Any time |
| **F** | Quality gates and invariants | At the end of every milestone |
| **G** | Tips for working with Opus 5.5 | Keep at hand |

---

## Part A. How to run the build

### A.1 Setup

1. Create a **new, empty repository** for the product (for example `unified-pos`). Don't build it inside a portfolio or documents repository.
2. Copy `POS_Concept_Note.md` into it as **`docs/concept-note.md`**. This is the specification. Every prompt below refers to it by section number (S1–S15, B1–B13, R1–R15, E1–E9, Section 5 deployment modes, Section 7 roles and usability).
3. Copy Part B of this guide into the repository root as **`CLAUDE.md`**. Claude Code reads it automatically at the start of every session, so the rules persist across sessions.
4. Fill in the **"Project facts"** block at the top of `CLAUDE.md`: countries, currencies, languages, first customers, and the ERP and payment providers to support first. Leave anything unknown as "TBD". Prompt 0 will list the open questions.
5. Choose model **Claude Opus 5.5**. Use effort **`xhigh`** for Prompt 0, milestones M1, M3, M4, M10 and M14 (architecture-heavy, correctness-critical). Use **`high`** for the others. Lower effort only for small fixes.

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
M2   Catalogue: item master, units, barcodes, tax, modifiers, recipes, price books
M3   Inventory engine: stock ledger, batches/expiry, counts, transfers, costing
M4   POS terminal core: offline-first selling, shifts, cash, receipts, sync
M5   Payments: provider abstraction, pre-auth tabs, tips, refunds, reconciliation
M6   Purchasing, receiving, supplier invoices, three-way match
M7   Supermarket mode (S1–S15)
M8   Bar mode (B1–B13)
M9   Resto-bar and café mode (R1–R15)
M10  Finance core and ERP integration hub (E1–E9, Modes 1–3)
M11  Management layer: role dashboards, routines, tasks, staff app, scheduling
M12  Reporting, analytics and AI features
M13  Intelligence layer over existing POS systems (Mode 3)
M14  Hardening: security, performance, offline chaos tests, accessibility, usability, release
```

M0–M6 build the shared foundation. M7–M9 can be reordered to match your first customer (for example, bar before supermarket).

---

## Part B. `CLAUDE.md`, the project rules file

Copy everything inside the block below into `CLAUDE.md` at the repository root.

````markdown
# Unified POS + ERP Platform: Project Rules

## Project facts (fill in; "TBD" if unknown)
- Countries / tax jurisdictions: TBD
- Currencies: TBD
- Languages (staff UI and receipts): TBD
- First pilot sites: TBD (e.g. 1 supermarket, 1 bar, 1 resto-bar)
- Priority ERP / accounting connectors: TBD (e.g. Odoo, ERPNext, QuickBooks, Xero)
- Priority payment providers / acquirers / mobile money: TBD
- Fiscal / e-invoicing requirements: TBD

## What we are building
A cloud-native, offline-first POS with an ERP-grade back office, for supermarkets, bars, resto-bars and cafés.
It must beat typical cloud POS products on back-office control, and beat ERP POS modules on speed and hospitality/grocery depth.
It runs in three deployment modes: standalone (Mode 1); POS front end on an existing ERP (Mode 2); intelligence layer over an existing POS and ERP (Mode 3).

**The specification is `docs/concept-note.md`.** Process IDs (S1–S15 supermarket, B1–B13 bar, R1–R15 resto-bar/café, E1–E9 back office) and Section 7 (roles, permissions, management routines, usability) are the functional requirements. When code implements a process, reference its ID in the module README and in tests.

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
   - The milestone plan M0–M14 (use the list in the prompt guide as a starting point and adjust if the design suggests a better order). For each milestone: goal, process IDs covered, acceptance criteria that can be verified, and the main risks.
   - A traceability table mapping every process ID in the concept note (S1–S15, B1–B13, R1–R15, E1–E9) and every Section 7 subsection to the milestone that delivers it. Nothing may be left unmapped; mark items explicitly "Later" if they are out of the first release.

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

### M3. Inventory engine

```text
[Standard opening]

Milestone M3: inventory engine. Concept note: S11, S12 (tracking part), B1, B5 (engine part), B12 (container ledger), R10, R11, Section 5.1 D3.

Build:
- Stock ledger with movement types: receipt, sale depletion (by recipe), return, transfer out/in (with in-transit), adjustment, waste (with reason), production (consume ingredients, create output), count variance, container issue/return.
- Stock locations per site (shop floor, back store, cold room, bar stations, cellar, kitchen stations).
- Batch/lot and expiry tracking with FEFO suggestions; expiring-soon queries.
- Costing: weighted average cost as default; FIFO as an option per tenant (ADR); cost of goods sold per movement for later finance posting.
- Par levels per location; requisitions from store room to bar/kitchen.
- Counts: full and cycle counts (ABC scheduling), blind counts, partial bottles by tenths or by weight (item stores empty and full weights), recount rules, variance approval using the M1 permission engine.
- Theoretical vs actual usage and variance calculation per item, category, location, station and period (engine and API; dashboards come later).
- Returnable containers ledger (kegs, crates, gas cylinders, bottles) with deposits.

Out of scope: purchasing documents (M6), POS UI (M4).

Done when: property-based tests prove on-hand always equals the sum of movements, no negative stock unless the location allows it, costing is correct across receipts and sales, and variance equals theoretical minus actual on scripted scenarios (for example, a bar week with known over-pouring).
```

### M4. POS terminal core (offline-first)

```text
[Standard opening]

Milestone M4: POS terminal core. Concept note: S5, S10, S8 (basic refund), Section 8 design principles (offline, conflict rules), Section 7.7 and 7.8 (usability), 7.5 (permissions at the till).

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

Out of scope: tabs, tables, KDS, promotions, scales.

Done when: automated end-to-end tests cover selling, paying, refunding and closing a shift; an offline chaos test (network cut mid-sale, device restarts, two terminals offline for an hour, then reconnect) shows no lost or duplicated sales and correct stock; speed targets from CLAUDE.md are measured by an automated UI test and reported.
```

### M5. Payments

```text
[Standard opening]

Milestone M5: payments. Concept note: Section 10 (payments, security), B3, B8, S10 (settlement part), E4.

Build:
- Payment provider abstraction (ADR): authorise, capture, pre-authorise, incremental authorise, adjust tip, void, refund, store-and-forward offline with configured per-transaction and total limits, and settlement report import. Implement a full mock provider plus one real provider adapter chosen in docs/open-questions.md (or a sandbox-only adapter if none is chosen yet).
- Terminal integration patterns: semi-integrated P2PE terminals and SoftPOS (tap to phone) via the provider SDK. Card data must never reach our code; store tokens, masked PAN and references only.
- Tips: prompts, tip adjustment, auto-gratuity rules; data model for tip pooling (rules come in M8/M11).
- Mobile money and QR payment adapter interface with a mock implementation.
- Settlement reconciliation: match provider settlements, fees and chargebacks to shifts and bank deposits; unmatched items queue.

Out of scope: finance journal posting (M10).

Done when: contract tests pass for the provider interface; the mock provider simulates declines, timeouts, partial approvals, offline store-and-forward and late declines, and the POS handles each correctly; a documented PCI scope statement exists in docs/security/.
```

### M6. Purchasing, receiving and supplier invoices

```text
[Standard opening]

Milestone M6: purchasing. Concept note: S2, S3, B12 (purchasing part), E3, Section 5.3 (supplier and PO ownership in Mode 2).

Build:
- Suppliers with catalogues, pack sizes, MOQ, lead times, price history, delivery schedules (including DSD suppliers).
- Purchase orders with approval limits from the permission engine; send by email/PDF, with WhatsApp/EDI/portal adapters as interfaces plus at least email implemented.
- Suggested orders (rule-based in this milestone: par, min/max, sales velocity, lead time, MOQ, pack rounding), with an explanation per line. AI forecasting improves this in M12.
- Receiving on handheld: scan against PO, blind receive option, over/short/damaged/wrong item with photos, batch and expiry capture, supplier claim/debit note generation.
- Cost-change handling: new cost → margin impact → suggested retail price → approval → price change and label/ESL queue.
- Supplier invoices: capture by upload with OCR extraction (behind an interface; implementation may use the AI module later), three-way match with tolerances, exception routing for approval.
- Supplier scorecards.

Out of scope: accounts payable payment runs (M10).

Done when: end-to-end tests cover order → receive with discrepancy → claim → invoice match → approval; stock and cost update correctly in the ledger.
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
- Role home screens for host, server, runner, floor manager, head chef, line cook, expeditor, head barista, barista, delivery coordinator.

Done when: scripted demo "Dinner service" passes: reservations and walk-ins, 40 covers, coursing with a delayed main, an allergy order, a sold-out item synced to QR and delivery, split bills, delivery orders, and a café rush with order-ahead; ticket times and food cost calculate correctly.
```

### M10. Finance core and ERP integration hub

```text
[Standard opening]

Milestone M10: finance and integration. Concept note: E1–E9, Section 5 (5.2 deployment modes, 5.3 system of record, 5.4 integration principles), Section 11 (integrations).

Build (finance core for Mode 1, ERP-grade):
- Chart of accounts templates, double-entry journals (balanced by construction), fiscal periods with close/lock, multi-entity and multi-currency with exchange rates.
- Posting rules: sales, tax, tenders, tips, discounts, service charges, COGS, waste, count variances, cash over/short, settlements and fees; per transaction, per shift or daily summary.
- Accounts receivable (customer credit accounts, statements, ageing) and accounts payable (supplier invoices from M6, payment runs, ageing).
- Tax engine: multiple rates, inclusive/exclusive, exemptions, rounding per jurisdiction; tax reports.
- Fiscal/e-invoicing connector interface (per-country plug-ins) with a mock and one real implementation if the jurisdiction is known.
- Management reports: trial balance, P&L by location/department, balance sheet, budget vs actual, daily flash P&L, prime cost.

Build (integration hub for Modes 2 and 3):
- Connector framework: authentication, field mapping UI (items, taxes, tenders, GL accounts, cost centres, locations), sync schedules, idempotent messages, retries, dead-letter/error queue with fix-and-resend, reconciliation reports, full logging.
- System-of-record configuration per data object as in 5.3, enforced by the sync logic.
- Connectors in priority order from docs/open-questions.md. Default order: Odoo, ERPNext, QuickBooks Online, Xero; then Dynamics 365 Business Central and SAP Business One. Each connector ships with contract tests against a sandbox or recorded fixtures.
- Generic API/webhook and CSV/SFTP connectors.

Done when: every posting rule is covered by tests that assert balanced journals; a month of seeded demo trading closes cleanly with a trial balance that ties to sales, stock and cash reports; the first ERP connector passes a round trip (items in, sales/journals/POs out) with duplicate-delivery and failure-recovery tests.
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
Add support for [country]. Research the current VAT/GST/sales-tax rules, rounding rules, receipt and invoice requirements, and fiscal device or e-invoicing obligations from official sources, and list the sources in docs/compliance/[country].md. Implement tax configuration and a fiscal connector plug-in behind the existing interface. Ask me before any legal interpretation that is ambiguous.
```

---

## Part F. Quality gates and invariants

**Every milestone must pass these gates before merge:**

1. Lint, type-check, unit, integration and end-to-end tests green in CI.
2. Verify prompt (E.1) run; all medium+ findings fixed.
3. `docs/PROGRESS.md` updated, including traceability by process ID; ADRs for new decisions.
4. Demo seed data updated so the new features can be shown.
5. No new dependency without a stated reason; no secrets in the repository.

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
- **Review what matters most yourself:** money, tax, permissions, and anything that touches real card terminals or real customer data. Get a qualified accountant to review posting rules and tax, and a security professional to review M14 findings before going live.
