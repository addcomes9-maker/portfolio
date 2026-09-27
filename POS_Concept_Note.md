# Concept Note: A Unified Cloud POS for Supermarkets, Bars and Resto-Bars

| | |
|---|---|
| **Document type** | Concept note (draft for discussion) |
| **Version** | 0.5: adds **business documents and document flows** (Section 24): the full procurement → delivery → invoicing → payment cycle and the selling cycle, with each document's effect on stock and the books, and the document engine. v0.4 made **Finance, Inventory and Sales the three core pillars** of the platform (Sections 19–23): built-in accounting books, books health checks and error correction, perpetual inventory posted to the books, and a sales core with quick reports and insight-to-task. Sections 1–18 keep their numbers. v0.3 added management roles, users and usability for each service (Section 7) and the barista / coffee station process (R15). v0.2 added the process-by-process feature design and positioning over existing POS and ERP systems. |
| **Date** | September 2026 |
| **Scope** | Point-of-sale (POS) and store-management platform for three verticals: supermarkets / grocery, bars / pubs / lounges, and resto-bars (restaurant + bar hybrids) |
| **Basis** | (a) How these businesses run each process manually today; (b) the features of POS platforms in wide use (Toast, Square, Lightspeed, Clover, TouchBistro, Revel, Shift4/SkyTab, Oracle MICROS Simphony, NCR Voyix/Aloha, IT Retail, LOC, and bar-inventory tools such as BinWise and BevSpot); (c) the features of ERP systems with POS or retail modules (Odoo, ERPNext, LS Central on Microsoft Dynamics 365 Business Central, Microsoft Dynamics 365 Commerce, SAP, Oracle Retail Xstore). See **Sources**. |

---

## Contents

1. Executive summary
2. Background and problem statement
3. Landscape: existing POS and ERP systems
4. Vision and objectives
5. Positioning: on top of existing POS and ERP systems
6. Process-by-process feature design
   - 6.1 Supermarket / grocery
   - 6.2 Bar / pub / lounge
   - 6.3 Resto-bar
   - 6.4 Shared back-office and ERP processes
7. People: management roles, users and usability
   - 7.1 Who runs and uses the system in each business
   - 7.2 Role-by-role design: supermarket
   - 7.3 Role-by-role design: bar
   - 7.4 Role-by-role design: resto-bar and café
   - 7.5 Permissions and approval matrix
   - 7.6 Management routines built into the system
   - 7.7 Usability by working environment
   - 7.8 Usability principles and accessibility
   - 7.9 Staff experience and adoption
8. Solution architecture and core modules
9. Hardware concept
10. Payments, security and compliance
11. Integrations and ERP connectors
12. Key reports and KPIs
13. Implementation approach
14. Commercial model
15. Indicative cost components
16. Risks and mitigation
17. Success measures
18. Conclusion and next steps
19. The three core pillars: Finance, Inventory and Sales
20. Finance core (F1–F19)
21. Inventory core (IN1–IN11), working with Finance
22. Sales core (SA1–SA12), working with Inventory and Finance
23. How the three cores work together
24. Business documents and document flows (DOC1–DOC12)
- Appendix A: Feature matrix by mode
- Appendix B: Glossary
- Sources

---

## 1. Executive summary

Modern POS is no longer a cash register with a screen. It is the operating system of the store or venue: it takes payments, but it also runs inventory, staff, customers, kitchen and bar workflows, online channels and reporting, and it increasingly uses AI to forecast demand and flag loss.

The market is split. **Cloud POS products** (Square, Toast, Lightspeed, Clover) are fast and easy to use at the counter, but their back office is thin. Purchasing controls, accounting, supplier invoice matching and multi-entity finance are basic or depend on add-ons. **ERP systems with POS modules** (Odoo, ERPNext, LS Central / Business Central, Dynamics 365 Commerce, SAP) have strong back offices, but they are usually harder to set up, slower at the till, and weak on bar- and venue-specific workflows. **Specialist tools** (liquor-inventory apps, grocery scale and label systems, reservation platforms) fill the gaps, which leaves operators with many disconnected systems.

This concept note proposes a **single cloud-native, offline-first POS platform** that aims to sit **on top of** both categories:

- **Better than existing POS:** ERP-grade controls (approvals, three-way matching, audit, multi-location finance) behind a POS-grade user experience.
- **Better than ERP POS modules:** fast checkout and service screens, and deep supermarket, bar and resto-bar workflows.
- **Able to run on top of what the customer already has:** it can run as a complete system, as a front end to an existing ERP, or as an **intelligence and control layer** over an existing POS and ERP (Section 5).

At the heart of the platform are **three core pillars, built in this order: Finance, Inventory and Sales** (Sections 19–23).
- **Finance** keeps the standard accounting books (cash book, bank book, petty cash, day books, ledgers, registers). It tracks income and expenses, and it runs daily health checks that find and help correct financial errors.
- **Inventory** records every stock movement with quantity and value and posts it to the books.
- **Sales** turns every sale into correct stock and correct books, and into quick reports and simple tasks for owners and managers.

The three cores reconcile with each other every day, so the financial figures shown are always backed by stock and sales records.

Every event in the cores starts from a **business document** (Section 24):
- commercial documents: requisition, RFQ, quotation, proforma, purchase and sales orders;
- delivery documents: delivery note, goods received note;
- billing documents: tax invoice, advance invoice, credit and debit notes;
- payment documents: payment voucher, receipt;
- adjustments and statements of account.

They link into complete flows for buying and selling (prepaid, postpaid and direct cash), so every figure can be traced back to its paperwork.

Around the cores, the platform has shared modules (payments, customers, staff, reporting) and **three vertical modes**: Supermarket, Bar and Resto-bar. A business switches on one or more. Section 6 designs every major business process in detail: how it is done manually today, what existing POS and ERP systems provide, where the gaps are, and the specific features this platform will provide. Section 7 designs the system around **the people who manage and use it**: owners, store, bar and restaurant managers, supervisors, cashiers, bartenders, baristas, servers, chefs, storekeepers and accountants. It covers role-specific screens, permissions, approval limits, daily management routines, and usability in each working environment, from a dark, noisy bar to a hot kitchen or a busy checkout lane.

**Expected benefits** (targets to validate in the pilot):

- Checkout / service speed up 20–30% through speed screens, handhelds, scale integration and self-service.
- Inventory shrink and liquor variance reduced by tying every sale to stock depletion and surfacing variance daily.
- Stock-outs and perishable waste reduced through AI-assisted ordering, expiry tracking and automatic markdowns.
- No double entry between POS and ERP: one item master, automatic posting of sales, costs and cash to accounts.
- Owners get real-time, multi-location visibility from a phone; managers approve voids, refunds and orders remotely.
- Every role is productive quickly: new frontline staff within one hour, with screens designed for their job and workplace.
- Payment security and compliance handled by design (PCI DSS v4.0, P2PE / tokenisation, local fiscal / e-invoicing rules).

---

## 2. Background and problem statement

### 2.1 What operators struggle with today

| Segment | Typical pain points |
|---|---|
| **Supermarkets / grocery** | Long queues at peak hours; manual price changes and shelf-label mismatches; weighted produce keyed by hand (errors, slow); perishables expiring unnoticed; stock counts that never match the system; promotions that are hard to set up and track; many SKUs (5,000–50,000+) with poor data; cashier fraud (voids, "sweethearting"); separate systems for scales, accounting, loyalty and e-commerce. |
| **Bars / pubs / lounges** | Walk-outs on unpaid tabs; slow service at the bar on busy nights; over-pouring, free drinks and theft (liquor is the highest-value, easiest-to-lose stock); no link between what was sold and what left the bottle; manual happy-hour price switching; tip disputes; underage sales risk; empty kegs and crates not tracked. |
| **Resto-bars** | Orders lost between front-of-house, kitchen and bar; paper tickets; wrong course timing; complex bill splitting; separate tablets for each delivery platform; separate tools for reservations and loyalty; food and beverage costs tracked in spreadsheets, if at all. |
| **All** | Legacy on-premise systems with no remote access; downtime when the internet drops; data typed twice into POS and accounting/ERP; poor reporting; payment-security exposure; hardware lock-in; high card-processing costs that are hard to see. |

### 2.2 Why now

- **Cloud is the default.** Industry data indicates that more than 70% of new POS installations are cloud-based. Hospitality and quick-service lead adoption; traditional grocery is catching up as legacy systems are replaced.
- **Payments have moved to contactless and mobile.** Tap-to-pay, digital wallets and QR/mobile-money payments are now expected. *SoftPOS* (accepting cards on an ordinary NFC phone) removes the need for dedicated terminals for some use cases.
- **Security rules have tightened.** PCI DSS v3.2.1 was retired on 31 March 2024. The future-dated requirements of PCI DSS v4.0 became mandatory on 31 March 2025, which pushes merchants towards encrypted (P2PE/E2EE) and tokenised card handling.
- **AI has become practical.** Demand forecasting, automated reorder suggestions, camera-based produce recognition at self-checkout, and anomaly detection on voids and refunds now ship in commercial products (for example, NCR Voyix Insight and Picklist Assist).
- **Integration expectations have risen.** Retail and hospitality groups increasingly expect POS and ERP to share one data model (LS Central on Business Central, Odoo, Dynamics 365 Commerce) rather than exchanging files overnight.

---

## 3. Landscape: existing POS and ERP systems

### 3.1 Categories of systems reviewed

| Category | Examples | Strengths | Typical gaps |
|---|---|---|---|
| **A. Cloud retail POS** | Square for Retail, Lightspeed Retail, Clover, Shopify POS | Fast setup, good UI, integrated payments, basic inventory and loyalty, app marketplaces | Weak for high-volume grocery (scales, ESL, complex promotions); limited purchasing controls and accounting; payments often tied to the vendor |
| **B. Cloud hospitality POS** | Toast, Square for Restaurants, Lightspeed Restaurant, TouchBistro, Shift4/SkyTab, Revel | Speed screens, tabs with pre-authorisation, floor plans, KDS, online ordering, handhelds | Liquor variance often needs third-party tools; basic purchasing and costing; retail/grocery not covered; accounting through integrations only |
| **C. Enterprise hospitality POS** | Oracle MICROS Simphony, NCR Voyix Aloha | Scale, multi-venue, hotel/PMS integration, forecasting, mobile inventory counts, labour management | High cost and complexity; long implementations; partner-dependent |
| **D. Enterprise retail / grocery POS** | NCR Voyix, Oracle Retail Xstore, LS Central, IT Retail, LOC, LogicERP | Lanes, scales, self-checkout, promotions, loss prevention, replenishment | Expensive; often on-premise heritage; hospitality weaker (except LS Central) |
| **E. ERP-integrated POS** | Odoo POS, ERPNext POS, LS Central on Business Central, Dynamics 365 Commerce, SAP Business One add-ons | One data model with accounting, purchasing and inventory; recipes/BoMs; approvals; audit | Slower till UI; offline mode weaker in some (ERPNext offline sync has known conflict issues, and community apps are often used instead); bar features thin; needs skilled implementers |
| **F. Specialist add-ons** | BinWise, BevSpot, Backbar (liquor); ESL vendors; scale systems; reservation platforms | Deep in one area | Another login, another data silo, another subscription |

### 3.2 What the market already does well (the baseline we must match)

1. Cloud back office, with local apps on terminals that keep selling offline and sync later.
2. Integrated payments: EMV chip, contactless/NFC, wallets, QR payments, and store-and-forward offline card acceptance.
3. Real-time inventory with automatic depletion from sales, including recipes and bills of materials (for example, Odoo deducts ingredients by BoM when a dish is sold).
4. Catalogue or menu management with modifiers, variants and time-based pricing.
5. Customer profiles, loyalty points, gift cards and marketing.
6. Staff management: PIN login, roles and permissions, time clock, tip pooling.
8. Kitchen and bar display (for example, Toast KDS, Lightspeed KDS, Odoo preparation display).
9. Self-ordering by QR code and kiosk.
10. Real-time dashboards on mobile; multi-location roll-ups.
11. Open APIs and marketplaces for accounting, delivery, e-commerce, payroll and reservations.

### 3.3 Where existing systems commonly fall short (our opportunity)

| # | Gap | Who has it |
|---|---|---|
| G1 | No single product serves **grocery, bar and restaurant** well together (for example, a supermarket with a café bar, or a hotel with a lounge and a mini-mart) | A, B, C, D |
| G2 | **Double entry** between POS and ERP, overnight batch syncs, and mismatched item and customer masters | A, B, F |
| G3 | **Back-office controls** (purchase approvals, three-way match, credit limits, audit) missing or basic | A, B |
| G4 | **Till speed and offline resilience** weaker than dedicated POS | E |
| G5 | **Liquor variance** requires separate apps and manual reconciliation | A, B, E |
| G6 | **Grocery specifics** (scales, ESL, expiry markdowns, vendor-funded promotions) missing or expensive | A, B, E |
| G7 | **Local fit** (mobile money, local fiscal devices and e-invoicing, local languages, low-bandwidth operation) often poor in global products | A, B, C |
| G8 | **Cost and lock-in**: processing tied to the vendor, expensive enterprise licences, data hard to export | A, B, C, D |
| G9 | **Insight, not just reports**: most systems show what happened but rarely say what to do next (what to reorder, which bartender's variance to investigate, which promotion lost money) | All |

---

## 4. Vision and objectives

**Vision:** *One platform that lets any supermarket, bar or resto-bar sell faster, lose less, and know exactly how the business is doing, from anywhere, whether it replaces the existing systems or works on top of them.*

| # | Objective | Indicative target (to validate in pilot) |
|---|---|---|
| O1 | Faster transactions | Average grocery checkout time −20%; bar drink-order-to-payment under 30 s |
| O2 | Reduce loss | Liquor variance kept under 5% (industry practice treats more than 5% as a trigger for investigation and more than 10% as a trigger for an immediate audit); grocery shrink visibly reduced against baseline |
| O3 | Reduce waste and stock-outs | Perishable write-offs −15%; out-of-stock incidents −25% |
| O4 | Eliminate double entry | 100% of sales, receipts, stock movements and cash automatically posted to the ERP/accounting system |
| O5 | Real-time visibility | Sales, stock and staff data visible on the owner dashboard within 1 minute |
| O6 | Resilience | Zero lost sales during internet outages (offline mode) |
| O7 | Compliance | PCI DSS v4.0 aligned; local tax/fiscal rules met from day one |
| O8 | Adoption | New cashier or bartender productive after 1 hour or less of training |
| O9 | Books always correct | Sales, inventory and finance reconcile daily with zero unexplained differences; every detected finance issue has an owner and a suggested fix; month-end close in 2 days or less |

---

## 5. Positioning: on top of existing POS and ERP systems

"On top" has two meanings in this concept, and the platform is designed for both.

### 5.1 Meaning 1: ahead of existing products (differentiators)

| # | Differentiator | Addresses gap |
|---|---|---|
| D1 | **One platform, three modes.** Supermarket, bar and resto-bar on one item master, one inventory engine and one report set. | G1 |
| D2 | **ERP-grade controls with POS-grade speed.** Purchase approvals, three-way matching, credit control, audit trail, and multi-entity accounting, all behind a till that adds an item in under a second. | G3, G4 |
| D3 | **Inventory engine for every unit type.** It handles pieces, cases, kilograms, millilitres, kegs and recipes. The same engine tracks a 750 ml bottle down to the pour and a cheese wheel down to the gram, with batch and expiry. | G5, G6 |
| D4 | **Built-in loss control.** Liquor variance, grocery shrink, voids/refunds/comps and cash over/short, analysed per staff member, shift and item, with AI alerts. | G5, G9 |
| D5 | **Offline-first with safe sync.** Every terminal works without internet; conflicts are resolved by clear rules (Section 8). | G4 |
| D6 | **Local fit.** Pluggable fiscal/e-invoicing connectors, mobile money and QR payments, multi-currency, multi-language, low-bandwidth mode. | G7 |
| D7 | **Open by design.** Processor-agnostic payments, full API, data export, and customer ownership of data. | G8 |
| D8 | **Recommendations, not just reports.** Suggested orders, suggested markdowns, variance investigations, promotion post-mortems and natural-language questions. | G9 |
| D9 | **Manager on the phone.** Approvals (voids, refunds, price overrides, POs) and alerts on a mobile app, so the manager doesn't have to walk to the till. | G3 |

### 5.2 Meaning 2: running on top of a customer's existing systems (deployment modes)

Many target customers already have a POS, an ERP, or both. The platform therefore supports three deployment modes, so a customer can adopt it gradually rather than all at once.

| Mode | What the customer keeps | What our platform does | Typical customer |
|---|---|---|---|
| **1. Standalone** | Nothing (or just their accountant's package) | Full POS + back office + reporting; exports to accounting | Independent bar, resto-bar or supermarket; new openings |
| **2. POS front end on an existing ERP** | Their ERP (Odoo, ERPNext, SAP Business One, Dynamics 365 Business Central, Sage, Tally, QuickBooks/Xero) as the financial system of record | Our POS runs the stores and venues; item master, prices, customers and suppliers sync with the ERP; sales, cash, stock movements and COGS post automatically to the ERP | Chains and groups that have invested in an ERP but have a weak POS module |
| **3. Intelligence and control layer on existing POS + ERP** | Both their POS and their ERP | We pull sales, voids, refunds and stock data via APIs or exports; provide variance, forecasting, suggested orders, loss-prevention alerts, consolidated multi-system dashboards, and push POs or journals back to the ERP | Groups with mixed systems (for example, Toast in the restaurant, a legacy grocery POS in the shop), or customers under contract with another POS |

**Migration path:** a customer can start with **Mode 3** (insight on top of current systems, low risk), move to **Mode 2** (replace the tills while keeping the ERP), and optionally reach **Mode 1**.

### 5.3 System-of-record matrix (who owns which data in each mode)

| Data object | Mode 1 | Mode 2 | Mode 3 |
|---|---|---|---|
| Item / menu master, barcodes | Our platform | ERP or our platform (chosen at setup); synced both ways | Existing POS / ERP (we read) |
| Retail prices, promotions | Our platform | Our platform (store-level); ERP price lists as the base | Existing POS (we recommend changes) |
| Recipes / BoMs | Our platform | Our platform or ERP BoM | Our platform (we map POS items to recipes) |
| Stock quantities | Our platform | Our platform per store; ERP for warehouses; synced | ERP / POS (we calculate theoretical stock and variance) |
| Customers & loyalty | Our platform | Our platform; ERP for credit accounts / receivables | Existing systems (we consolidate) |
| Suppliers, POs, supplier invoices | Our platform | ERP (we create POs/GRNs; ERP does invoice and payment) | ERP (we suggest POs and push them as drafts) |
| General ledger, tax filing | Accounting export | ERP | ERP |
| Sales transactions | Our platform | Our platform, posted to ERP as daily summaries or per invoice | Existing POS (we import) |

### 5.4 Integration design principles for Modes 2 and 3

- **Connectors, not custom code:** prebuilt connectors for the most common ERPs and POS systems (Section 11), plus a generic API, webhook and CSV/SFTP route for everything else.
- **Mapping layer:** field mapping for items, taxes, tenders, GL accounts, cost centres and locations, maintained in a visual screen, not code.
- **Posting options:** per transaction (real time), per shift, or daily summary journals, chosen per customer.
- **Idempotent and auditable:** every sync message has a unique ID, retries safely, and is logged with a success or failure status and an error queue that a human can fix and resend.
- **Conflict rules:** the system of record wins for its objects; offline sales are always kept, never overwritten; stock is corrected with an adjustment movement, not by overwriting the quantity.

---

## 6. Process-by-process feature design

Each process below is described in the same way:

- **Manual today:** how small and mid-size operators typically do it without a modern system.
- **Existing POS:** what current POS products usually provide.
- **Existing ERP:** what ERP systems with POS or retail modules usually provide.
- **Gap:** what is still missing.
- **Our features:** what this platform will provide. This is the functional requirement for design.

### 6.1 Supermarket / grocery

#### S1. Item master and new product setup
- **Manual today:** Supplier price lists on paper; price stickers written by hand; items without barcodes rung up as "open department" (just a price), so nobody knows what was actually sold.
- **Existing POS:** Item catalogue with barcode, price, category and tax; CSV import; basic variants.
- **Existing ERP:** Full item master: multiple units of measure (UoM), suppliers, costs, tax codes, GL accounts, lot/expiry tracking settings.
- **Gap:** Items are keyed twice (POS and ERP) and drift apart. Many POS products can't hold several barcodes or pack sizes per item. Open-department sales hide real sales data.
- **Our features:**
  - Single item master shared across POS, inventory and accounting (or synced with the ERP in Mode 2).
  - Barcode (GTIN) lookup to pre-fill name, brand and size from product databases; supplier catalogue import.
  - Multiple barcodes and pack levels per item (each / inner / case / pallet) with automatic conversion.
  - Item attributes: department, category, brand, tax code, age restriction, weighed/unit, shelf life, storage (ambient/chilled/frozen), deposit/returnable container.
  - New-item approval workflow (buyer creates, manager approves) and duplicate detection.
  - Open-department sales limited by role and require a reason, with a "convert to item" report.

#### S2. Purchasing and replenishment
- **Manual today:** Owner walks the aisles with a notebook, then phones or WhatsApps suppliers and orders by gut feeling. Over-stocking of slow items; stock-outs of fast ones.
- **Existing POS:** Low-stock alerts; simple purchase orders; reorder points in higher tiers.
- **Existing ERP:** Min/max reorder rules, supplier price lists, lead times, approval workflows, purchase agreements. Retail ERPs such as LS Central add automated replenishment.
- **Gap:** Reorder rules are static (ignore seasonality, promotions, holidays and shelf life) and only as good as the stock figures. Small stores can't configure ERP replenishment.
- **Our features:**
  - AI-suggested orders based on sales velocity, day-of-week pattern, seasonality and holidays, planned promotions, shelf life, supplier lead time, minimum order quantity (MOQ) and pack size.
  - The order suggestion screen shows *why* (for example, "promotion next week +40%") and lets the buyer adjust.
  - One-click PO sent by email, WhatsApp, supplier portal or EDI; PO approval limits by amount and role.
  - Direct-store-delivery (DSD) vendors (bread, milk, soft drinks) with visit schedules.
  - Supplier scorecards: fill rate, on-time delivery, price changes, credit-note history.
  - In Mode 2, the PO is created in the ERP (or pushed to it) so purchasing stays under ERP controls.

#### S3. Goods receiving
- **Manual today:** Staff count boxes against the delivery note, sign it and file the paper. Short deliveries and damaged goods are discovered later and never claimed.
- **Existing POS:** Receive against a PO in the back office, usually at a desk computer.
- **Existing ERP:** Goods received note (GRN), quality check, three-way match (PO / GRN / supplier invoice), landed cost.
- **Gap:** Receiving happens at the dock, not at the desk. Discrepancies are not captured with evidence. Cost changes do not flow through to retail prices and margins.
- **Our features:**
  - Mobile receiving on a handheld: scan each item against the PO, with an optional "blind receive" (quantity not shown to the receiver).
  - Capture short, over, damaged and wrong items with photos; generate a supplier claim / debit note automatically.
  - Capture batch number and expiry date at receipt for perishables.
  - Cost-change alert: when the supplier's cost changes, show the new margin and suggest a new retail price; on approval, queue shelf-label or ESL updates.
  - Supplier invoice capture (photo/PDF with OCR) and three-way match; exceptions routed for approval. In Mode 2, the GRN and invoice post to the ERP.

#### S4. Pricing, labels and promotions
- **Manual today:** Price-gun stickers and paper shelf tags; shelf price and till price don't match; promotions posted on paper and applied by the cashier by hand.
- **Existing POS:** Price book, simple discounts, some mix-and-match.
- **Existing ERP:** Price lists and discount rules; retail-specific ERPs have stronger promotion engines, but general ERPs are weak here.
- **Gap:** Complex promotions and label printing are not joined up; vendor-funded promotions are not claimed back; nobody measures whether a promotion made money.
- **Our features:**
  - Central price book with store price zones and scheduled price changes (effective date/time).
  - Promotion engine: percentage/amount off, multi-buy (3 for 2), mix-and-match across items, buy X get Y, basket threshold ("spend 50, save 5"), member-only prices, coupons, time-of-day offers. Includes stacking rules, priority and budget caps.
  - Vendor-funded promotions with an automatic supplier claim report.
  - Shelf labels: auto-generated print batches for changed prices (including unit price per kg/litre where required), or real-time push to electronic shelf labels (ESL).
  - Promotion post-analysis: sales uplift, cannibalisation of other items, margin impact, and stock left over.

#### S5. Checkout for scanned items
- **Manual today:** Calculator or cash book; mental arithmetic; long queues.
- **Existing POS:** Scan, total and pay; suspend/resume; price override with a manager.
- **Existing ERP:** POS screens exist but are slower and have less hardware support; offline sync can be fragile.
- **Gap:** Peaks cause queues; approvals need a manager to walk over; outages stop trading.
- **Our features:**
  - Scanner- and keyboard-first lane UI; item added in under 1 second; quantity multiplier; quick keys for non-barcoded items.
  - Suspend and recall baskets on any lane; queue-busting handhelds that pre-scan baskets in the queue.
  - Remote manager approval on the phone for overrides, voids and refunds.
  - Customer-facing display with running total and promotions applied.
  - Receipts printed, emailed, sent by SMS/WhatsApp, or as a QR code; fiscal receipt where required.
  - Full offline operation (Section 8).

#### S6. Weighed produce and fresh counters
- **Manual today:** Produce weighed on a separate scale, price written on the bag, cashier types it in.
- **Existing POS:** Scanner-scale integration (weight sent to the POS), PLU lookup, and support for price-embedded barcodes printed by label scales.
- **Existing ERP:** Rarely handles scales directly; needs add-ons.
- **Gap:** Label scales hold their own price files that drift from POS prices; tare errors; slow PLU lookup.
- **Our features:**
  - Certified checkout scanner-scales and deli label scales integrated. Prices and PLUs are pushed from our price book to the scales, so they always match.
  - Tare library (containers, trays) and variable-weight barcode parsing (weight- or price-embedded).
  - Visual PLU picker with pictures and search; camera-based produce recognition that suggests the top matches.
  - Compliance with local weights-and-measures (legal-for-trade) requirements.

#### S7. Deli, bakery and in-store production
- **Manual today:** Recipes kept in the baker's head; production not recorded; unsold goods thrown away unrecorded.
- **Existing POS:** Sells finished items only.
- **Existing ERP:** Manufacturing orders and BoMs, usually too heavy for a store bakery.
- **Gap:** In-store production isn't linked to ingredient stock or to demand.
- **Our features:**
  - Simple production batches ("bake 60 baguettes") that consume ingredients and create finished stock.
  - Daily production plan suggested from the sales forecast.
  - End-of-day waste recording and automatic evening markdowns for fresh items.
  - Label printing with ingredients, allergens and best-before date.

#### S8. Returns, refunds and exchanges
- **Manual today:** Cash taken from the drawer with no record.
- **Existing POS:** Refund against the receipt, with manager approval.
- **Existing ERP:** Credit notes; return to vendor (RTV).
- **Gap:** Refund fraud; returned goods not handled properly afterwards (resell, write off, or return to supplier).
- **Our features:**
  - Receipt lookup by QR code, card or phone number; refund to the original tender.
  - Reason codes; limits on refunds without a receipt; fraud rules (for example, many refunds by one cashier or one customer).
  - Disposition of returned goods: back to stock, damaged/write-off, or return to vendor with an RTV document and supplier credit claim (posted to ERP in Mode 2).

#### S9. Age-restricted and regulated items
- **Manual today:** Cashier judgement.
- **Existing POS:** Date-of-birth prompt on flagged items.
- **Our features:**
  - Mandatory item flags (alcohol, tobacco, knives, medicines) with a required ID check or ID scanner.
  - Sale-hour restrictions per licence and per location.
  - Log of every check (for inspections); attendant approval at self-checkout.
  - Tender eligibility by item where government benefit or voucher programmes apply (for example, EBT/SNAP in the US).

#### S10. Cash management and till closing
- **Manual today:** Cashier counts the cash and writes the total on paper; differences are never explained.
- **Existing POS:** Open/close shift, X and Z reports, cash drops.
- **Existing ERP:** Cash and bank accounts, bank reconciliation.
- **Gap:** Blind close not enforced; safe and float movements not tracked; card settlements reconciled in spreadsheets.
- **Our features:**
  - Float issue at shift start; cash-drop prompts when the drawer exceeds a limit; safe management.
  - Blind close: cashier counts by denomination before seeing the expected amount.
  - Over/short by cashier with trend alerts.
  - Automatic reconciliation of card and mobile-money settlements against processor reports, including fees.
  - Bank deposit slips; posting to ERP cash and bank accounts.

#### S11. Stock counts and shrink
- **Manual today:** The store closes once a year for a paper count.
- **Existing POS:** Count sheets or handheld counts.
- **Existing ERP:** Physical inventory, cycle counts, adjustments posted to accounts.
- **Gap:** Full counts are disruptive, and there's no root-cause analysis of shrink.
- **Our features:**
  - ABC-based cycle counting: high-value or high-risk items counted weekly, others monthly or quarterly, with no store closure.
  - Handheld counting with blind quantities and recount rules for large variances.
  - Variance approval workflow; shrink classified by reason (theft, damage, admin error, supplier short).
  - High-risk item watchlist (razors, spirits, baby formula) with more frequent counts.

#### S12. Expiry, waste and markdowns
- **Manual today:** Staff check shelves occasionally and throw expired items away.
- **Existing POS:** Few track expiry.
- **Existing ERP:** Lot and expiry tracking with first-expired-first-out (FEFO), mainly used in warehouses.
- **Our features:**
  - Expiry captured at receiving; a daily "expiring soon" task list on the handheld, organised by aisle.
  - Automatic markdown rules (for example, −30% two days before expiry, −50% on the day). The system prints a reduced-price label with its own barcode, or updates the ESL.
  - Waste recorded by reason; donation logging; waste value per department in reports.

#### S13. Customer accounts, loyalty and credit
- **Manual today:** Credit for regular customers kept in a notebook.
- **Existing POS:** Loyalty points and customer profiles.
- **Existing ERP:** Accounts receivable, credit limits and statements.
- **Gap:** POS systems lack credit control; ERP systems lack loyalty.
- **Our features:**
  - Customer credit accounts with limits, a hold when over the limit, statements, and payment on account (posted to ERP receivables in Mode 2).
  - Wholesale / B2B price tiers for business customers (restaurants buying from the supermarket).
  - Loyalty points, tiers and personalised offers based on purchase history; digital receipts; opt-in marketing.

#### S14. Online orders, click and collect, and delivery
- **Manual today:** Orders by phone or WhatsApp, picked from the shelf, often out of stock.
- **Existing POS:** Integration with an online store (Shopify, WooCommerce, own store).
- **Existing ERP:** Sales orders and delivery notes.
- **Our features:**
  - Shared stock between shop and online store; picking app with route by aisle and a substitution workflow (customer approves by message).
  - Final price adjusted for weighed items after picking.
  - Delivery and collection slots with capacity limits; pay online or on collection.

#### S15. Multi-store and head-office control
- **Existing POS:** Multi-location reporting.
- **Existing ERP:** Strong multi-warehouse, inter-company and transfer handling.
- **Our features:**
  - Head-office console: central item and price management, store-specific range (assortment), store price zones.
  - Inter-store and warehouse-to-store transfers with in-transit stock.
  - Store comparison dashboards; head-office push of promotions and planograms.

### 6.2 Bar / pub / lounge

#### B1. Opening, par stock and requisitions
- **Manual today:** Bartender checks the shelves, fetches bottles from the store room, writes a requisition slip (or doesn't).
- **Existing POS:** No support.
- **Existing ERP:** Internal transfers between locations.
- **Gap:** Stock moves between store room and bar aren't recorded, so variance can't be traced to a location.
- **Our features:**
  - Par levels per bar station; the system suggests a restock list at opening.
  - Requisition from store room to bar as a location transfer (scan or tap), approved by the manager.
  - Opening checklist (float issued, fridges stocked, ice, garnish prep) on the tablet.

#### B2. Taking drink orders at the counter
- **Manual today:** Orders remembered or shouted; cash kept in a jar or open drawer.
- **Existing POS:** Quick screens and modifiers; Toast and others offer bar "speed screens".
- **Gap:** Still slow on peak nights; shared terminals mean bartenders ring sales under each other's names.
- **Our features:**
  - Speed screen: top 20–40 drinks on one page, auto-arranged by what sells at that time of night.
  - One-tap "repeat round"; drink modifiers (double, mixer choice, ice, garnish) with pricing.
  - Fast bartender switching at a shared terminal (NFC wristband/card or PIN) so each sale is attributed correctly.
  - Bar display screen (BDS) for cocktail queues on busy nights.

#### B3. Tabs
- **Manual today:** Paper tab slips; customers' cards kept behind the bar (a security risk); walk-outs.
- **Existing POS:** Open a tab with a card pre-authorisation (Toast, SkyTab and others); name tabs.
- **Existing ERP:** No support.
- **Our features:**
  - Open a tab by tapping or dipping a card with pre-authorisation, plus incremental authorisation as the tab grows.
  - Tab limit alerts; transfer a tab from bar to table; merge and split tabs.
  - Guest views and pays their tab from their phone by QR code.
  - Auto-close rules at end of night with default tip rules, where the law allows.
  - Abandoned-tab and walk-out report by bartender.

#### B4. Recipes and pour standards
- **Manual today:** Free pouring or jiggers; no recipe cards; every bartender makes it differently.
- **Existing POS:** Some offer recipe-based depletion (for example, Odoo BoMs; Toast and Lightspeed with inventory add-ons).
- **Our features:**
  - Recipe library: ingredients in ml/oz, glass, garnish, method, photo and cost, shown on the BDS or tablet.
  - Standard pour size per spirit category and a price per pour.
  - Unit conversions: bottle sizes (700/750/1000 ml), kegs (litres to pints or glasses), wine by the glass (for example, 5 or 6 glasses per bottle), and soft-drink post-mix.
  - Live cost and margin per drink; alert when a supplier price change pushes pour cost above target.

#### B5. Liquor stock counts and variance
- **Manual today:** Weekly or monthly visual count estimating each bottle in tenths; results never compared with sales.
- **Specialist tools:** BinWise, BevSpot and Backbar weigh or scan bottles and compare with POS sales.
- **Existing ERP:** Stock counts, but no concept of partial bottles.
- **Gap:** Variance needs a separate app and manual reconciliation; nobody sees it until month end.
- **Our features:**
  - Partial-bottle counts by weight (Bluetooth scale: the system knows the empty and full weight of each bottle) or by tenths.
  - Theoretical usage (from POS sales × recipes) versus actual usage (from counts), per product, category, bar station and shift.
  - Variance shown in quantity, value and %, with thresholds (investigate above 5%, audit above 10%).
  - Optional real-time monitoring with smart pour spouts and keg flow meters.
  - Suggested causes (over-pouring, unrung sales, unrecorded comps, receiving errors) and a follow-up task log.

#### B6. Comps, spills, staff drinks and giveaways
- **Manual today:** Not recorded, so they show up as unexplained variance.
- **Existing POS:** Comp with reason and manager approval.
- **Our features:**
  - Reason codes (spill, returned drink, promotion, VIP, staff drink) that deplete stock at zero price, so variance stays explainable.
  - Comp budgets per manager/bartender per shift or month, with alerts.
  - Staff drink allowances by role.

#### B7. Happy hour and time-based pricing
- **Manual today:** Bartender changes prices by memory; mistakes both ways.
- **Existing POS:** Time-based price rules in most hospitality POS.
- **Our features:**
  - Price schedules by day and time, per location and per item group, applied automatically (tabs opened before happy hour are handled by clear rules).
  - Event pricing (match nights, live music); report on happy-hour uplift and margin.

#### B8. Tips and gratuity
- **Manual today:** Tip jar split by hand; disputes.
- **Existing POS:** Tip prompts; tip pooling in some; payroll export through integrations.
- **Our features:**
  - Tip prompts on terminals and QR payments; auto-gratuity for large groups.
  - Tip-pool rules by role, hours worked or points; tip-out to barbacks and runners.
  - Cash tip declaration; tip reports for payroll and tax, following local law.

#### B9. Door, cover charge, events and capacity
- **Manual today:** Cash box and hand stamps at the door.
- **Existing POS:** Usually a separate ticketing tool.
- **Our features:**
  - Cover charge sold on a handheld at the door; ticket and guest-list scanning.
  - Live capacity counter (in/out) against licensed capacity.
  - VIP table bookings with deposits.

#### B10. Bottle service and minimum spend
- **Manual today:** Minimum spend tracked on paper.
- **Our features:**
  - Table packages (bottle + mixers) that deplete the right stock.
  - Minimum-spend tracking with a live balance and automatic top-up line at close.
  - Deposits applied to the final bill.

#### B11. Responsible service and licensing
- **Our features:**
  - ID prompts and scanner integration; last-call and licensed-hours lockouts (manager override logged).
  - Incident log (refusals, disturbances) for licence inspections.

#### B12. Beverage purchasing, kegs and returnable containers
- **Manual today:** Phone orders to distributors; empty kegs and crates not tracked, so deposits are lost.
- **Existing ERP:** Purchase orders; deposit handling requires customisation.
- **Our features:**
  - Par-based suggested orders per distributor; distributor price-change tracking.
  - Returnable container ledger (kegs, gas cylinders, crates, bottles) with deposits paid and refunded.
  - Excise and licence purchase records where the law requires them.

#### B13. Closing and bartender cash-out
- **Manual today:** Cash counted on the bar top; tips split by hand.
- **Existing POS:** Server/bartender checkout reports.
- **Our features:**
  - Forced close or transfer of all open tabs before the shift can end.
  - Bartender cash-out report (sales by tender, tips, cash owed to the house), blind count, and manager sign-off.
  - End-of-night checklist and closing stock count for high-value lines.

### 6.3 Resto-bar

Resto-bar mode includes all bar processes (B1–B13) plus the following.

#### R1. Reservations, walk-ins and waitlist
- **Manual today:** Reservation book; walk-ins told "about 20 minutes".
- **Existing POS:** Integrated or partner reservations (for example, via OpenTable-style tools); Simphony and LS Central include table reservations.
- **Our features:**
  - Native reservations with deposits and no-show fees; online booking widget.
  - Waitlist with SMS/WhatsApp notification and quoted wait times based on actual table turn times.
  - Guest notes (allergies, preferences, occasions) visible when the guest is seated.

#### R2. Seating and floor management
- **Manual today:** Host remembers which tables are free.
- **Existing POS:** Floor plans with table status (Toast, Lightspeed, Odoo and others).
- **Our features:**
  - Drag-and-drop floor plans per area (dining room, terrace, bar); combine and split tables.
  - Live table status (seated, ordered, mains served, bill requested, paid) with timers; server sections and a fair seating rotation.

#### R3. Order taking (server, handheld and guest self-order)
- **Manual today:** Paper pad; server walks the order to the kitchen.
- **Existing POS:** Handhelds; QR self-ordering and kiosks (Toast, Square, Odoo self-order).
- **Our features:**
  - Handheld ordering at the table, by seat number, with courses.
  - QR ordering at the table linked to the open bill, so guests can add rounds and servers see them.
  - Suggestive selling prompts (pairings, add-ons) driven by margin.

#### R4. Kitchen and bar routing, coursing and KDS
- **Manual today:** Paper tickets on a rail; lost or smudged tickets; starters and mains out of sync.
- **Existing POS:** Printer and KDS routing by station; bump screens; Odoo's preparation display separates kitchen and bar.
- **Our features:**
  - Items routed automatically to the right station (grill, cold, pastry, bar).
  - Course firing ("hold mains", "fire mains"), with timers and colour alerts for late tickets.
  - Expo screen that brings food and drinks for one table together.
  - All-day counts ("12 burgers in progress"); recall and re-fire with reason; ticket-time reports by station.

#### R5. Allergens and special requests
- **Manual today:** Written on the ticket, often missed.
- **Our features:**
  - Allergen data on every recipe (for example, the 14 allergens required in the EU and UK, or the local list), shown on digital menus.
  - Allergy modifiers highlighted on KDS and printed tickets, with a mandatory acknowledgement by the kitchen.

#### R6. Menu management and "86" (sold-out) control
- **Manual today:** Server tells guests an item has run out, after they've ordered it.
- **Existing POS:** Menu management; item availability toggles.
- **Our features:**
  - One menu published to POS, QR menu, kiosk, website and delivery platforms, with channel-specific prices.
  - Sold-out ("86") from the kitchen or automatically from stock levels, synced to every channel.
  - Scheduled menus (breakfast, lunch, dinner, late-night bar menu).

#### R7. Bills, splitting and payment
- **Manual today:** Handwritten bill; splitting worked out on a calculator.
- **Existing POS:** Split by seat, item or amount; pay-at-table on handhelds.
- **Our features:**
  - Split by seat, item, equal shares or custom amount; merge bills across tables.
  - Service charge rules; pay at table on handheld or SoftPOS; guest pays by QR.
  - Room charge to the hotel property management system (PMS) for hotel venues.

#### R8. Delivery and takeaway orders
- **Manual today:** One tablet per delivery platform; orders re-typed into the POS.
- **Existing POS:** Aggregator integrations (often via middleware).
- **Our features:**
  - Delivery-platform orders injected directly into the POS and KDS, with no separate tablets.
  - Menu, price and availability synced to each platform.
  - Own online ordering with pickup times based on kitchen load; commission and net-revenue report per platform.

#### R9. Recipe costing and food cost
- **Manual today:** Food cost estimated in a spreadsheet, if at all.
- **Existing POS:** Basic recipe costing in some; often an add-on.
- **Existing ERP:** BoM costing with standard or average cost.
- **Our features:**
  - Recipe and sub-recipe (sauces, prepped items) costing with yield and trim loss.
  - Theoretical versus actual food cost by category and period.
  - Supplier price change alerts that show which dishes are affected.
  - Menu-engineering matrix (popularity versus margin) with suggested actions.

#### R10. Prep planning and production
- **Manual today:** Chef decides prep quantities from experience.
- **Our features:**
  - Daily prep list from forecast covers and reservations.
  - Sub-recipe production that consumes raw stock and creates prepped stock, with batch dates and use-by labels.

#### R11. Kitchen waste
- **Our features:**
  - Waste logged by item and reason (spoiled, overproduction, returned plate, dropped) on the KDS or tablet.
  - Waste value trends and top waste items.

#### R12. Staff scheduling and labour control
- **Manual today:** Rota on a whiteboard or WhatsApp group.
- **Existing POS:** Time clock and scheduling in major suites (Toast, Simphony, Square).
- **Existing ERP:** HR and payroll modules.
- **Our features:**
  - Schedule built from forecast sales and covers, with a labour-cost % target.
  - Shift swaps and availability in a staff app; clock-in by PIN or face at the terminal; break and overtime rules.
  - Hours and tips exported to payroll, or posted to ERP HR in Mode 2.

#### R13. Guest feedback and loyalty
- **Our features:**
  - QR feedback after payment, linked to the server, table and dishes.
  - Loyalty points shared across restaurant, bar and shop (for mixed businesses); birthday and win-back offers.

#### R14. Daily sales report and shift close
- **Manual today:** The manager writes a daily sales report (DSR) and sends it to the owner.
- **Our features:**
  - Automatic DSR emailed or sent by WhatsApp at close: sales, covers, average spend, labour %, comps, voids, cash over/short and notes.

#### R15. Coffee bar and barista station
This process also applies to bars that serve coffee and to supermarkets with an in-store café.
- **Manual today:** Cashier shouts the order or writes it on the cup; barista works from memory; milk and beans ordered when they run out; loyalty stamp cards in wallets.
- **Existing POS:** Café-focused POS products (Square, Clover, Toast and others) offer nested modifiers (milk, size, temperature, syrups), barista display screens and order-ahead through a shared item library.
- **Gap:** Coffee stock (beans by gram, milk by litre) is rarely tracked against drinks sold; order-ahead and counter orders often arrive in different queues; stamp cards are easy to abuse.
- **Our features:**
  - Barista display screen (or cup-label printer) with one queue for counter, table, QR and order-ahead drinks, sorted by promised time, with the customer's name.
  - Drink builder: size, shots, milk type, temperature, syrups and extras in two or three taps; allergen flags (for example, nut milks).
  - Coffee recipes: dose (g), yield (ml) and milk per size, so beans, milk, cups and lids are depleted automatically; milk waste logging.
  - Order-ahead with pickup times set by barista load; "ready" SMS or screen call-out.
  - Digital stamp card and loyalty shared with the bar, restaurant or supermarket.
  - Station checklist: opening calibration, cleaning and closing tasks, with sign-off by the barista.

### 6.4 Shared back-office and ERP processes

> From v0.4, these back-office processes are specified in full by the **Finance core (Section 20)**. E1–E9 remain valid as summaries and map to F-IDs: E1 → F2/F3; E2 → F11; E3 → F10; E4 → F8; E5 → F13; E6 → F17/F18; E7 → F1; E8 → F15/F19; E9 → F1/F17.

These processes are where ERP systems are strongest and POS systems weakest. The platform either performs them natively (Mode 1) or feeds the customer's ERP (Modes 2 and 3).

| # | Process | Manual today | Existing POS | Existing ERP | Our features |
|---|---|---|---|---|---|
| E1 | **Posting sales to accounts** | Accountant keys daily totals from Z reports | Export or integration to QuickBooks/Xero | Native (ERP POS) or via import | Automatic journal by day or shift: sales by category, tax, tenders, tips, discounts, COGS, cash over/short; mapped to the chart of accounts |
| E2 | **Tax and fiscalisation** | Tax computed by hand at month end | Sales tax/VAT settings; fiscal support varies by country | Tax engine and returns | Multi-rate tax engine; pluggable fiscal-device and e-invoicing connectors per country; tax reports ready for filing |
| E3 | **Supplier invoices and payments** | Paper invoices in a folder | Limited | Accounts payable, three-way match, payment runs | Invoice capture by photo/PDF with OCR; three-way match against PO and GRN; approval; posting to ERP payables or export |
| E4 | **Bank, card and mobile-money reconciliation** | Spreadsheet | Processor deposit reports | Bank reconciliation | Automatic matching of settlements (including fees and chargebacks) to shifts and bank deposits |
| E5 | **Payroll and HR** | Paper timesheets | Time clock, tips | HR and payroll | Hours, tips and service charge distribution exported to payroll/HR |
| E6 | **Budgets and management reporting** | Owner's spreadsheet | Sales dashboards | Financial reports, budgets | Budget versus actual by location and department; prime cost; flash P&L daily |
| E7 | **Master data governance** | None | Per-location data | Central master data | Central catalogue with approval workflows and change history; controlled sync with ERP |
| E8 | **Internal control and audit** | Owner's trust | Void/refund reports | Audit trail, segregation of duties | Tamper-evident log of all sensitive actions; segregation of duties (for example, receiver can't approve invoice); exception reports |
| E9 | **Multi-company / franchise** | Separate books | Multi-location | Inter-company accounting | Multiple legal entities and franchisees, royalty calculation, consolidated reporting |

---

## 7. People: management roles, users and usability

A POS succeeds or fails on the people who use it every shift. Many existing systems are designed around the transaction, not the person. They offer one generic "employee" profile, a manager PIN, and a back office that only the owner understands. This platform is designed around **the real roles in each business**: what each person is responsible for, what they need to see, what they may and may not do, and the conditions they work in.

### 7.1 Who runs and uses the system in each business

| Level | Supermarket / grocery | Bar / pub / lounge | Resto-bar / café |
|---|---|---|---|
| **Owner / directors** | Owner, directors, head office (HQ) | Owner, investors | Owner, partners |
| **General management** | Store manager, assistant / duty manager | General manager, assistant manager | General manager, restaurant manager |
| **Department / area management** | Department managers (fresh produce, butchery, bakery, deli, grocery, beverages, non-food); front-end (checkout) supervisor / head cashier; buyer / category manager | Bar manager, head bartender, events / promotions manager | Floor manager / supervisor, bar manager, head chef / executive chef, sous chef, head barista / café lead |
| **Control and back office** | Inventory controller / stock clerk, receiving clerk, loss-prevention officer, accountant, HQ admin / IT | Cellar person / storekeeper, accountant | Storekeeper, purchasing officer, cost controller, accountant, marketing |
| **Frontline staff** | Cashiers, self-checkout attendants, shelf stockers / merchandisers, counter staff (deli, butchery, bakery), online-order pickers, security | Bartenders, mixologists, barbacks, baristas, cocktail waiters, door staff / door cashier, VIP host | Host / hostess, waiters / servers, runners, bartenders, baristas, barbacks, cashiers, line cooks / chefs de partie, expeditor, kitchen porters, delivery / takeaway coordinator |
| **Outside users** | Customers (self-checkout, scan-and-go, online orders), suppliers (portal), auditors | Guests (QR tabs, ticketing) | Guests (QR ordering, reservations, order-ahead), delivery riders |

Each person has **one login across all modes and locations**, with a role per location. For example, one person can be a bartender in the lounge and a supervisor in the café. The platform ships with **role templates** for every role above; owners can copy and adjust them.

### 7.2 Role-by-role design: supermarket

| Role | Main responsibilities | What they do in the system | Device | Home screen on login |
|---|---|---|---|---|
| **Owner / HQ** | Profit, growth, control across stores | Approve large POs, price strategy and budgets; review multi-store performance; set policies and permissions | Mobile app, web | Sales vs budget by store, margin, shrink, cash position, alerts |
| **Store manager** | Whole-store results, staff, compliance | Approve voids/refunds/overrides remotely; approve stock adjustments and POs within limit; review daily report; manage rota; handle escalations | Mobile app, back-office tablet or PC | Today's sales vs target, queue length by lane, staff on shift, pending approvals, alerts |
| **Duty / assistant manager** | Runs the shift | Open/close store; float and safe; approvals on the floor; incident log | Mobile app, handheld | Shift checklist, approvals, lane status |
| **Front-end supervisor / head cashier** | Checkouts, cashiers and cash | Assign cashiers to lanes; issue floats; cash drops; blind-close review; open more lanes when queues build; self-checkout interventions | Handheld, supervisor terminal | Lane map (open/closed, queue, cashier), cash in drawers, exceptions |
| **Cashier** | Fast, accurate checkout | Scan, weigh, take payment, apply coupons, loyalty lookup, suspend/recall, age checks | Lane terminal + scanner-scale | Checkout screen only; personal speed and accuracy shown at shift end |
| **Self-checkout attendant** | Help customers, prevent loss | Clear weight/age/security alerts; approve restricted items; assist with produce | Handheld or attendant tablet | All self-checkout lanes with alert status |
| **Department manager** (fresh, butchery, bakery, deli, grocery) | Department sales, margin, waste, availability | Review and adjust suggested orders; production plans; markdowns; waste; cycle counts; planogram tasks | Handheld, tablet | Department sales, margin, waste, expiring items, out-of-stocks, tasks |
| **Counter staff** (deli, butchery, bakery) | Serve and label fresh goods | Weigh and label; record production batches and waste | Label scale, tablet | Production plan, today's orders |
| **Buyer / category manager** | Range, suppliers, cost price and promotions | New items; supplier terms; promotions; approve cost changes; review promotion results | Web back office | Category performance, supplier fill rates, cost changes waiting, promotion calendar |
| **Receiving clerk** | Accurate deliveries | Receive against PO on handheld; record shorts/damages with photos; capture expiry | Rugged handheld | Deliveries expected today, open POs |
| **Inventory controller / stock clerk** | Accurate stock | Cycle counts; transfers; adjustments (for approval); variance investigation | Handheld, PC | Count schedule, variances to review |
| **Shelf stocker / merchandiser** | Full, correct shelves | Gap scans (empty shelf → replenish task); price-label checks; expiry checks | Handheld | Task list by aisle |
| **Online-order picker** | Pick online orders | Picking route; substitutions; weighed-item adjustments | Handheld | Orders to pick, by slot |
| **Loss-prevention officer** | Reduce theft and fraud | Review exception reports and AI alerts; link to CCTV; high-risk item watchlist | Web | Exception dashboard by cashier and lane |
| **Accountant** | Books, tax and reconciliation | Review postings to ERP; settlement reconciliation; tax reports | Web | Posting status, unreconciled items, period close checklist |

### 7.3 Role-by-role design: bar

| Role | Main responsibilities | What they do in the system | Device | Home screen on login |
|---|---|---|---|---|
| **Owner** | Profit and control | Review sales, pour cost, variance, labour, cash; approve large purchases; set permissions | Mobile app | Tonight's sales live, pour cost, variance trend, comps, cash, alerts |
| **General manager** | Whole venue, staff, licence | Approvals; rota; events; incident log; weekly variance review; closing sign-off | Mobile app, tablet | Live sales by station, staff on shift, open tabs, approvals, capacity |
| **Bar manager** | Drinks menu, stock, bar team | Recipes and pour standards; par levels; distributor orders; stock counts and variance investigation; comp budgets; happy-hour rules | Tablet, bottle scale, web | Variance by product and station, stock to order, recipe margins |
| **Head bartender / shift lead** | Runs the bar during the shift | Assign stations; approve voids/comps within limit; manage tabs over limit; bartender cash-outs | Bar terminal with elevated PIN, handheld | Station view, open tabs, tab alerts, voids and comps this shift |
| **Bartender / mixologist** | Fast, accurate drinks and payments | Speed screen, tabs, repeat rounds, payments, tips; comp and spill logging with reason; view recipe cards | Bar terminal, BDS, NFC wristband or card to log in | Speed screen; own open tabs; own sales and tips |
| **Barback** | Keep the bar stocked | Restock from par list; record store-room transfers; log empty kegs and bottles; report breakages | Handheld | Restock list, keg status |
| **Barista** (in bars with coffee service) | Coffee drinks | Barista queue, recipes, milk waste (see R15) | Barista display | Drink queue |
| **Cocktail waiter / server** | Table service in the bar | Orders and tabs by table on a handheld; pay at table | Handheld / SoftPOS | Own tables and tabs |
| **Door staff / door cashier** | Entry, cover charge, capacity, ID | Sell covers and tickets; scan guest lists; ID checks; capacity count; incident log | Handheld, ID scanner | Capacity gauge, guest list, cover sales |
| **VIP host** | Tables and bottle service | Bookings, deposits, minimum spend tracking | Handheld | VIP tables with spend vs minimum |
| **Cellar person / storekeeper** | Store room | Receive distributor deliveries; issue stock to bars; keg and crate returns | Handheld | Requisitions to fill, deliveries due |
| **Events / promotions manager** | Events and promoters | Event pricing, guest lists, promoter commissions and results | Web, mobile | Event performance, promoter reports |
| **Accountant** | Books | Postings, cash and settlement reconciliation, tip and tax reports | Web | Posting status, reconciliation |

### 7.4 Role-by-role design: resto-bar and café

All bar roles in 7.3 also apply to the bar in a resto-bar.

| Role | Main responsibilities | What they do in the system | Device | Home screen on login |
|---|---|---|---|---|
| **General / restaurant manager** | Guest experience, sales, labour, cost | Approvals; rota vs forecast; daily sales report; food and beverage cost review; complaints | Mobile app, tablet | Covers, sales vs forecast, labour %, ticket times, approvals, guest feedback |
| **Floor manager / supervisor / captain** | Runs the floor during service | Table assignments and sections; comps and voids within limit; bill problems; pace of service | Handheld | Live floor plan with table timers and alerts |
| **Host / hostess** | Reservations, waitlist, seating | Bookings, waitlist messages, seating, table combining | Host tablet | Floor plan + reservations timeline + waitlist |
| **Waiter / server** | Take orders and serve | Order by seat and course on handheld; fire courses; split bills; pay at table; tips | Handheld / SoftPOS | Own tables with status and timers |
| **Runner** | Deliver food and drinks | See ready items and table numbers | Expo screen, handheld | Items ready to run |
| **Head chef / executive chef** | Menu, kitchen team, food cost | Recipes and costing; menu engineering; prep plans; supplier orders; waste review; kitchen rota | Tablet, web | Food cost %, top waste items, prep plan, ticket times by station |
| **Sous chef / kitchen lead** | Runs the kitchen during service | Fire and pace orders; mark items sold out ("86"); approve re-fires | KDS (expo) | Expo view, all-day counts |
| **Line cook / chef de partie** | Cook their station's items | Bump tickets on station KDS; record waste | Station KDS with bump bar | Station tickets only |
| **Expeditor** | Complete, correct plates and trays | Bring food and drinks for one table together; call runners | Expo KDS | Expo view by table |
| **Kitchen porter / steward** | Cleaning, dishes, receiving support | Checklists; waste logging | Wall tablet | Checklists |
| **Head barista / café lead** | Coffee quality and the café team | Coffee recipes, milk and bean orders, calibration and cleaning logs, café rota | Tablet | Drinks per hour, queue time, milk waste, stock to order |
| **Barista** | Coffee and café drinks | Barista queue; drink builder; mark ready; milk waste; station checklist | Barista display, counter terminal | Drink queue by promised time |
| **Cashier / counter staff** | Counter orders and payments | Order, payment, loyalty, takeaway | Counter terminal | Order screen |
| **Storekeeper / purchasing officer** | Stock and suppliers | Receiving, issues to kitchen and bar, POs, supplier invoices | Handheld, web | Deliveries due, stock below par |
| **Cost controller** | Food and beverage cost | Theoretical vs actual cost; variance; recipe cost updates; month-end stock valuation | Web | Cost dashboards, variances |
| **Delivery / takeaway coordinator** | Aggregator and takeaway orders | Accept and pace orders; rider handover; availability per platform | Tablet | Incoming orders by platform, prep times |
| **Marketing** | Guests and loyalty | Campaigns, loyalty rules, feedback follow-up | Web | Guest counts, loyalty, reviews |

### 7.5 Permissions and approval matrix

Permissions are granted by role, can be adjusted per person and location, and are logged. The approach reflects practice in current systems, such as the configurable role permissions in Lightspeed Restaurant, TouchBistro and LS Central. Two improvements over typical POS behaviour:

1. **Thresholds, not just on/off.** A bartender may comp up to a set value per shift, and a supervisor up to a higher value.
2. **Remote approval.** The request goes to the manager's phone with the details (item, amount, staff member, reason), so no one has to key a manager PIN into a staff terminal. Manager PINs typed on shared screens are a common source of fraud.

The table below shows default templates for a typical venue. ✔ = allowed; ◐ = allowed up to a threshold; ✖ = needs approval from a higher level.

| Action | Frontline (cashier, bartender, server, barista) | Supervisor / head bartender / head cashier | Manager (store, bar, restaurant, GM) | Owner / HQ |
|---|:---:|:---:|:---:|:---:|
| Void an item before it is sent to kitchen/bar or paid | ✔ | ✔ | ✔ | ✔ |
| Void after sending / after payment | ✖ | ◐ | ✔ | ✔ |
| Discount | ◐ preset discounts only | ◐ up to set % | ✔ | ✔ |
| Price override | ✖ | ◐ | ✔ | ✔ |
| Refund | ✖ | ◐ with receipt, up to set amount | ✔ | ✔ |
| Comp / spill / staff drink | ◐ spills logged; comps within budget | ◐ within budget | ✔ | ✔ |
| Open cash drawer without a sale | ✖ | ✔ (reason required) | ✔ | ✔ |
| Tab over limit, transfer another person's tab | ✖ | ✔ | ✔ | ✔ |
| Close shift / blind close | ✔ own drawer | ✔ review | ✔ approve variances | ✔ |
| Stock adjustment | ✖ (can count) | ◐ | ◐ up to set value | ✔ |
| Receive goods | ✔ (receiving role) | ✔ | ✔ | ✔ |
| Create PO | ✖ | ◐ suggested orders only | ◐ up to set value | ✔ |
| Change prices, recipes, promotions | ✖ | ✖ | ◐ local items | ✔ central |
| Edit time clock entries | ✖ | ◐ same day | ✔ | ✔ |
| View sales reports | Own sales and tips | Shift | Location | All |
| View costs and margins | ✖ | ✖ | ✔ | ✔ |
| Manage users and permissions | ✖ | ✖ | ◐ local staff | ✔ |

Controls that apply to everyone:
- Segregation of duties: the person who receives goods cannot approve the supplier invoice; the person who counts cash cannot approve their own variance.
- Every sensitive action records who did it, who approved it, when, on which device, and why.
- Owners get a weekly "exceptions by person" summary.

### 7.6 Management routines built into the system

Managers work in routines. The platform turns each routine into a guided checklist with the data they need already on screen, instead of leaving them to find reports.

| When | Supermarket (store / department manager) | Bar (bar manager / GM) | Resto-bar / café (GM, head chef, café lead) |
|---|---|---|---|
| **Before opening** | Opening checklist; floats issued; lanes assigned from forecast; expiring-items task list; deliveries due | Par restock list; floats; station assignment; happy-hour and event prices confirmed | Reservations and forecast covers; prep list; 86 list; staff briefing notes; café calibration check |
| **During the shift** | Queue alerts (open another lane); remote approvals; out-of-stock alerts; self-checkout alerts | Live sales by station; tab alerts; capacity; comp and void alerts; remote approvals | Ticket-time alerts; table timers; sold-out sync; labour vs sales alert (send staff home or call in) |
| **Closing** | Blind closes and cash-up; safe; waste and markdown recording; daily sales report | Close or transfer all tabs; bartender cash-outs; high-value stock count; incident log | Server cash-outs and tips; waste log; daily sales report; closing checklists |
| **Weekly** | Suggested orders review; cycle counts; promotion results; rota | Full liquor count and variance review; distributor orders; rota | Food cost and variance; menu-engineering review; supplier orders; rota |
| **Monthly** | Stock valuation; shrink review; supplier scorecards; P&L | Pour cost and margin; comp budgets; P&L | Food and beverage cost; menu changes; P&L; guest feedback trends |

Other management features:
- **Shift handover notes:** structured notes passed from one manager to the next (for example, "keg 3 running low", "customer complaint pending").
- **Task assignment:** managers assign tasks (count this shelf, clean the coffee machine, reset planogram) with due times and photo proof.
- **Staff communication:** announcements and pre-shift briefing (specials, sold-out items, events) shown on staff screens and in the staff app.
- **Performance coaching:** per-person reports (speed, accuracy, upselling, voids) are shown to the person and their manager to support fair coaching, not just discipline.

### 7.7 Usability by working environment

Each workplace has different physical conditions, so screens and devices are designed for where they are used.

| Environment | Conditions | Design response |
|---|---|---|
| **Supermarket checkout lane** | Standing for hours; high item volume; repetitive motion; bright lighting and glare | Scanner and keyboard first; minimal screen touches per item; large totals; anti-glare screens; ergonomic layout (scanner-scale placement, screen height); items-per-minute feedback |
| **Self-checkout** | Untrained customers; stress; many languages; accessibility needs | Step-by-step guidance with pictures; language choice; audio and visual cues; wheelchair-reachable height; big buttons; fast help call to attendant |
| **Shop floor, receiving dock, store room** | Walking; cold rooms; cartons; poor Wi-Fi in back areas | Rugged handhelds with built-in scanners; one-handed use; glove-friendly buttons; offline tasks that sync later |
| **Bar counter** | Dark; loud; wet; very busy; many staff on one terminal | Dark theme with high contrast; large buttons; no reliance on sound; spill-proof screens; login by NFC tap in under a second; speed screen; most actions in one or two taps |
| **Coffee station** | Steam, heat and noise; both hands busy; fast queue | Barista display readable from a distance; bump by large button or foot pedal; cup-label printer as an alternative to the screen; names on orders |
| **Restaurant floor** | Moving between tables; sunlight on terraces; guests watching | Handheld with sunlight-readable screen; one-handed ordering; seat and course numbering; fast payment at the table; discreet design |
| **Kitchen** | Heat, grease and steam; gloves; noise; no time to touch screens | KDS mounted at eye level, readable from 2 m; colour-coded timers; bump bars instead of touch; allergy alerts in a distinct colour and icon |
| **Door / outside** | Night; rain; queues | Bright, weather-resistant handheld; large capacity counter; fast ID scan |
| **Back office / head office** | Desk; long sessions; bulk data | Full web app with keyboard shortcuts, bulk editing, imports/exports and saved report views |
| **Owner on the move** | Phone; short attention; low bandwidth | Mobile app with a one-screen summary, push alerts, one-tap approvals, and data-light mode |

### 7.8 Usability principles and accessibility

1. **Role-based home screens:** everyone sees only what their job needs. A cashier never sees cost prices; a line cook sees only their station.
2. **Speed targets for common tasks:** add an item in 1 tap or scan; open a tab in 1 card tap; repeat a round in 1 tap; take payment in 2 taps; find any item in 3 keystrokes. These are measured in usability tests.
3. **Consistent layout** across modes, so staff who move between bar, café and shop don't have to relearn.
4. **Pictures and icons with words,** for staff with lower literacy or who work in a second language; product photos on buttons.
5. **Language per user,** so two bartenders on the same terminal can each see their own language. Receipts can use the customer's language where needed.
6. **Error prevention over error messages:** unavailable items are greyed out; large or unusual quantities are flagged ("24 bottles of vodka?"); confirmation only for actions that are hard to undo.
7. **Visible system status:** clear offline, sync and printer status indicators, and what to do next when something fails.
8. **Customisable, but controlled:** managers can rearrange buttons and speed screens for their venue; HQ can lock layouts where consistency matters.
9. **Accessibility:** customer-facing screens (self-checkout, kiosks, QR menus) target WCAG 2.2 AA. This means adjustable text size, colour-blind-safe status colours, screen-reader support on web and mobile, and reachable heights for kiosks. Staff screens support left- or right-handed layouts.
10. **Training mode on every device:** practice transactions that don't affect sales or stock, plus in-app tips on first use of each feature.

### 7.9 Staff experience and adoption

- **Staff app:** schedule, shift swaps, availability, clock-in reminders, tips and earnings, announcements, and training modules.
- **Onboarding:** a new cashier, bartender, server or barista should be productive within one hour. Built-in guided lessons are tracked in the staff profile.
- **Recognition:** optional sales contests and upselling goals, used carefully and transparently.
- **Fairness and privacy:** staff can see the performance data held about them. Monitoring (such as CCTV overlays and exception reports) follows local labour and privacy law.
- **Measuring usability:** during the pilot, measure task times, error rates, training time and a standard usability questionnaire such as the System Usability Scale (SUS) for each role. The target is a SUS score of 75 or more, which is generally considered good. Test with real staff in real conditions (for example, a Friday-night bar shift) before rollout.

---

## 8. Solution architecture and core modules

### 8.1 Architecture overview

```
      ┌──────────────── EXISTING SYSTEMS (Modes 2 & 3) ─────────────────┐
      │  ERP (Odoo, ERPNext, SAP B1, Business Central, Sage, QuickBooks…) │
      │  Existing POS (Toast, Square, Lightspeed, legacy grocery POS…)    │
      └───────────────▲──────────────────────────────▲───────────────────┘
                      │ ERP connectors               │ POS connectors
                      │ (API · webhooks · SFTP · EDI)│ (API · exports)
      ┌───────────────┴──────────────────────────────┴───────────────────┐
      │                        CLOUD PLATFORM                              │
      │  Integration hub & mapping · Catalogue · Pricing/Promotions        │
      │  Inventory & recipes · Purchasing · Customers & loyalty · Staff    │
      │  Finance posting · Reporting · AI engine · Multi-location HQ       │
      └───────────────▲──────────────────────────────▲───────────────────┘
                      │ sync (HTTPS)                 │
      ┌───────────────┴─────────────┐        ┌───────┴──────────────────┐
      │   STORE / VENUE EDGE        │        │  OWNER & MANAGER APPS     │
      │  Local server or lead       │        │  Web back office · Mobile │
      │  terminal: offline cache,   │        │  approvals · Alerts       │
      │  queue, printer/KDS hub     │        └──────────────────────────┘
      └──┬────────┬────────┬────────┘
         │        │        │
  ┌──────┴──┐ ┌───┴────┐ ┌─┴──────────┐  ┌────────────┐  ┌──────────────┐
  │Checkout │ │Bar /   │ │Handhelds / │  │ KDS / BDS  │  │Self-checkout │
  │lanes +  │ │counter │ │SoftPOS     │  │ screens    │  │kiosks / QR   │
  │scales   │ │terminal│ │phones      │  │            │  │ordering      │
  └─────────┘ └────────┘ └────────────┘  └────────────┘  └──────────────┘
                   │
            Payment terminals (P2PE) ──► Payment processor / acquirer
```

### 8.2 Design principles

- **Cloud-native, offline-first.** Every terminal holds a local copy of the catalogue, prices, promotions and open tabs. Sales continue without internet, and offline card payments are stored and forwarded within configurable risk limits.
- **Offline conflict rules.** Sales recorded offline are never discarded. Price or promotion changes made centrally apply from the time they reach the terminal. Stock is corrected by adjustment movements, never overwritten. Tabs are locked to one terminal at a time, or merged with an audit record.
- **Modular.** A shared core plus vertical modes (Supermarket, Bar, Resto-bar) switched on per location.
- **Hardware-agnostic where possible.** Runs on Android and iPadOS tablets, Windows lane PCs, and certified payment terminals, to avoid lock-in.
- **API-first.** Documented REST/webhook APIs; the integration hub (Section 5.4) is part of the product, not a project add-on.
- **Secure by design.** Card data never touches the POS application (P2PE / tokenisation), role-based access control, full audit trail, encryption in transit and at rest.

### 8.3 Core modules (shared by all modes)

**Finance, Inventory and Sales are the three core pillars;** Sections 19–23 define them in detail and they take precedence over the summaries below. All other modules use the core service contracts (19.3).

| Module | Key capabilities |
|---|---|
| **Sales & checkout** | Barcode/PLU/search/quick keys; modifiers and variants; discounts with approval rules; returns and exchanges; split tender (cash, card, mobile money, voucher, account); receipts printed, emailed, SMS/WhatsApp or QR |
| **Payments** | EMV chip & PIN, contactless, wallets, QR/mobile money, SoftPOS, pre-authorisation, tips, offline store-and-forward, settlement reconciliation, cash drawer management with blind close |
| **Inventory** | Multi-location stock; multi-UoM; recipes and sub-recipes; batch/expiry; purchase orders; receiving; transfers; cycle counts; waste; par levels; returnable containers |
| **Purchasing & suppliers** | Suggested orders, approvals, supplier catalogues and price history, invoice capture and three-way match, supplier claims and scorecards |
| **Customers & loyalty** | Profiles, points/tiers, stored value and gift cards, credit accounts, coupons, consent-based marketing |
| **Staff** | PIN / NFC / biometric login; roles and permissions; scheduling; time clock; tips; per-employee performance and exception reports |
| **Finance posting** | Journal generation, tax engine, fiscal connectors, ERP/accounting sync, settlement reconciliation |
| **Reporting & analytics** | Real-time dashboards; standard and custom reports; scheduled reports; multi-location comparisons; data export |
| **Loss prevention** | Void, discount, no-sale, refund and comp exception reports; CCTV transaction overlay; AI anomaly alerts |
| **Integration hub** | Connectors, mapping, sync monitoring, error queue and resend |
| **Admin** | Multi-entity, locations, devices, users, audit log, backups |

### 8.4 AI and advanced features (phase 2+)

| Feature | Value | Status in today's market |
|---|---|---|
| Demand forecasting and suggested orders | Fewer stock-outs, less over-ordering | Shipping in several grocery, hospitality and ERP products |
| Expiry-driven dynamic markdowns (with ESL) | Less perishable waste | Early adoption in grocery |
| Produce recognition at checkout (computer vision) | Faster, more accurate self-checkout | Deployed by enterprise grocery vendors (for example, NCR Voyix Picklist Assist) |
| Anomaly and fraud detection (voids, refunds, comps, variance) | Less internal theft | Enterprise products (for example, NCR Voyix Insight); rare in small-business POS |
| Labour scheduling from forecast | Lower labour cost | Available in major hospitality suites |
| Natural-language questions ("What were last Friday's top cocktails?") | Easier insight for owners | Emerging |
| Voice or chat ordering (phone, WhatsApp) | Captures orders without staff time | Emerging |
| Menu engineering and price optimisation | Higher margins | Available via analytics add-ons |

---

## 9. Hardware concept

| Device | Supermarket | Bar | Resto-bar |
|---|:---:|:---:|:---:|
| Fixed POS terminal (touchscreen, all-in-one) | ✔ each lane | ✔ each station | ✔ host/bar |
| Barcode scanner (2D, handheld or in-counter) | ✔ | optional | optional |
| Scanner-scale (checkout) and label scale (deli) | ✔ | – | – |
| Customer-facing display | ✔ | ✔ | ✔ |
| Receipt printer (thermal) and cash drawer | ✔ | ✔ | ✔ |
| Payment terminal (EMV, NFC, P2PE-validated) | ✔ | ✔ | ✔ |
| Handheld POS / SoftPOS phone | receiving, counts, queue-busting | ✔ floor service, door | ✔ table service |
| Kitchen/bar printers or KDS/BDS screens | deli/bakery | ✔ BDS | ✔ KDS + BDS |
| Barista display or cup-label printer | in-store café | optional | ✔ café |
| Manager smartphone (approvals, alerts) | ✔ | ✔ | ✔ |
| NFC staff cards / wristbands for fast login | optional | ✔ | ✔ |
| Self-checkout / kiosk | ✔ | – | optional |
| Electronic shelf labels | ✔ | – | – |
| ID scanner | alcohol/tobacco | ✔ | ✔ |
| Bluetooth bottle scale / smart spouts / keg meters | – | optional | optional |
| Label printer (reduced-price, prep, shelf) | ✔ | – | ✔ prep labels |
| Local edge server / UPS | ✔ larger stores | optional | optional |

**Principles:** use commercially available, supported devices; spill- and heat-resistant screens for bars and kitchens; rugged handhelds; keep a spare-device pool; use a UPS for lanes and network gear.

---

## 10. Payments, security and compliance

**Payments**

- Card-present acceptance (EMV chip, contactless), wallets, and locally dominant methods such as mobile money or QR schemes, depending on the market.
- Pre-authorisation and incremental authorisation for bar tabs; tip adjustment.
- SoftPOS (tap to phone) for queue-busting and table-side payment, using solutions certified under the PCI MPoC or CPoC standards.
- Offline store-and-forward with per-transaction and total limits.
- Processor-agnostic, or at least transparent processing rates. Card processing is typically the largest recurring cost for a venue, commonly around 2.3–3.0% plus a fixed per-transaction fee in the US market.

**Security (PCI DSS v4.0 aligned)**

- P2PE-validated or E2EE terminals so card data never enters the merchant network, plus tokenisation for stored cards.
- Multi-factor authentication for back-office and admin access; role-based permissions; unique staff logins, with no shared PINs.
- Encryption in transit (TLS 1.2+) and at rest; secure device management; automatic updates.
- Tamper-evident audit logs (voids, refunds, price overrides, drawer opens, master-data changes).
- Regular penetration testing and vulnerability scanning; integration credentials stored in a secrets vault.

**Regulatory**

- Tax: VAT/GST/sales tax including multiple rates, exemptions and tax-inclusive pricing.
- Fiscalisation / e-invoicing: many countries require POS systems to transmit invoices to the tax authority in real time or to use certified fiscal devices. The platform provides a pluggable fiscal-connector layer per country.
- Data protection: GDPR-style consent and data minimisation for customer data; clear retention policies.
- Alcohol licensing: licensed hours, age verification, capacity and responsible-service records.
- Weights and measures: certified scales for sale by weight; unit pricing on shelf labels where required.
- Food safety: allergen information and use-by labelling.

---

## 11. Integrations and ERP connectors

| Category | Planned connectors (priority to be set in discovery) |
|---|---|
| **ERP** | Odoo, ERPNext, SAP Business One, SAP S/4HANA (via API/middleware), Microsoft Dynamics 365 Business Central and Finance & Operations, Sage, Tally; generic API/CSV |
| **Accounting** | QuickBooks, Xero, Zoho Books, Sage |
| **Existing POS (Mode 3 data import)** | Toast, Square, Lightspeed, Clover, Odoo POS, and legacy POS via database or file export |
| **E-commerce & delivery** | Own web store, Shopify, WooCommerce; global and local delivery aggregators |
| **Payments** | Multiple processors / acquirers, mobile-money providers, gift-card providers |
| **Supplier & inventory** | Supplier EDI/portals, ESL vendors, label and checkout scales, smart spouts and keg meters |
| **Hospitality** | Reservation platforms, hotel PMS (room charge), ticketing |
| **HR & payroll** | Payroll providers and HR systems |
| **Marketing & CRM** | Email/SMS/WhatsApp platforms, review sites |
| **Security** | CCTV/video-analytics overlays, access control |
| **BI** | Data warehouse connector, Power BI / Looker / Excel feeds |

---

## 12. Key reports and KPIs

**Supermarket:** sales per hour and per lane; average basket value and size; items per minute per cashier; gross margin by category; shrink % (known and unknown); waste and markdown value; stock turn; out-of-stock rate; supplier fill rate; promotion uplift and margin; self-checkout usage and intervention rate.

**Bar:** pour cost % (overall and by category); liquor variance % and value by product, station and bartender; sales per bartender per hour; average check; walk-out losses; comp/spill value; happy-hour lift; keg yield; top sellers and dead stock.

**Café / barista station:** drinks per hour; average wait from order to "ready"; order-ahead share; milk and bean usage versus theoretical; loyalty redemption.

**People and management:** sales and items per labour hour by role; approval response time; exceptions by person; training completion; staff turnover; usability (SUS) score by role.

**Resto-bar:** covers, table turn time, average spend per cover; ticket times by station; food cost % and beverage cost %; prime cost (COGS + labour); sales by channel (dine-in, takeaway, delivery, QR); delivery commission; menu-engineering matrix; labour % of sales; guest feedback score.

**All:** daily sales versus forecast and budget; payment mix and processing fees; voids, refunds and discounts by staff; cash over/short; sync health (ERP postings succeeded or failed); customer retention and loyalty redemption.

---

## 13. Implementation approach

| Phase | Duration (indicative) | Key activities | Output |
|---|---|---|---|
| **0. Discovery** | 3–4 weeks | Site visits; **process walk-throughs against Section 6** for each service; existing POS/ERP inventory and data audit; legal/tax/fiscal requirements; choose the deployment mode | Requirements, fit-gap and integration design |
| **1. Design & build / configure** | 8–12 weeks | Core platform and selected modes; ERP/POS connectors and mappings; payments, scales, ESL; data migration (items, recipes, customers, stock) | Configured system in test |
| **2. Pilot** | 6–8 weeks | One supermarket, one bar and one resto-bar (or those available); training; parallel run with the existing system; KPI comparison against baseline; ERP posting validation | Pilot report, go/no-go |
| **3. Rollout** | Phased per site | Hardware install, train-the-trainer, on-site go-live support for the first week | Live sites |
| **4. Optimise** | Ongoing | AI features, advanced analytics, loyalty, more connectors; quarterly reviews | Continuous improvement |

For customers starting in **Mode 3**, Phase 1 is shorter (connectors and dashboards only), and the pilot focuses on variance, forecasting and loss-prevention insight before any tills are replaced.

**Change management and training**

- Role-based training using the role templates in Section 7: cashier, self-checkout attendant, department manager, receiving clerk, buyer, bartender, barback, barista, server, host, chef and line cook, storekeeper, supervisor, manager, accountant and owner.
- Managers trained on approvals, daily routines (Section 7.6) and reading exception reports.
- Short in-app guides and videos; a sandbox "training mode" on every terminal.
- Super-users at each site; two weeks of intensive support after go-live.

---

## 14. Commercial model (options)

| Model | Description | Suited to |
|---|---|---|
| **SaaS subscription** | Monthly fee per location and/or per terminal, tiered by features (Basic / Pro / Enterprise) | Most operators |
| **Intelligence layer (Mode 3)** | Lower-priced subscription for analytics, variance and forecasting over existing systems | Groups with existing POS contracts |
| **Payments-bundled** | Low or zero software fee, with revenue from card processing | Small bars and cafés |
| **Hardware** | Purchase upfront, lease, or bundle into subscription | All |
| **Add-ons** | Self-checkout, ESL, advanced inventory, AI forecasting, loyalty, online ordering, extra KDS, extra ERP connectors | Scale-up |
| **Services** | Implementation, data migration, integration, training, premium 24/7 support | Larger chains |

**Market reference (indicative, US, 2026; verify before use):** mainstream hospitality POS software ranges from $0 to about $400 per month per location depending on tier, with card-present processing typically around 2.3–3.0% + $0.10–0.15 per transaction. Industry comparisons suggest a single full-service location should budget roughly $300–$1,200 per month all-in, with processing fees the largest line item. Enterprise and ERP-based systems (Simphony, NCR Voyix, LS Central, SAP) are usually quoted per project through partners.

---

## 15. Indicative cost components (for budgeting)

| Item | Notes |
|---|---|
| Software licences / subscription | Per location and per terminal; add-ons priced separately |
| Hardware | Terminals, printers, drawers, scanners, scales, payment terminals, handhelds, KDS, kiosks, ESL, networking, UPS |
| Payment processing | Percentage + fixed fee per transaction; negotiate interchange-plus pricing at volume |
| Implementation | Discovery, configuration, integrations, data migration |
| ERP/POS connectors | Standard connectors included; custom mappings or unusual systems quoted separately |
| Training & change management | Initial and refresher |
| Support & maintenance | SLA tiers; spare-device pool |
| Connectivity | Primary broadband + 4G/5G failover |
| Compliance | PCI assessments, fiscal devices or certification where required |

*(Specific figures should be quoted once the number of sites, lanes, terminals, existing systems and the country of operation are confirmed.)*

---

## 16. Risks and mitigation

| Risk | Impact | Mitigation |
|---|---|---|
| Internet outage | Lost sales | Offline-first terminals, local edge server, 4G/5G failover, store-and-forward payments |
| Poor data quality (SKUs, recipes) | Wrong stock and variance figures | Data cleansing in discovery; GTIN standards; recipe audit with the bar manager and chef |
| Integration failures with the customer's ERP/POS | Missing or duplicate postings | Idempotent messages, sync monitoring, error queue, reconciliation reports, pilot validation of postings |
| Existing POS vendor limits API access (Mode 3) | Incomplete data | Use official APIs/exports first; negotiate access; fall back to Mode 2 for affected sites |
| Staff resistance | Slow adoption, workarounds | Role-based screens, training mode, super-users, involving staff of every role in the pilot |
| Poor usability in real conditions (dark bar, hot kitchen, peak rush) | Errors, slow service, staff bypass the system | Design per environment (Section 7.7); usability tests on live shifts; SUS measured by role |
| Permissions too loose or too strict | Fraud, or managers overloaded with approvals | Threshold-based permissions, remote approvals, and monthly review of approval volumes and exceptions |
| Staff privacy concerns about monitoring | Distrust, legal risk | Transparency to staff about their data; follow local labour and privacy law |
| Scope creep (trying to match every ERP feature) | Delays and cost | Process-based scope from Section 6; ERP remains the system of record where it is strong (Mode 2) |
| Payment security breach | Financial and reputational loss | P2PE/tokenisation, PCI DSS v4.0 controls, MFA, monitoring |
| Vendor/processor lock-in | Higher long-term costs | Open APIs, data export, processor-agnostic design |
| Regulatory change (tax, fiscalisation) | Penalties | Pluggable fiscal connectors; monitoring of local regulation |
| Hardware failure in harsh environments | Downtime at bar or kitchen | Rugged/spill-proof devices, spares, device management |
| Cost overrun | Budget pressure | Phased rollout, pilot-gated investment, clear scope |

---

## 17. Success measures (pilot evaluation)

| Area | Measure | Baseline → Target |
|---|---|---|
| Speed | Avg. checkout time (grocery); order-to-payment time (bar); ticket time (kitchen) | Measure in discovery → −20% |
| Loss | Liquor variance %; grocery shrink %; walk-outs; cash over/short | Measure → variance under 5%, walk-outs ≈ 0 |
| Waste | Perishable and kitchen waste value | Measure → −15% |
| Availability | Out-of-stock incidents | Measure → −25% |
| Integration | ERP postings completed without manual correction | → ≥ 99% |
| Books accuracy | Daily three-way reconciliation (Section 23.3) passing; open finance issues older than 7 days; days to close the month | → 100% passing; 0 issues older than 7 days; close in ≤ 2 days |
| Admin time | Hours per week on manual entry, reconciliation and reports | Measure → −50% |
| Uptime | Sales lost to downtime | → 0 |
| Usability | System Usability Scale (SUS) by role; time to productivity for new staff; task error rate | → SUS ≥ 75; ≤ 1 hour to productivity |
| Management | Approval response time; managers' admin hours per week | Measure → approvals < 2 min; admin time −50% |
| Satisfaction | Customer satisfaction/NPS | → ≥ 8/10 |
| Financial | Gross margin; labour %; processing cost % | Improvement vs baseline |

---

## 18. Conclusion and next steps

The platform combines what cloud POS products do best (speed, simplicity, payments, mobility) with what ERP systems do best (controls, purchasing, accounting, audit). It adds deep supermarket, bar and resto-bar workflows designed from how each process is actually run today. Its three deployment modes let a business adopt it at its own pace: as an intelligence layer over current systems, as a POS front end to an existing ERP, or as a complete platform.

**Proposed next steps**

1. Build the three cores first, in order: **Finance, then Inventory, then Sales** (Section 23.5), before the vertical modes.
2. Confirm the target businesses: number of sites, lanes and terminals, country or countries of operation, and **the POS and ERP systems they use today**.
3. Validate Sections 6 and 7 with operators: walk through each process and each role with a store manager, department manager, head cashier, bar manager, bartender, barista, floor manager and chef, and rank features as *must*, *should* or *later*.
4. Decide the delivery route: build a custom platform, build on an open-source base (for example, Odoo or ERPNext), or extend an existing product. A short build-versus-buy analysis is recommended.
5. Choose the first deployment mode for the pilot and the priority connectors.
6. Shortlist hardware and payment partners; approve pilot budget and timeline.

---

## 19. The three core pillars: Finance, Inventory and Sales

> **Why this section was added (v0.4).** Sections 1–18 describe what the platform does for each type of business. Sections 19–23 define its **core**: Finance, Inventory and Sales. Every supermarket, bar, resto-bar and café feature is built on these three. Section numbers 1–18 and process IDs S, B, R and E are unchanged, so references in a build already under way stay valid. New process IDs are **F** (finance), **IN** (inventory) and **SA** (sales).

### 19.1 The principle: one event, three views, one truth

Every business event touches all three cores at once. A sale, delivery, return, waste entry or stock count is recorded as **one source document** (the document types and flows are defined in Section 24). From that document the platform produces three consistent records:

| Core | What it records from the event | Question it answers |
|---|---|---|
| **Sales** | What was sold, to whom, at what price, how it was paid | "What did we sell and how were we paid?" |
| **Inventory** | What stock moved, where from, and at what cost | "What do we have, where, and what is it worth?" |
| **Finance** | The balanced double-entry postings: revenue, tax, cost of goods sold (COGS), cash, bank, receivables, payables | "Did we make money, what do we own and owe, and are the books right?" |

The three records are created together, in one database transaction or through a guaranteed outbox. None of them can exist without the others, and the platform checks every day that they still agree (Section 23.3).

### 19.2 Why Finance comes first

- **Finance is the reference that everything else must match.** If the ledger design is right from the start (chart of accounts, posting rules, periods, audit), inventory and sales plug into it cleanly. If finance is added last, it inherits every shortcut the other modules took, and the books never fully tie out. This is a common weakness of POS systems that bolt on accounting later.
- **Inventory comes second** because for supermarkets and bars it is the largest asset on the balance sheet and the source of COGS. It is where most financial errors start (unrecorded waste, count differences, wrong costs).
- **Sales comes third** because it is the highest-volume event and depends on both. Each sale needs stock and cost from inventory, and tax, tender and revenue accounts from finance.
- **Everything else is built on the three cores.** Supermarket, bar and resto-bar modes, payments, purchasing, people, reporting and AI never write money or stock records directly. They call the core services, so no module can bypass the controls.

### 19.3 Core architecture

```
          ┌───────────────────────────────────────────────────────────────┐
          │           VERTICAL MODES & SUPPORTING MODULES                  │
          │  Supermarket · Bar · Resto-bar/Café · Payments · Purchasing    │
          │  People & permissions · Management · Reporting · AI · Integr.  │
          └──────────────┬──────────────────┬──────────────────┬──────────┘
                         │ sales documents  │ stock movements  │ postings (only via cores)
          ┌──────────────▼──────┐   ┌───────▼────────────┐   ┌─▼──────────────────────┐
          │    SALES CORE (SA)   │──►│ INVENTORY CORE (IN)│──►│   FINANCE CORE (F)      │
          │ orders, receipts,    │   │ stock ledger,      │   │ posting engine, books,  │
          │ invoices, returns,   │   │ valuation, counts, │   │ ledgers, reconciliation,│
          │ tenders, Z-close     │   │ GRNI, variance     │   │ health checks, close    │
          └──────────┬───────────┘   └─────────┬──────────┘   └───────────▲────────────┘
                     └────────── revenue, tax, tenders, tips, deposits ──┘
                                          ▲
                           Daily three-way reconciliation (Section 23.3)
```

**Core service contracts** (the only way other modules touch money or stock):

| Contract | Owned by | Used by | Guarantees |
|---|---|---|---|
| `postJournal(sourceDocument, lines)` | Finance | Inventory, Sales, Purchasing, Payments, People | Balanced, dated in an open period, linked to its source document, idempotent, audited |
| `recordStockMovement(sourceDocument, movements)` | Inventory | Sales, Purchasing, Production, Counts | Quantity and value recorded; matching journal posted through Finance |
| `completeSale(salesDocument)` | Sales | POS terminals, online, delivery, QR, self-checkout | Stock depleted through Inventory; revenue, tax, tenders and COGS posted through Finance |

---

## 20. Finance core (F1–F19)

The finance core gives clients **full accounting** (the books their accountants already know) and tools to **find and correct financial errors**. It gives owners who aren't accountants a simple view of **income and expenses**.

### 20.1 Accounting foundations

- **Double-entry accounting** with journals that are balanced by construction; accrual basis, with a cash-basis reporting view for small businesses that file on a cash basis.
- **Chart of accounts templates** for each business type (supermarket, bar, resto-bar, café, mixed), adaptable to local standards (for example, IFRS for SMEs or the national chart where required). Accounts are grouped by class: assets, liabilities, equity, income, expenses.
- **Dimensions** on every posting: legal entity, location, department (grocery, fresh, bar, kitchen, café), cost centre and event. So a P&L can be produced for any slice of the business without separate books.
- **Fiscal years and periods**, multi-entity, multi-currency with exchange-rate revaluation.
- **Source-document rule:** every entry links to a source document (receipt, invoice, credit note, bank line, voucher, count sheet, journal voucher with attachment).

### 20.2 The accountants' books, built in

The platform keeps the standard **books of original entry** (day books) and **ledgers** that accountants and bookkeepers use. They are filled automatically from daily operations, and each can be viewed, printed and exported in the familiar layout: date, reference, particulars, debit, credit and running balance, with drill-down to the source document.

**Books of original entry (day books)**

| Book | What it records | Filled automatically from |
|---|---|---|
| **Cash book** | All cash received and paid, per till, safe and cash account; cash and bank columns view available | POS cash sales, cash refunds, paid-ins/paid-outs, cash drops, safe movements, bank deposits |
| **Bank book** | All receipts and payments through each bank account | Card and mobile-money settlements, supplier payments, bank deposits, bank fees, imported bank statement lines |
| **Petty cash book** | Small cash expenses, on the imprest system (float topped up to a fixed amount), with analysis columns by expense type | Petty cash vouchers with photo of the receipt, approved by a manager |
| **Daily sales book (POS sales summary)** | Each business day's sales by department, tax rate and tender | Z-close of every terminal (SA8) |
| **Sales day book** | Credit sales invoices (B2B customers, customer accounts, event invoices) | Credit invoices (SA5) |
| **Sales returns day book** (returns inward) | Credit notes issued to customers | Refunds and returns (SA4) |
| **Purchases day book** | Supplier invoices for goods and services | Supplier invoices after three-way match (S3, E3) |
| **Purchases returns day book** (returns outward) | Debit notes and supplier credits | Returns to vendor, short/damaged delivery claims (S3, S8) |
| **Expense book** | Non-stock expenses: rent, utilities, wages, repairs, marketing, licences, fees | Expense entries and bills (F5), payroll (F13), card fees (SA2) |
| **Other income register** | Income that isn't from sales: supplier rebates, commissions, event hire, rent received, interest | Income entries (F5), vendor-funded promotion claims (S4) |
| **General journal** | Everything else: opening balances, corrections, accruals and prepayments, depreciation, year-end adjustments | Journal vouchers (F15 correction wizard, F16 close) |

**Ledgers and registers**

| Ledger / register | Purpose | Control check |
|---|---|---|
| **General (nominal) ledger** | Every account, with balances by period and dimension | Trial balance always balances |
| **Debtors (sales) ledger** | One account per customer: invoices, receipts, credit notes | Sum of customer balances = debtors control account |
| **Creditors (purchases) ledger** | One account per supplier: invoices, payments, debit notes | Sum of supplier balances = creditors control account |
| **Stock ledger / valuation** | Quantity and value per item and location (from Inventory core) | Stock valuation = inventory accounts in the GL (IN10) |
| **Fixed asset register** | Equipment (fridges, coffee machines, POS hardware, furniture, vehicles), cost and depreciation | Register total = fixed asset accounts |
| **Payroll register / wages book** | Gross pay, deductions, net pay, employer costs per person and period | Payroll liabilities = wage control accounts |
| **Tax register (VAT / GST / sales tax)** | Output tax on sales, input tax on purchases, adjustments | Register = tax control accounts; basis of tax returns |
| **Tips and service charge ledger** | Tips and service charge collected and paid out, per person | Liability account = unpaid tips |
| **Deposits, gift cards and vouchers ledger** | Customer deposits, reservation deposits, container deposits, gift card and voucher balances | Liability accounts = outstanding balances |
| **Capital, loans and drawings** | Owner's capital, loans and repayments, drawings | Balances agree with loan statements |

### 20.3 Income and expense tracking for owners (F5)

Many owners are not accountants. The finance core offers a **simple view** alongside the full accountant view:

- **Money in / money out** by category (sales, other income; stock purchases, wages, rent, utilities, repairs, marketing, fees) for today, this week, this month and this year, in plain language.
- **Expense capture from a phone:** photograph a receipt or bill; the system reads the supplier, date, amount and tax (OCR) and suggests the category. A manager approves, and it posts to the expense book.
- **Recurring expenses and bills** (rent, subscriptions, loan repayments) with due-date reminders.
- **Cash position:** cash in tills, safes and banks; money owed to the business and by it; a simple 30/60/90-day cash forecast.
- **Budget vs actual** per category, with alerts when spending runs ahead of budget.
- Every simple-view number drills down to the accountant view and the source documents, so the two views always agree.

### 20.4 Reconciliations (F8)

| Reconciliation | What is matched | Frequency |
|---|---|---|
| **Bank** | Bank statement lines (feed or file import) against the bank book, with auto-match rules and suggested matches | Daily |
| **Cash** | Till counts, safe counts and bank deposits against the cash book | Every shift / daily |
| **Card and mobile money** | Provider settlements, fees and chargebacks against card/mobile-money clearing accounts (SA2) | Daily |
| **Debtors and creditors control** | Subsidiary ledgers against control accounts; customer and supplier statements against our records | Monthly (continuous check daily) |
| **Inventory** | Stock valuation against inventory accounts; GRNI (goods received not invoiced) listing against the GRNI account | Daily |
| **Tax** | Tax register against tax control accounts and tax return | Per return period |
| **Liabilities** | Tips, gift cards, deposits against their liability accounts | Weekly |
| **Inter-company** | Balances between legal entities of the same group | Monthly |

### 20.5 Finding and correcting financial errors (F15)

This is the main promise to clients: **the system helps them find and fix finance problems, not just record transactions.**

**1. Prevention (errors stopped at entry)**
- Journals must balance; posting to closed periods is blocked; mandatory source documents and reasons.
- Duplicate detection (for example, the same supplier invoice number twice, or the same bank line matched twice).
- Account-class rules: for example, a purchase above the capitalisation threshold is flagged as a possible fixed asset; stock purchases can only post to inventory or GRNI.
- Three-way match for supplier invoices; approval thresholds and segregation of duties (Section 7.5).

**2. Detection: automatic "books health checks"**

A trial balance only catches errors that make debits and credits unequal. Many errors don't do that, so the platform runs daily checks aimed at each type of error:

| Error type | What it means | How the platform detects it |
|---|---|---|
| **Omission** | A transaction was never recorded | Goods received with no supplier invoice after X days; a trading day with no Z-close posting; a bank line with no match; a recurring bill not entered; a card settlement with no matching sales |
| **Commission** | Right type of account, wrong one (for example, wrong customer or supplier) | Supplier and customer statement reconciliation; payment matched to an invoice of another party; unusual balance direction (a debit balance on a supplier) |
| **Principle** | Wrong class of account (for example, an asset expensed or an expense capitalised) | Account-class rules; amount thresholds; items and suppliers mapped to expected accounts; review list of unusual account choices |
| **Original entry** | Wrong amount entered at source | Three-way match differences; invoice vs PO price; OCR amount vs keyed amount; tax that doesn't match the rate |
| **Reversal** | Debit and credit swapped | Balance direction checks per account type; comparison with the same entry pattern in history |
| **Compensating** | Two errors that cancel each other | Reconciliation at detail level (sub-ledgers, bank, stock) rather than only at totals |
| **Transposition** | Digits swapped (for example, 540 entered as 450) | When a reconciliation difference is divisible by 9, the system flags a likely transposition and lists candidate entries |
| **Duplication** | Same transaction entered twice | Same party, amount, date and reference within a window |
| **Cut-off** | Entry in the wrong period | Goods received before period end but invoiced after → suggested accrual (GRNI); prepaid expenses → suggested prepayment |
| **Unbalanced or unexplained differences** | Differences between sub-ledger and GL, cash over/short, stock valuation gaps | Daily reconciliation differences, with age and owner |
| **Suspense items** | Entries parked in a suspense account | Suspense ageing report; suspense must be zero before a period can close |

**3. Correction: a guided wizard that never edits history**
- Posted entries are never edited or deleted. The accountant selects the wrong entry, chooses the kind of correction (reverse and re-enter, reclassify to another account, correct the amount, or move to another period), and the system generates the correcting journal. That journal carries the reason, a link to the original and any attachment.
- Corrections above a threshold need approval (for example, a finance manager or the owner).
- If the original period is closed, the correction posts in the current period with a reference. Reopening a period requires two approvals and is logged.
- An **issue inbox** for the bookkeeper and accountant lists every detected problem with severity, the suggested fix and one-click apply (with approval), and shows when each issue was resolved.
- A **"what changed" report** shows every correction in a period for owners and auditors.

### 20.6 Period and year-end close (F16)

A guided close checklist: reconciliations complete; suspense cleared; health-check issues resolved or accepted with reason; accruals and prepayments; depreciation run; stock count adjustments posted; payroll and tips posted; tax computed. Then the period is locked. Year-end closes income and expense to retained earnings and opens the new year with carried-forward balances.

### 20.7 Financial statements and reports (F17)

Trial balance; profit and loss (by location, department and period); balance sheet; cash-flow statement; statement of changes in equity; aged debtors and creditors; tax return support; departmental and daily flash P&L; budget vs actual; prime cost. All are exportable to Excel and PDF. An **accountant pack** (all of the above plus ledgers and reconciliations for the period) can be generated in one click for the external accountant or auditor.

### 20.8 Finance process IDs

| ID | Capability | Key features |
|---|---|---|
| **F1** | Setup | Chart of accounts templates, dimensions, fiscal years and periods, entities, currencies, opening balances import |
| **F2** | Posting engine | Balanced, idempotent, source-linked postings; posting rules configurable per business; no direct postings bypassing rules |
| **F3** | Books of original entry | Cash book, bank book, petty cash book, daily sales book, sales/purchases day books and returns books, expense book, other income register, general journal |
| **F4** | Ledgers | General ledger, debtors ledger, creditors ledger, control accounts |
| **F5** | Income and expense tracking | Simple owner view, phone receipt capture with OCR, recurring bills, categories, budget alerts |
| **F6** | Petty cash | Imprest floats, vouchers with photos, approval, top-up |
| **F7** | Bank and cash management | Bank accounts, statement import and feeds, deposits, transfers, safes |
| **F8** | Reconciliations | Bank, cash, card/mobile money, control accounts, inventory, tax, liabilities, inter-company |
| **F9** | Accounts receivable | Customer invoices, receipts, credit control, statements, ageing, reminders |
| **F10** | Accounts payable | Supplier invoices, payment runs, supplier statements, ageing |
| **F11** | Tax | Tax engine, tax register, return preparation, fiscal/e-invoicing connectors |
| **F12** | Fixed assets | Asset register, depreciation methods, disposals |
| **F13** | Payroll and tips posting | Payroll journals from time clock and payroll provider; tips and service charge liabilities |
| **F14** | Customer liabilities | Deposits, gift cards, vouchers, container deposits |
| **F15** | Books health and error correction | Daily health checks, issue inbox, correction wizard, suspense control, "what changed" report |
| **F16** | Period and year-end close | Close checklist, locks, controlled reopening, year-end roll-forward |
| **F17** | Financial statements and reports | TB, P&L, balance sheet, cash flow, equity, ageing, tax, departmental and flash P&L, accountant pack |
| **F18** | Budgets and cash forecast | Budgets by account and dimension, budget vs actual, cash-flow forecast |
| **F19** | Accountant collaboration | Roles for bookkeeper, accountant, finance manager, external accountant/auditor (read-only); comments and document requests on entries |

---

## 21. Inventory core (IN1–IN11), working with Finance

Inventory is **perpetual**: every stock movement records both **quantity and value** and immediately posts the matching journal through the Finance core. The stock ledger and the general ledger therefore always agree. If they ever don't, the difference appears in the books health checks.

### 21.1 How each stock movement posts to the books

| Stock movement | Inventory effect | Journal posted by the Finance core |
|---|---|---|
| Goods received (before supplier invoice) | + quantity, + value at PO cost | Dr Inventory · Cr GRNI (goods received not invoiced) |
| Supplier invoice matched | Cost adjusted if invoice price differs (within tolerance) | Dr GRNI, Dr Input tax · Cr Supplier (creditors); price difference to Inventory or purchase price variance |
| Sale (by recipe for drinks and dishes) | − quantity at current cost | Dr COGS (by department) · Cr Inventory |
| Customer return to stock | + quantity at original cost | Dr Inventory · Cr COGS |
| Return to supplier | − quantity | Dr Supplier (debit note) · Cr Inventory |
| Transfer between locations | − at source, + at destination (via in-transit) | Dr Inventory in transit · Cr Inventory (source); then Dr Inventory (destination) · Cr Inventory in transit |
| Production (bakery, prep, cocktails batched) | − ingredients, + finished item | Dr Inventory (finished) · Cr Inventory (ingredients); yield differences to production variance |
| Waste / spoilage | − quantity | Dr Waste expense (by reason) · Cr Inventory |
| Comps, spills, staff drinks | − quantity | Dr Comps / Staff welfare / Marketing (by reason) · Cr Inventory |
| Count variance (shrink or gain) | ± quantity | Dr Shrinkage expense · Cr Inventory (or the reverse for a gain) |
| Revaluation (cost correction) | ± value only | Dr/Cr Inventory · Cr/Dr Inventory revaluation |
| Container deposits (kegs, crates) | Container ledger | Dr Deposits receivable · Cr Supplier (and reverse on return) |

### 21.2 Valuation and costing
- Weighted average cost (default) or FIFO per tenant; landed costs (freight, duties) added to item cost.
- Recipe cost rolls up from ingredient costs, so every drink and dish has a live cost and margin.
- Negative stock (selling before a delivery is recorded) is allowed only where configured. When the delivery arrives, the cost is corrected automatically with a revaluation entry.

### 21.3 Inventory process IDs

| ID | Capability | Links |
|---|---|---|
| **IN1** | Item, unit and pack master (catalogue) | S1, B4, R9 |
| **IN2** | Stock ledger and stock locations | S15, B1 |
| **IN3** | Valuation and costing (average, FIFO, landed cost, recipe cost) | F2 |
| **IN4** | Receiving and GRNI accrual | S3, F10 |
| **IN5** | Transfers and in-transit stock | S15, B1 |
| **IN6** | Recipes, production and yield | S7, B4, R9, R10, R15 |
| **IN7** | Counts and variance posted to the books | S11, B5 |
| **IN8** | Waste, comps, spills, staff consumption and shrink classification | S12, B6, R11 |
| **IN9** | Batch, lot and expiry | S12 |
| **IN10** | Inventory–finance reconciliation (stock valuation = GL inventory; GRNI listing = GRNI account) | F8, F15 |
| **IN11** | Inventory insights: stock value, days of cover, stock turn, dead stock, reorder needs, variance hot spots | S2, B5 |

---

## 22. Sales core (SA1–SA12), working with Inventory and Finance

The Sales core turns every sale into correct stock and correct books automatically. It also turns the combined data into **quick reports and simple tasks** that owners and managers can act on.

### 22.1 What happens when a sale completes

1. **Sales record:** the sales document (receipt, tab, table bill, credit invoice, online or delivery order) is finalised with lines, modifiers, discounts, tax, tenders, tips and customer.
2. **Inventory:** each line depletes stock through its recipe (IN6) at current cost (IN3).
3. **Finance:** the posting engine (F2) records:
   - Revenue by department and category; output tax by rate.
   - Tenders to **clearing accounts** (cash to till, card to card clearing, mobile money to mobile-money clearing, account sales to debtors, gift cards and deposits to their liability accounts).
   - Discounts, as a reduction of revenue or as a promotion expense, by policy; vendor-funded amounts to a supplier claim receivable.
   - Tips and service charges to liability accounts.
   - COGS against inventory.
4. **Settlement:** when the card provider pays out, the clearing account is cleared to the bank, with fees posted to card-fee expense (F8). Cash moves from till to safe to bank through the cash book.

### 22.2 Quick reports (answers in one tap)

Reports use plain-language names, and each one opens from the home screen of the roles that need it (Section 7).

| Quick report | Answers | For |
|---|---|---|
| **Today at a glance** | Sales, profit estimate, customers/covers, average sale, compared with the same day last week | Owner, managers |
| **What sold and what made money** | Best sellers by quantity and by margin; slow sellers | Owner, buyers, bar manager, chef |
| **How we were paid** | Payment mix, card fees, cash expected vs counted | Managers, accountant |
| **Profit this week / month** | Simple P&L (sales − cost of sales − expenses) | Owner |
| **Cash position** | Cash in tills, safe and bank; what's coming in and going out | Owner, accountant |
| **Who owes us / whom we owe** | Aged debtors and creditors | Owner, accountant |
| **Stock value and stock to order** | Stock on hand at cost, low stock, suggested orders | Managers, buyers |
| **Waste, variance and loss** | Waste, comps, shrink and liquor variance in money | Owner, bar manager, store manager |
| **Tax due** | Tax collected minus tax paid for the period | Owner, accountant |
| **Staff performance** | Sales per hour, voids, discounts, tips by person | Managers |

### 22.3 From insight to task

Reports are only useful if someone acts on them. The Sales core feeds an **insight-to-task engine** that turns findings from all three cores into short, assignable tasks with the data attached. For example:
- "Order 14 items that will run out before the next delivery" (Inventory + Sales forecast).
- "3 card batches from Tuesday not yet settled" (Finance reconciliation).
- "Vodka variance 9% this week at Bar 2: recount and review comps" (Inventory + Sales).
- "Supplier invoice INV-2231 is 6% above PO price: approve or claim" (Finance + Inventory).
- "Lunch sales down 18% vs last month: review menu prices of 5 low-margin dishes" (Sales + Finance).

Tasks go to the right role (Section 7.6), carry a due time, and close automatically when the underlying issue is resolved.

### 22.4 Sales process IDs

| ID | Capability | Links |
|---|---|---|
| **SA1** | Sales document model (receipt, tab, table bill, credit invoice, online, delivery, QR, self-checkout) | S5, B3, R7 |
| **SA2** | Tenders and clearing accounts (cash, card, mobile money, account, gift card, deposit) | F7, F8 |
| **SA3** | Discounts and promotions accounting (including vendor-funded claims) | S4, F2 |
| **SA4** | Returns, refunds and credit notes | S8, F3 |
| **SA5** | Credit sales and customer accounts | S13, F9 |
| **SA6** | Deposits, gift cards and vouchers | B10, R1, F14 |
| **SA7** | Tips and service charges | B8, F13 |
| **SA8** | Daily sales close (Z) posted to the daily sales book | S10, B13, R14, F3 |
| **SA9** | Real-time margin per line, sale, category and shift | IN3 |
| **SA10** | Quick reports (22.2) | Section 12 |
| **SA11** | Insight-to-task engine (22.3) | Section 7.6 |
| **SA12** | Channel sales (online, delivery platforms with commissions, QR, kiosk) | S14, R8 |

---

## 23. How the three cores work together

### 23.1 Worked example: a gin and tonic on a bar tab, paid by card

Assumptions: price 11.50 including 20% VAT; gin cost 1.60 (50 ml), tonic cost 0.50; tip 1.00; card fee 3%.

| Step | Sales core | Inventory core | Finance core (journal) |
|---|---|---|---|
| Drink rung on the tab | Tab line: G&T 11.50 | — (depleted when the tab is paid, or immediately if configured) | — |
| Tab paid by card with tip | Receipt: 11.50 + tip 1.00 = 12.50 card | Gin −50 ml (1.60), tonic −1 (0.50) | Dr Card clearing 12.50 · Cr Beverage sales 9.58 · Cr VAT output 1.92 · Cr Tips payable 1.00 |
| | | | Dr COGS beverage 2.10 · Cr Inventory 2.10 |
| Next day: card settlement | — | — | Dr Bank 12.12 · Dr Card fees 0.38 · Cr Card clearing 12.50 |
| Tips paid out with payroll | — | — | Dr Tips payable 1.00 · Cr Payroll/cash 1.00 |

Result: the bar sees the sale and its margin (9.58 − 2.10 = 7.48) immediately; the stock ledger and the GL both show 2.10 less inventory; the card clearing account returns to zero when the provider settles; the tip is a liability until paid out.

### 23.2 Worked example: a supermarket delivery

| Step | Inventory core | Finance core (journal) |
|---|---|---|
| Delivery received against PO (100 cases at 10.00) | +100 cases, value 1,000.00 | Dr Inventory 1,000.00 · Cr GRNI 1,000.00 |
| 2 cases damaged, returned on the spot | −2 cases, debit note raised | Dr GRNI 20.00 · Cr Inventory 20.00 |
| Supplier invoice arrives: 98 cases, 980.00 + 20% VAT | — | Dr GRNI 980.00 · Dr VAT input 196.00 · Cr Supplier 1,176.00 |
| Supplier paid | — | Dr Supplier 1,176.00 · Cr Bank 1,176.00 |

If the invoice doesn't arrive by period end, the GRNI balance remains as a correct accrual, and a health check (F15, omission) reminds the accountant to chase it.

### 23.3 Daily three-way reconciliation

Every night (and on demand), the platform proves that the three cores agree. Any failure becomes an item in the finance issue inbox (F15) and a task for the right role (SA11).

| Check | Must be equal |
|---|---|
| Sales ↔ Finance | Net sales + tax + tips + deposits in the Sales core = the related revenue, tax and liability postings in the GL, per day and location |
| Sales ↔ Inventory | Recipe depletion from completed sales = sale movements in the stock ledger |
| Inventory ↔ Finance | Stock valuation = inventory accounts in the GL; GRNI listing = GRNI account; COGS in the stock ledger = COGS in the GL |
| Tenders ↔ Finance | Tender totals from Z-closes = movements in cash, card and mobile-money clearing accounts |
| Settlements ↔ Bank | Provider settlements = bank lines; clearing accounts return to zero within the expected settlement delay |
| Sub-ledgers ↔ GL | Debtors, creditors, tips, gift cards and deposits = their control accounts |

### 23.4 Consistency guarantees

- A sale, stock movement and journal are committed together, or through an outbox that guarantees delivery exactly once.
- A posting can never be silently dropped. If a posting rule fails (for example, an unmapped tax code), the document is held in the issue inbox and the owner is alerted, and the books show a clearly labelled "unposted" amount until it's fixed.
- The books are never more than a few minutes behind trading when online. Offline terminals catch up on reconnection, and the gap is visible.

### 23.5 Build order

1. **Document engine (DOC1–DOC12, Section 24)** and **Finance core (F1–F19)**, including posting engine, books, ledgers, reconciliations, health checks and close.
2. **Inventory core (IN1–IN11)**, wired to Finance: every movement posts; IN10 reconciliation passes.
3. **Sales core (SA1–SA12)**, wired to both: every sale depletes stock and posts; the three-way reconciliation passes; quick reports and insight-to-task work.
4. Then the POS terminal apps, payments, purchasing and the supermarket, bar and resto-bar modes, all on top of the cores.

For a build already under way, the implementation prompt guide (Part H) contains an **upgrade path** that re-plans the existing code around these three cores without discarding working code.

---

## 24. Business documents and document flows (DOC1–DOC12)

> **Added in v0.5.** Every event in the three cores (Section 19) starts from a **business document**. This section defines which documents the platform supports, what each one does to stock and to the books, how they link into flows for buying and selling, and the document engine that manages them.

### 24.1 Kinds of document

Not every document is an invoice. The platform classifies documents by what they prove or change, which determines their effect on the cores:

| Kind | What it does | Effect on the cores | Examples |
|---|---|---|---|
| **Commercial** | Proposes, requests or commits to a transaction | None on stock or books; tracked as commitments (open orders, budget commitments, reserved stock) | Purchase requisition, RFQ, quotation, proforma invoice, purchase order, sales order |
| **Delivery / stock** | Proves goods physically moved | Stock movement (Inventory core); accrual in the books where needed (GRNI) | Delivery note, goods received note, return note, transfer note, stock adjustment |
| **Billing** | Records the amount legally owed | Books (Finance core): receivable or payable, revenue or cost, tax | Tax invoice, commercial invoice, advance payment invoice, credit note, debit note |
| **Payment** | Authorises, records or proves payment | Books: cash, bank or clearing accounts; settles receivables and payables | Payment voucher, receipt, remittance advice, deposit slip |
| **Accounting adjustment** | Corrects or adjusts the books without a new trade | Books only, through the correction wizard (F15) | Journal voucher, credit/debit adjustment |
| **Reporting** | Summarises a relationship or a period | None (read-only) | Statement of account, Z-report, cash-up sheet |

### 24.2 Document catalogue

**Core documents (from the procurement → delivery → invoicing → payment cycle)**

| # | Document | Main purpose | Issued by → to | Stock effect | Finance effect | Book / ledger |
|---|---|---|---|---|---|---|
| 1 | **Purchase requisition (PR)** | Internal request to buy goods or services | Staff/manager → buyer | None | Optional budget pre-commitment | — |
| 2 | **Request for quotation (RFQ)** | Ask one or more suppliers for prices and terms | Buyer → suppliers | None | None | — |
| 3 | **Quotation** | Price and terms offered | Supplier → us (buying); us → customer (selling) | None | None | — |
| 4 | **Proforma invoice** | Preliminary invoice showing what is expected to be charged; often used to request prepayment or for import/customs | Seller → buyer | None | None (not an accounting document) | — |
| 5 | **Purchase order (PO)** | Official order to a supplier | Us → supplier | "On order" quantity | Budget commitment | — |
| 6 | **Sales order (SO)** | Records a customer's confirmed order | Us (seller) | Stock reserved/allocated | None (commitment) | — |
| 7 | **Delivery note (DN)** | Confirms what goods were dispatched/delivered; travels with the goods | Seller → buyer | Selling: stock out (and COGS). Buying: basis for our GRN | Selling: COGS posted | Stock ledger |
| 8 | **Goods received note (GRN)** | Buyer confirms what was actually received (quantity, condition, batch, expiry) | Us (receiver) | Stock in | Dr Inventory (or asset/expense) · Cr GRNI | Stock ledger; GRNI |
| 9 | **Tax invoice / commercial invoice** | Legally records the amount owed, with tax | Seller → buyer | None (stock already moved) | Selling: Dr Debtors · Cr Sales, Cr Output tax. Buying: Dr GRNI/expense, Dr Input tax · Cr Supplier | Sales or purchases day book; debtors or creditors ledger |
| 10 | **Advance payment invoice** | Requests or records payment before delivery | Seller → buyer | None | Selling: customer advance (liability). Buying: supplier advance (asset). Tax on advances where the law requires it | Sales/purchases day book |
| 11 | **Receipt** | Proof that payment was received | Seller → buyer | None | Selling: Dr Cash/Bank/Clearing · Cr Debtors (or sales, for a cash sale) | Cash book / bank book |
| 12 | **Credit note** | Reduces a previously invoiced amount (return, discount, price error) | Seller → buyer | If goods returned: stock in (selling) or out (buying) | Reverses revenue/cost and tax proportionally | Sales/purchases returns day book |
| 13 | **Debit note** | *Seller's use:* increases a previously invoiced amount (undercharge, extra cost). *Buyer's use:* claims a reduction from the supplier (return, shortage, damage), which the supplier answers with a credit note | Seller → buyer, or buyer → supplier | Buyer's claim with return: stock out | Seller's: additional receivable, revenue and tax. Buyer's: Dr Supplier · Cr Inventory/GRNI (claim) | Sales day book or purchases returns day book |
| 14 | **Payment voucher** | Internal document that authorises and records a payment, with approvals and attached invoices | Us (accounts) | None | Dr Supplier (or expense) · Cr Bank/Cash | Cash book / bank book; creditors ledger |
| 15 | **Credit/debit adjustment (journal voucher)** | Accounting correction or adjustment without a new trade (reclassification, write-off, accrual) | Us (accountant) | Only if it is a stock revaluation | Any balanced adjustment, with reason and approval | General journal |
| 16 | **Statement of account** | Summary of invoices, credit notes, payments and outstanding balance for one customer or supplier | Us → customer; supplier → us | None | None (reconciled against the ledger) | Debtors/creditors ledger |

**Other documents that modern POS and ERP systems use**

| Document | Purpose | Used in | Core effect |
|---|---|---|---|
| **POS receipt (simplified tax invoice)** | At the till, one document is both the simplified tax invoice and proof of payment | All counters, bars, cafés | Sale + payment together (SA1, SA2) |
| **Full tax invoice on request** | Issued from a POS receipt for a business customer, with their tax ID; references the receipt | Supermarket, restaurant (business meals) | Replaces the simplified invoice for tax purposes; no double revenue |
| **Refund receipt** | Proof of a refund at the till | All | Credit note + payment out |
| **Deposit receipt** | Proof of a reservation, event or container deposit | Resto-bar, bar (VIP), supermarket (crates) | Liability (F14) |
| **Gift card / voucher receipt** | Proof of sale or top-up of stored value | All | Liability (F14) |
| **Order ticket (KOT/BOT)** | Kitchen or bar order ticket sent to KDS/BDS or printer | Bar, resto-bar, café | None (operational) |
| **Guest check / table bill** | Running bill for a table or tab before payment | Bar, resto-bar | None until paid (then POS receipt) |
| **Pick list / packing slip** | Picking and packing for online, click & collect and B2B orders | Supermarket | None (operational) |
| **Return to vendor (RTV) note** | Goods sent back to a supplier | All | Stock out; buyer's debit note |
| **Customer return note** | Goods returned by a customer | Supermarket | Stock in (or to damaged); credit note |
| **Transfer note** | Stock moved between stores or from store room to bar/kitchen | All | Stock transfer (IN5) |
| **Stock adjustment / count sheet** | Records counts and approved differences | All | Stock + variance posting (IN7) |
| **Production order / batch sheet** | Records in-store production (bakery, prep, batched cocktails) | Supermarket, resto-bar, bar | Production movement (IN6) |
| **Waste note** | Records spoiled, broken or expired goods | All | Stock out + waste expense (IN8) |
| **Z-report / cash-up sheet** | End-of-day or end-of-shift sales and cash summary | All | Daily sales book (SA8) |
| **Petty cash voucher** | Small cash payment with receipt photo | All | Petty cash book (F6) |
| **Expense claim** | Staff reimbursement request | All | Expense book, payable to staff |
| **Remittance advice** | Tells a supplier which invoices a payment covers | Buying | None (sent with payment) |
| **Bank deposit slip** | Cash banked from the safe | All | Cash book → bank book |
| **Payment reminder (dunning letter)** | Chases overdue customer invoices | B2B selling | None |

### 24.3 Buying flows (procure-to-pay)

**Standard (postpaid / credit terms)**

> **Requisition → RFQ → Quotation → (Proforma) → Purchase order → Delivery note → GRN → Tax invoice → Payment voucher → Payment → Receipt**

**Prepaid**

> **PO → Proforma or advance payment invoice → Payment voucher → Payment (supplier advance) → Delivery note → GRN → Final tax invoice (advance allocated) → Receipt**

**Direct / cash purchase** (for example, buying ice or lemons from a market)

> **Purchase → Invoice or cash receipt → Immediate payment (petty cash or card) → Receipt**, recorded as one quick entry that creates the GRN (if it is stock), the invoice and the payment together.

**How this fits each business**

| Situation | Typical flow |
|---|---|
| Supermarket, warehouse supplier on credit | PO → DN + GRN on handheld → invoice matched (three-way) → payment run → remittance advice |
| Supermarket, direct-store-delivery supplier (bread, milk, soft drinks) | Supplier's DN and invoice arrive together at the door → GRN against standing order → invoice matched on the spot |
| Bar, beverage distributor | Suggested order → PO → DN with kegs/crates → GRN (including returnable containers) → invoice → payment; empty kegs returned on a return note, deposit credited |
| Resto-bar, market purchases | Direct purchase with petty cash or card → receipt photo → one quick entry |
| Any business buying equipment (fixed asset) | PR → RFQ → quotations compared → PO → DN → GRN → invoice posted to fixed assets (not inventory) → payment → asset registered, depreciation starts |

**Matching rules**
- **Two-way match:** PO ↔ invoice (services, where nothing is physically received).
- **Three-way match:** PO ↔ GRN ↔ invoice (goods), with price and quantity tolerances.
- **Four-way match (optional):** adds a quality inspection record (for example, cold-chain temperature at receipt).

### 24.4 Selling flows (order-to-cash)

**Walk-in POS sale (most sales in all three businesses)**

> **Sale at the till → Payment → POS receipt** (simplified tax invoice and proof of payment in one). If the customer is a business, a **full tax invoice** can be issued from the receipt.

**Bar tab or restaurant table**

> **Order tickets (KOT/BOT) → Guest check / tab → Payment (card pre-authorised for tabs) → POS receipt**, with tips recorded on the receipt.

**Account customer (for example, a restaurant buying from the supermarket on credit)**

> **Quotation → Sales order → Pick list → Delivery note → Tax invoice → Statement of account (monthly) → Payment → Receipt**, with payment reminders for overdue invoices.

**Prepaid selling (events, catering, large orders, online orders)**

> **Quotation → Proforma or advance payment invoice → Payment → Deposit receipt → Delivery / event → Final tax invoice (advance allocated) → Receipt for any balance**

**Returns and corrections**

> **Customer return note → Credit note → Refund (refund receipt) or credit to account**, and **Debit note** for an undercharge.

### 24.5 Important distinctions the system enforces

- **A delivery note is not an invoice.** A delivery note or GRN moves stock; only a tax/commercial invoice creates the receivable or payable.
- **A proforma invoice is not an accounting document.** It never posts to the books, and it is numbered in its own series so it can't be confused with a tax invoice.
- **A receipt proves payment; an invoice records what is owed.** At the till, the POS receipt combines both for a cash or card sale. For credit sales they remain separate.
- **Issued documents are never edited or deleted.** A wrong tax invoice is corrected with a credit note (and a new invoice if needed), and a wrong PO with a revision. Every change is audited, and fiscal rules (sequential numbering, e-invoicing submission) are respected.
- **Debit notes have two meanings** (24.2, #13). The system labels which one each debit note is: seller's supplementary charge or buyer's claim.
- **Advance payments are held separately** (customer advances as a liability, supplier advances as an asset) until the final invoice is issued, then allocated to it automatically.

### 24.6 Worked examples

**Buying 10 laptops for the office (fixed asset, 20% VAT), postpaid**

| Document | Stock / asset | Journal |
|---|---|---|
| Requisition, RFQ, quotations (3 compared), proforma | — | — |
| Purchase order: 10 × 1,000.00 | On order: 10 | Budget commitment 10,000.00 (no journal) |
| Delivery note + GRN: 10 received | Asset under receipt | Dr Fixed assets (IT equipment) 10,000.00 · Cr GRNI 10,000.00 |
| Tax invoice: 10,000.00 + 2,000.00 VAT | — | Dr GRNI 10,000.00 · Dr VAT input 2,000.00 · Cr Supplier 12,000.00 |
| Payment voucher approved, payment made | — | Dr Supplier 12,000.00 · Cr Bank 12,000.00 |
| Supplier's receipt attached | 10 laptops added to the fixed asset register; depreciation scheduled | — |

*Prepaid version:* payment before delivery posts Dr Supplier advances 12,000.00 · Cr Bank 12,000.00. When the final invoice arrives, the advance is allocated: Dr Supplier 12,000.00 · Cr Supplier advances 12,000.00. Some countries allow or require input VAT to be recognised at the advance invoice; the tax engine applies the jurisdiction's rule.

**Selling 50 cases of soft drinks to a restaurant on account (price 20.00, cost 14.00, 20% VAT)**

| Document | Stock | Journal |
|---|---|---|
| Quotation → sales order | 50 reserved | — |
| Pick list → delivery note | −50 cases | Dr COGS 700.00 · Cr Inventory 700.00 |
| Tax invoice | — | Dr Debtors 1,200.00 · Cr Sales 1,000.00 · Cr VAT output 200.00 |
| Statement of account (month end) | — | — |
| Payment received → receipt | — | Dr Bank 1,200.00 · Cr Debtors 1,200.00 |
| 2 damaged cases returned → return note + credit note | +2 to "damaged", then written off | Dr Sales returns 40.00 · Dr VAT output 8.00 · Cr Debtors 48.00; Dr Inventory 28.00 · Cr COGS 28.00; Dr Waste 28.00 · Cr Inventory 28.00 |

### 24.7 Document engine features

| ID | Feature | Details |
|---|---|---|
| **DOC1** | Document types and numbering | Configurable document types; separate number series per type, entity, location, terminal and fiscal year; sequential and gapless where the law requires; prefixes (for example, `INV-2026-000123`, `PF-…` for proformas) |
| **DOC2** | Lifecycle and status | Draft → submitted → approved → issued/sent → partially fulfilled → fulfilled → closed; cancelled/voided with rules per type (issued billing documents are corrected by credit note, never voided silently) |
| **DOC3** | Convert and copy | Create the next document from the previous one with one action (RFQ → PO, quotation → SO → DN → invoice, PO → GRN → invoice), carrying lines, prices, taxes and references |
| **DOC4** | Partial fulfilment tracking | Quantities ordered, delivered/received, invoiced, returned and paid, per line; back-orders; over/under-delivery tolerances |
| **DOC5** | Document flow view | Every document shows its full chain (predecessors and successors) as a tree, with amounts and statuses; open from any document |
| **DOC6** | Matching | Two-, three- and four-way matching with tolerances; exceptions to the approval queue (links F10, S3) |
| **DOC7** | Payment terms and flows | Prepaid, postpaid (net days, end-of-month, instalments), cash on delivery, direct cash; early-payment discounts; advances and their allocation |
| **DOC8** | Approvals and controls | Approval rules per document type and amount (from 7.5); segregation of duties (requester ≠ approver; receiver ≠ invoice approver) |
| **DOC9** | Templates and legal content | Printable/PDF templates per country and language with mandatory legal fields (seller/buyer tax IDs, tax breakdown, fiscal codes, QR codes where required) |
| **DOC10** | E-invoicing and sending | Send by email, WhatsApp, supplier/customer portal or EDI; structured e-invoice formats (for example, UBL/Peppol or the national format) through the fiscal connector (F11) |
| **DOC11** | Revisions and audit | PO and quotation revisions with version history; attachments (photos, signed delivery notes, supplier receipts); full audit trail |
| **DOC12** | Statements and reminders | Customer and supplier statements of account; automatic payment reminders; supplier statement reconciliation (F8) |

### 24.8 Where each document is created (roles and screens)

| Document | Created by (role, Section 7) | Where |
|---|---|---|
| Purchase requisition | Department manager, bar manager, head chef, head barista | Handheld/tablet, or automatically from suggested orders |
| RFQ, supplier quotation comparison, PO | Buyer, bar manager, purchasing officer | Back office |
| GRN, return to vendor note | Receiving clerk, cellar person, storekeeper | Handheld at the dock |
| Supplier tax invoice, debit note, payment voucher | Accountant / bookkeeper | Back office; invoice capture by photo/PDF |
| Quotation, proforma, sales order, delivery note, tax invoice (B2B) | Store manager, events manager, B2B sales | Back office or tablet |
| POS receipt, refund receipt, deposit receipt | Cashier, bartender, server, barista | Terminal / handheld |
| KOT/BOT, guest check | Server, bartender | Terminal / handheld |
| Transfer, waste, count, production documents | Stock clerk, barback, cooks, bakers | Handheld / tablet |
| Journal voucher, adjustments | Accountant | Back office (correction wizard) |
| Statement of account, reminders | Accountant (automated) | Back office, scheduled |

---

## Appendix A: Feature matrix by mode

| Feature | Core | Supermarket | Bar | Resto-bar |
|---|:---:|:---:|:---:|:---:|
| **Document engine and flows: PR, RFQ, quotation, proforma, PO, SO, DN, GRN, invoices, credit/debit notes, payment vouchers, receipts, statements (DOC1–DOC12)** | ● | ● | ● | ● |
| **Finance core: books, ledgers, reconciliations, health checks, close (F1–F19)** | ● | ● | ● | ● |
| **Inventory core: perpetual stock posted to the books (IN1–IN11)** | ● | ● | ● | ● |
| **Sales core: sales posted to stock and books, quick reports, insight-to-task (SA1–SA12)** | ● | ● | ● | ● |
| Cloud back office + offline terminals | ● | ● | ● | ● |
| Integrated payments, SoftPOS | ● | ● | ● | ● |
| Inventory with multi-UoM, recipes, batch/expiry | ● | ● | ● | ● |
| Purchasing with suggested orders and approvals | ● | ● | ● | ● |
| Receiving on handheld, supplier claims | ● | ● | ● | ● |
| Invoice capture and three-way match | ● | ● | ● | ● |
| ERP / accounting posting and connectors | ● | ● | ● | ● |
| Loyalty, gift cards, credit accounts | ● | ● | ● | ● |
| Staff roles, scheduling, time clock, tips | ● | ● | ● | ● |
| Scale & PLU integration, label scales | | ● | | |
| Self-checkout / kiosks | | ● | | ○ |
| Electronic shelf labels | | ● | | |
| Expiry markdowns | | ● | ○ | ○ |
| Promotions engine (mix & match, vendor-funded) | ○ | ● | ○ | ○ |
| In-store production (bakery/deli) | | ● | | ○ |
| Click & collect / picking app | | ● | | ○ |
| Tabs with pre-authorisation | | | ● | ● |
| Speed screen / BDS | | | ● | ● |
| Pour-level liquor variance, bottle scales | | | ● | ● |
| Returnable containers (kegs, crates) | | ○ | ● | ● |
| Happy-hour / time-based pricing | ○ | ○ | ● | ● |
| Door, cover charge, capacity, minimum spend | | | ● | ○ |
| Floor plan & table management | | | ○ | ● |
| Barista display, drink builder, coffee recipes | | ○ (in-store café) | ○ | ● |
| KDS, coursing, expo | | ○ (deli) | | ● |
| Allergen management | | ○ | | ● |
| QR table ordering & pay-at-table | | | ○ | ● |
| Reservations & waitlist | | | ○ | ● |
| Delivery aggregator integration | | ○ | | ● |
| Recipe costing & menu engineering | | ○ | ● | ● |
| AI forecasting, anomaly detection | ○ | ● | ● | ● |
| Intelligence layer over existing POS (Mode 3) | ○ | ○ | ○ | ○ |
| Role templates, threshold permissions, remote approvals | ● | ● | ● | ● |
| Management routines, checklists, handover notes, tasks | ● | ● | ● | ● |
| Staff app (rota, swaps, tips, training) | ● | ● | ● | ● |
| Environment-specific UI (dark bar, KDS bump bars, lane ergonomics) | ● | ● | ● | ● |

● = included / core to the mode  ○ = optional add-on

## Appendix B: Glossary

- **BDS / KDS:** Bar / Kitchen Display System; screens that replace paper order tickets.
- **Barback:** Bar assistant who restocks, changes kegs and clears glasses.
- **Barista:** Staff member who prepares espresso-based and other coffee drinks.
- **Books of original entry (day books):** Books where transactions are first recorded from source documents before being posted to the ledger (cash book, sales day book, purchases day book, returns books, general journal).
- **Advance payment invoice:** An invoice requesting or recording payment before delivery; creates a customer advance (seller) or supplier advance (buyer) until the final invoice.
- **BoM:** Bill of materials; the list of ingredients or components in a product (a recipe, in hospitality).
- **Credit note:** A document from the seller that reduces a previously invoiced amount.
- **Clearing account:** A temporary account that holds money between two events, such as a card sale and the provider's settlement to the bank.
- **Control account:** A general-ledger account whose balance must equal the total of a subsidiary ledger (for example, debtors control = sum of customer balances).
- **COGS:** Cost of goods sold.
- **Debit note:** Either a seller's document that increases a previously invoiced amount, or a buyer's claim asking the supplier for a reduction (answered by a credit note).
- **Delivery note (DN):** A document that travels with goods and confirms what was delivered; not an invoice.
- **DSD:** Direct store delivery; suppliers who deliver straight to the store rather than via a warehouse.
- **DSR:** Daily sales report.
- **E2EE / P2PE:** End-to-end / point-to-point encryption; card data is encrypted inside the payment terminal.
- **EMV:** The global chip-card payment standard.
- **ESL:** Electronic shelf label; a digital price tag updated from the POS.
- **Expeditor (expo):** Person, or screen, that coordinates finished food and drinks so a table's order leaves together.
- **FEFO / FIFO:** First-expired-first-out / first-in-first-out stock rotation.
- **KOT / BOT:** Kitchen / bar order ticket.
- **GRNI:** Goods received not invoiced; a liability recorded when stock arrives before the supplier's invoice.
- **GRN:** Goods received note.
- **GTIN:** Global Trade Item Number; the number behind a product barcode.
- **Imprest system:** A petty cash method where the float is topped up to a fixed amount after spending.
- **Perpetual inventory:** Stock records updated with quantity and value at every movement, instead of only at periodic counts.
- **Payment voucher:** An internal document that authorises and records a payment.
- **PLU:** Price look-up code, used for produce and weighted items.
- **Proforma invoice:** A preliminary invoice showing what is expected to be charged; not an accounting document.
- **Purchase requisition (PR):** An internal request to buy goods or services.
- **PMS:** Hotel property management system.
- **Pour cost:** Cost of the liquor in a drink ÷ its selling price.
- **Pre-authorisation:** A temporary hold on a card when a tab is opened.
- **Receipt:** Proof that payment was received. At the till, the POS receipt is usually also the simplified tax invoice.
- **Remote approval:** A manager approves a staff request (void, refund, discount) on their phone instead of entering a PIN at the terminal.
- **RFQ:** Request for quotation sent to suppliers.
- **Role template:** A predefined set of screens and permissions for a job role, adjustable per person.
- **Posting engine:** The component that turns business documents into balanced journal entries using configured rules.
- **Statement of account:** A periodic summary of invoices, credit notes, payments and the outstanding balance for one customer or supplier.
- **SoftPOS:** Software that accepts contactless cards on an ordinary NFC phone or tablet.
- **Suspense account:** A temporary account for amounts that can't yet be classified; must be cleared before a period closes.
- **SUS (System Usability Scale):** A standard 10-question usability survey scored 0–100.
- **System of record:** The system whose data is treated as the official version of a given object.
- **Trial balance:** A list of all ledger balances; total debits must equal total credits.
- **Tax invoice / commercial invoice:** The document that legally records the amount owed, including tax.
- **Three-way match:** Checking a supplier invoice against the purchase order and goods received note before paying.
- **UoM:** Unit of measure.
- **Variance:** The difference between the stock that should have been used (according to sales) and the stock actually used (according to counts).

## Sources

Industry articles, vendor pages and documentation consulted in September 2026. Figures from vendors and market-research firms are indicative and should be verified before they are used in a funding or procurement document. Descriptions of what "existing POS" and "existing ERP" systems provide are typical of each category; individual products and plans differ.

- Supermarket / grocery POS: [LogicERP: Top 5 Supermarket POS Trends 2026](https://www.logicerp.com/blog/top-5-supermarket-pos-software-2026-trends/) · [IT Retail: Best Supermarket POS Systems](https://www.itretail.com/blog/best-supermarket-pos-system) · [PaymentNerds: Supermarket POS Features Guide 2026](https://paymentnerds.com/blog/supermarket-pos-system-features-that-streamline-inventory-loyalty-and-checkout/) · [LOC Software: Modern Supermarket POS Features](https://locsoftware.com/how-supermarket-pos-technology-solves-business-problems/) · [retailcloud: Essential POS Features 2026](https://retailcloud.com/modern-retail-pos-features/) · [BMC POS: Best POS for Grocery 2026](https://bmc-pos.com/best-pos-systems-for-grocery-stores/)
- Scales and ESL: [IT Retail: Electronic Shelf Labels](https://www.itretail.com/blog/electronic-shelf-labels) · [IT Retail: Deli Scale POS](https://www.itretail.com/deli-scale-pos) · [ScaleBlog: How checkout scales send weight to POS](https://scaleblog.com/grocery-pos-scale-produce-weighing/) · [ElectronicShelfTags: POS-compatible ESL 2026](https://www.electronicshelftags.com/pos-compatible-electronic-shelf-labels-the-2026-integration-standard/)
- Enterprise retail and loss prevention: [NCR Voyix: Retail Loss Prevention](https://www.ncrvoyix.com/platform/retail-applications/loss-prevention) · [NCR Voyix: Retail Platform](https://www.ncrvoyix.com/industry/retail) · [Viewpoint Analysis: POS Software Options 2026](https://www.viewpointanalysis.com/post/pos-software-options-2026)
- ERP-integrated POS: [LS Retail: LS Central for Retail](https://www.lsretail.com/products/ls-central-for-retail) · [LS Retail: LS Central and Dynamics 365 Business Central](https://www.lsretail.com/products/microsoft-dynamics-erp) · [ERP Research: LS Central Review 2026](https://www.erpresearch.com/erp-add-ons/pos/ls-central) · [Odoo 18 Point of Sale documentation](https://www.odoo.com/documentation/18.0/applications/sales/point_of_sale.html) · [Odoo 18 Restaurant features](https://www.odoo.com/documentation/18.0/applications/sales/point_of_sale/restaurant.html) · [Odoo POS Restaurant features](https://www.odoo.com/app/point-of-sale-restaurant-features) · [Ksolves: ERPNext POS vs Traditional POS](https://www.ksolves.com/blog/erpnext/erpnext-pos-vs-traditional-pos-systems) · [ClefinCode: ERPNext Offline Sync](https://clefincode.com/blog/extra-notes-for-you/en/solving-offline-erp-challenges-with-a-custom-erpnext-synchronization-solution) · [Gitnux: Integrated Retail Software 2026](https://gitnux.org/best/integrated-retail-software/)
- Enterprise hospitality: [Oracle MICROS Simphony](https://www.oracle.com/food-beverage/micros/) · [Restaurant Inventory Management Software: Oracle MICROS Simphony Review 2026](https://restaurantinventorymanagementsoftware.com/solutions/oracle-micros) · [POSUSA: Oracle Simphony Review](https://www.posusa.com/oracle-micros-simphony-pos/)
- Bar POS: [The Restaurant HQ: Best Bar POS 2026](https://www.therestauranthq.com/technology/best-bar-pos-system/) · [tech.co: Best Bar POS](https://tech.co/pos-system/best-bar-pos) · [Lavu: Best POS for Bars](https://lavu.com/best-pos-for-bar/) · [POSUSA: Best POS for Bars](https://www.posusa.com/best-pos-systems-for-bars/)
- Liquor inventory and variance: [BinWise: Bar Inventory Guide](https://home.binwise.com/guides/bar-inventory-management) · [BinWise: 5 Causes of Variance](https://home.binwise.com/blog/5-causes-of-variance) · [BinWise: Bevager vs BevSpot vs BinWise](https://home.binwise.com/blog/bevager-vs-bevspot-vs-binwise)
- Roles, permissions and café POS: [Lightspeed Restaurant: Managing user permissions](https://o-series-support.lightspeedhq.com/hc/en-us/articles/31329418175003-Managing-user-permissions) · [Lightspeed Restaurant: User roles](https://o-series-support.lightspeedhq.com/hc/en-us/articles/35765184351387-Managing-staff-access-with-user-roles) · [TouchBistro: Deciding on staff permissions](https://www.touchbistro.com/blog/how-to-decide-on-staff-permissions-settings-in-your-pos/) · [LS Central: Staff permissions and hospitality POS commands](https://help.lsretail.com/lscentral250/Content/LS-Hospitality/Dining-Table-Management/Staff-Permissions-And-Hospitality.htm) · [SumUp: Managing team POS access](https://www.sumup.com/en-gb/running-business/management/managing-teams-pos-access-effectively/) · [KORONA: Best coffee shop POS 2026](https://koronapos.com/blog/best-pos-system-for-coffee-shop/) · [Expert Market: Best POS for cafés](https://www.expertmarket.com/pos/best-pos-systems-for-cafes)
- Restaurant / resto-bar POS: [EHL Insights: Restaurant Technology 2026](https://insights.ehl.edu/restaurant-technology) · [Expert Market: Best Restaurant POS 2026](https://www.expertmarket.com/pos/best-pos-restaurants) · [RestroScout: Toast vs Square vs Lightspeed](https://restroscout.com/best-restaurant-pos-systems) · [Guideflow: Kitchen Display Systems 2026](https://www.guideflow.com/blog/kitchen-display-system) · [LithosPOS: POS Trends 2026](https://lithospos.com/blog/pos-system-trends-2026-ai-voice-ordering-cloud-technology-reshaping-retail-and-restaurants/)
- Pricing: [Toast Pricing](https://pos.toasttab.com/pricing) · [Restaurant Velocity: Toast vs Square vs Lightspeed vs Clover vs TouchBistro vs Revel](https://restaurantvelocity.com/blog/best-restaurant-pos-systems/) · [Beancount.io: Toast vs Square vs Clover 2026](https://beancount.io/blog/2026/07/10/toast-square-clover-pos-system-guide) · [ECOSIRE: Odoo POS vs Square/Toast/Clover/Lightspeed 2026](https://ecosire.com/blog/odoo-pos-vs-square-toast-clover-lightspeed-2026)
- Market data: [IHL Group: USA POS Terminal Market 2026](https://www.ihlservices.com/news/analyst-corner/2026/03/usa-pos-terminal-market-2026/) · [Straits Research: Cloud POS Market](https://straitsresearch.com/press-release/global-cloud-pos-market-trends) · [LocalExpress: Grocery POS Statistics 2026](https://www.localexpress.io/post/grocery-pos-system-integration-statistics) · [Business Research Insights: Grocery POS Market](https://www.businessresearchinsights.com/market-reports/grocery-pos-systems-market-116654)
- Accounting books, errors and inventory accounting: [Achievable: Books of prime entry](https://achievable.me/define/books-of-prime-entry/) · [Thinka: Books of original entry and types of ledgers](https://www.thinka.ai/en-US/Senior-Secondary-HKDSE/Business-Accounting-and-Financial-Studies/Books-of-Original-Entry-and-Types-of-Ledgers) · [AdminAdvice: Bookkeeping journals and ledgers](https://adminadvice.com/bookkeeping-journals-and-ledgers) · [GeeksforGeeks: Types of errors in trial balance](https://www.geeksforgeeks.org/accountancy/types-of-errors-in-trial-balance/) · [InTime: Errors not revealed by a trial balance](https://intimeaccounting.com/blog/trial-balance/errors-not-revealed) · [WallStreetMojo: Trial balance errors](https://www.wallstreetmojo.com/trial-balance-errors/) · [NetSuite: Perpetual inventory](https://www.netsuite.com/portal/resource/articles/inventory-management/what-is-perpetual-inventory.shtml) · [AccountingCoach: Inventory and COGS](https://www.accountingcoach.com/inventory-and-cost-of-goods-sold/explanation) · [Cleverence: COGS in a perpetual inventory system](https://www.cleverence.com/articles/for-business/cost-of-goods-sold-perpetual-inventory-system-4729/)
- Payments and security: [Bluefin: What is PCI DSS 4.0](https://www.bluefin.com/bluefin-news/what-is-pci-dss-4-0/) · [PCI SSC: SAQ P2PE v4.0](https://listings.pcisecuritystandards.org/documents/PCI-DSS-v4-0-SAQ-P2PE.pdf) · [Finix: In-Person Payments Guide](https://finix.com/resources/blogs/complete-guide-inpersonpaymentsprocessing-2025) · [EazyPay Tech: SoftPOS and PCI MPoC](https://eazypaytech.com/softpos-and-pci-mpoc-certification-demystified/)
