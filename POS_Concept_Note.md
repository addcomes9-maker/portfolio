# Concept Note: A Unified Cloud POS for Supermarkets, Bars and Resto-Bars

| | |
|---|---|
| **Document type** | Concept note (draft for discussion) |
| **Version** | 0.2: adds process-by-process feature design and the positioning over existing POS and ERP systems |
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
7. Solution architecture and core modules
8. Hardware concept
9. Payments, security and compliance
10. Integrations and ERP connectors
11. Key reports and KPIs
12. Implementation approach
13. Commercial model
14. Indicative cost components
15. Risks and mitigation
16. Success measures
17. Conclusion and next steps
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

The platform has a shared core (sales, payments, inventory, customers, staff, reporting) and **three vertical modes**: Supermarket, Bar and Resto-bar. A business switches on one or more. Section 6 designs every major business process in detail: how it is done manually today, what existing POS and ERP systems provide, where the gaps are, and the specific features this platform will provide.

**Expected benefits** (targets to validate in the pilot):

- Checkout / service speed up 20–30% through speed screens, handhelds, scale integration and self-service.
- Inventory shrink and liquor variance reduced by tying every sale to stock depletion and surfacing variance daily.
- Stock-outs and perishable waste reduced through AI-assisted ordering, expiry tracking and automatic markdowns.
- No double entry between POS and ERP: one item master, automatic posting of sales, costs and cash to accounts.
- Owners get real-time, multi-location visibility from a phone.
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
7. Kitchen and bar display (for example, Toast KDS, Lightspeed KDS, Odoo preparation display).
8. Self-ordering by QR code and kiosk.
9. Real-time dashboards on mobile; multi-location roll-ups.
10. Open APIs and marketplaces for accounting, delivery, e-commerce, payroll and reservations.

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
| D5 | **Offline-first with safe sync.** Every terminal works without internet; conflicts are resolved by clear rules (Section 7). | G4 |
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

- **Connectors, not custom code:** prebuilt connectors for the most common ERPs and POS systems (Section 10), plus a generic API, webhook and CSV/SFTP route for everything else.
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
  - Full offline operation (Section 7).

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

### 6.4 Shared back-office and ERP processes

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

## 7. Solution architecture and core modules

### 7.1 Architecture overview

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

### 7.2 Design principles

- **Cloud-native, offline-first.** Every terminal holds a local copy of the catalogue, prices, promotions and open tabs. Sales continue without internet, and offline card payments are stored and forwarded within configurable risk limits.
- **Offline conflict rules.** Sales recorded offline are never discarded. Price or promotion changes made centrally apply from the time they reach the terminal. Stock is corrected by adjustment movements, never overwritten. Tabs are locked to one terminal at a time, or merged with an audit record.
- **Modular.** A shared core plus vertical modes (Supermarket, Bar, Resto-bar) switched on per location.
- **Hardware-agnostic where possible.** Runs on Android and iPadOS tablets, Windows lane PCs, and certified payment terminals, to avoid lock-in.
- **API-first.** Documented REST/webhook APIs; the integration hub (Section 5.4) is part of the product, not a project add-on.
- **Secure by design.** Card data never touches the POS application (P2PE / tokenisation), role-based access control, full audit trail, encryption in transit and at rest.

### 7.3 Core modules (shared by all modes)

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

### 7.4 AI and advanced features (phase 2+)

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

## 8. Hardware concept

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
| Self-checkout / kiosk | ✔ | – | optional |
| Electronic shelf labels | ✔ | – | – |
| ID scanner | alcohol/tobacco | ✔ | ✔ |
| Bluetooth bottle scale / smart spouts / keg meters | – | optional | optional |
| Label printer (reduced-price, prep, shelf) | ✔ | – | ✔ prep labels |
| Local edge server / UPS | ✔ larger stores | optional | optional |

**Principles:** use commercially available, supported devices; spill- and heat-resistant screens for bars and kitchens; rugged handhelds; keep a spare-device pool; use a UPS for lanes and network gear.

---

## 9. Payments, security and compliance

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

## 10. Integrations and ERP connectors

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

## 11. Key reports and KPIs

**Supermarket:** sales per hour and per lane; average basket value and size; items per minute per cashier; gross margin by category; shrink % (known and unknown); waste and markdown value; stock turn; out-of-stock rate; supplier fill rate; promotion uplift and margin; self-checkout usage and intervention rate.

**Bar:** pour cost % (overall and by category); liquor variance % and value by product, station and bartender; sales per bartender per hour; average check; walk-out losses; comp/spill value; happy-hour lift; keg yield; top sellers and dead stock.

**Resto-bar:** covers, table turn time, average spend per cover; ticket times by station; food cost % and beverage cost %; prime cost (COGS + labour); sales by channel (dine-in, takeaway, delivery, QR); delivery commission; menu-engineering matrix; labour % of sales; guest feedback score.

**All:** daily sales versus forecast and budget; payment mix and processing fees; voids, refunds and discounts by staff; cash over/short; sync health (ERP postings succeeded or failed); customer retention and loyalty redemption.

---

## 12. Implementation approach

| Phase | Duration (indicative) | Key activities | Output |
|---|---|---|---|
| **0. Discovery** | 3–4 weeks | Site visits; **process walk-throughs against Section 6** for each service; existing POS/ERP inventory and data audit; legal/tax/fiscal requirements; choose the deployment mode | Requirements, fit-gap and integration design |
| **1. Design & build / configure** | 8–12 weeks | Core platform and selected modes; ERP/POS connectors and mappings; payments, scales, ESL; data migration (items, recipes, customers, stock) | Configured system in test |
| **2. Pilot** | 6–8 weeks | One supermarket, one bar and one resto-bar (or those available); training; parallel run with the existing system; KPI comparison against baseline; ERP posting validation | Pilot report, go/no-go |
| **3. Rollout** | Phased per site | Hardware install, train-the-trainer, on-site go-live support for the first week | Live sites |
| **4. Optimise** | Ongoing | AI features, advanced analytics, loyalty, more connectors; quarterly reviews | Continuous improvement |

For customers starting in **Mode 3**, Phase 1 is shorter (connectors and dashboards only), and the pilot focuses on variance, forecasting and loss-prevention insight before any tills are replaced.

**Change management and training**

- Role-based training: cashier, bartender, server, supervisor, receiver/buyer, manager, accountant, owner.
- Short in-app guides and videos; a sandbox "training mode" on every terminal.
- Super-users at each site; two weeks of intensive support after go-live.

---

## 13. Commercial model (options)

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

## 14. Indicative cost components (for budgeting)

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

## 15. Risks and mitigation

| Risk | Impact | Mitigation |
|---|---|---|
| Internet outage | Lost sales | Offline-first terminals, local edge server, 4G/5G failover, store-and-forward payments |
| Poor data quality (SKUs, recipes) | Wrong stock and variance figures | Data cleansing in discovery; GTIN standards; recipe audit with the bar manager and chef |
| Integration failures with the customer's ERP/POS | Missing or duplicate postings | Idempotent messages, sync monitoring, error queue, reconciliation reports, pilot validation of postings |
| Existing POS vendor limits API access (Mode 3) | Incomplete data | Use official APIs/exports first; negotiate access; fall back to Mode 2 for affected sites |
| Staff resistance | Slow adoption, workarounds | Simple UI, training mode, super-users, involving staff in the pilot |
| Scope creep (trying to match every ERP feature) | Delays and cost | Process-based scope from Section 6; ERP remains the system of record where it is strong (Mode 2) |
| Payment security breach | Financial and reputational loss | P2PE/tokenisation, PCI DSS v4.0 controls, MFA, monitoring |
| Vendor/processor lock-in | Higher long-term costs | Open APIs, data export, processor-agnostic design |
| Regulatory change (tax, fiscalisation) | Penalties | Pluggable fiscal connectors; monitoring of local regulation |
| Hardware failure in harsh environments | Downtime at bar or kitchen | Rugged/spill-proof devices, spares, device management |
| Cost overrun | Budget pressure | Phased rollout, pilot-gated investment, clear scope |

---

## 16. Success measures (pilot evaluation)

| Area | Measure | Baseline → Target |
|---|---|---|
| Speed | Avg. checkout time (grocery); order-to-payment time (bar); ticket time (kitchen) | Measure in discovery → −20% |
| Loss | Liquor variance %; grocery shrink %; walk-outs; cash over/short | Measure → variance under 5%, walk-outs ≈ 0 |
| Waste | Perishable and kitchen waste value | Measure → −15% |
| Availability | Out-of-stock incidents | Measure → −25% |
| Integration | ERP postings completed without manual correction | → ≥ 99% |
| Admin time | Hours per week on manual entry, reconciliation and reports | Measure → −50% |
| Uptime | Sales lost to downtime | → 0 |
| Satisfaction | Staff ease-of-use score; customer satisfaction/NPS | → ≥ 8/10 |
| Financial | Gross margin; labour %; processing cost % | Improvement vs baseline |

---

## 17. Conclusion and next steps

The platform combines what cloud POS products do best (speed, simplicity, payments, mobility) with what ERP systems do best (controls, purchasing, accounting, audit). It adds deep supermarket, bar and resto-bar workflows designed from how each process is actually run today. Its three deployment modes let a business adopt it at its own pace: as an intelligence layer over current systems, as a POS front end to an existing ERP, or as a complete platform.

**Proposed next steps**

1. Confirm the target businesses: number of sites, lanes and terminals, country or countries of operation, and **the POS and ERP systems they use today**.
2. Validate Section 6 with operators: walk through each process with a store manager, bar manager and chef, and rank features as *must*, *should* or *later*.
3. Decide the delivery route: build a custom platform, build on an open-source base (for example, Odoo or ERPNext), or extend an existing product. A short build-versus-buy analysis is recommended.
4. Choose the first deployment mode for the pilot and the priority connectors.
5. Shortlist hardware and payment partners; approve pilot budget and timeline.

---

## Appendix A: Feature matrix by mode

| Feature | Core | Supermarket | Bar | Resto-bar |
|---|:---:|:---:|:---:|:---:|
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
| KDS, coursing, expo | | ○ (deli) | | ● |
| Allergen management | | ○ | | ● |
| QR table ordering & pay-at-table | | | ○ | ● |
| Reservations & waitlist | | | ○ | ● |
| Delivery aggregator integration | | ○ | | ● |
| Recipe costing & menu engineering | | ○ | ● | ● |
| AI forecasting, anomaly detection | ○ | ● | ● | ● |
| Intelligence layer over existing POS (Mode 3) | ○ | ○ | ○ | ○ |

● = included / core to the mode  ○ = optional add-on

## Appendix B: Glossary

- **BDS / KDS:** Bar / Kitchen Display System; screens that replace paper order tickets.
- **BoM:** Bill of materials; the list of ingredients or components in a product (a recipe, in hospitality).
- **DSD:** Direct store delivery; suppliers who deliver straight to the store rather than via a warehouse.
- **DSR:** Daily sales report.
- **E2EE / P2PE:** End-to-end / point-to-point encryption; card data is encrypted inside the payment terminal.
- **EMV:** The global chip-card payment standard.
- **ESL:** Electronic shelf label; a digital price tag updated from the POS.
- **FEFO / FIFO:** First-expired-first-out / first-in-first-out stock rotation.
- **GRN:** Goods received note.
- **GTIN:** Global Trade Item Number; the number behind a product barcode.
- **PLU:** Price look-up code, used for produce and weighted items.
- **PMS:** Hotel property management system.
- **Pour cost:** Cost of the liquor in a drink ÷ its selling price.
- **Pre-authorisation:** A temporary hold on a card when a tab is opened.
- **SoftPOS:** Software that accepts contactless cards on an ordinary NFC phone or tablet.
- **System of record:** The system whose data is treated as the official version of a given object.
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
- Restaurant / resto-bar POS: [EHL Insights: Restaurant Technology 2026](https://insights.ehl.edu/restaurant-technology) · [Expert Market: Best Restaurant POS 2026](https://www.expertmarket.com/pos/best-pos-restaurants) · [RestroScout: Toast vs Square vs Lightspeed](https://restroscout.com/best-restaurant-pos-systems) · [Guideflow: Kitchen Display Systems 2026](https://www.guideflow.com/blog/kitchen-display-system) · [LithosPOS: POS Trends 2026](https://lithospos.com/blog/pos-system-trends-2026-ai-voice-ordering-cloud-technology-reshaping-retail-and-restaurants/)
- Pricing: [Toast Pricing](https://pos.toasttab.com/pricing) · [Restaurant Velocity: Toast vs Square vs Lightspeed vs Clover vs TouchBistro vs Revel](https://restaurantvelocity.com/blog/best-restaurant-pos-systems/) · [Beancount.io: Toast vs Square vs Clover 2026](https://beancount.io/blog/2026/07/10/toast-square-clover-pos-system-guide) · [ECOSIRE: Odoo POS vs Square/Toast/Clover/Lightspeed 2026](https://ecosire.com/blog/odoo-pos-vs-square-toast-clover-lightspeed-2026)
- Market data: [IHL Group: USA POS Terminal Market 2026](https://www.ihlservices.com/news/analyst-corner/2026/03/usa-pos-terminal-market-2026/) · [Straits Research: Cloud POS Market](https://straitsresearch.com/press-release/global-cloud-pos-market-trends) · [LocalExpress: Grocery POS Statistics 2026](https://www.localexpress.io/post/grocery-pos-system-integration-statistics) · [Business Research Insights: Grocery POS Market](https://www.businessresearchinsights.com/market-reports/grocery-pos-systems-market-116654)
- Payments and security: [Bluefin: What is PCI DSS 4.0](https://www.bluefin.com/bluefin-news/what-is-pci-dss-4-0/) · [PCI SSC: SAQ P2PE v4.0](https://listings.pcisecuritystandards.org/documents/PCI-DSS-v4-0-SAQ-P2PE.pdf) · [Finix: In-Person Payments Guide](https://finix.com/resources/blogs/complete-guide-inpersonpaymentsprocessing-2025) · [EazyPay Tech: SoftPOS and PCI MPoC](https://eazypaytech.com/softpos-and-pci-mpoc-certification-demystified/)
