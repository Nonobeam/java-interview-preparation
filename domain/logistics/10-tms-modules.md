## D10: What does a logistics TMS (Transportation Management System) manage? What are its key modules?

**Why asked:** CLT builds and maintains logistics software. Understanding TMS architecture helps you contribute effectively from day one.

**Answer:**

A **TMS (Transportation Management System)** is the core platform for managing freight forwarding and container shipping operations.

### Key Modules

**1. Booking Management**
- Create and confirm bookings with carriers
- Handle amendments (cargo, container, dates) and cancellations
- Space allocation per vessel/voyage
- Integration with carrier booking APIs (INTTRA, CargoSmart, DCSA APIs)

**2. Documentation Management**
- Bill of Lading creation, draft, approval, and amendment workflow
- Shipping Instructions submission to carriers
- Document archival and sharing (PDF generation, email dispatch)
- Template management with auto-fill from shipment data
- AMS/ENS/ISF electronic filing

**3. Container Tracking & Visibility**
- Real-time container and vessel tracking
- Milestone event recording: gate-in, loaded on vessel, departed, transshipped, arrived, discharged, customs released, gate-out, empty returned
- Exception alerts (e.g., customs hold, vessel delay)
- Predictive ETA based on AIS + historical data
- Integration with carrier tracking APIs

**4. Rate & Tariff Management**
- Maintain carrier contract rates (NAC) with validity periods
- Spot rate management
- Surcharge table management (THC, BAF, PSS, etc.)
- Rate comparison across carriers for a given lane
- Auto-rate calculation for shipments
- Rate expiry alerts

**5. Quotation Management**
- Customer quotation creation from rate tables
- Profit/margin calculation
- Quotation approval workflow
- Quote-to-booking conversion
- Quote version history

**6. Customs & Compliance**
- Customs declaration preparation (import + export)
- HS code classification assistance
- EDI/API integration with customs systems
- Denied party screening (OFAC, EU sanctions)
- AMS/ENS/ISF filing

**7. Financial / Billing Module**
- Invoice generation to customers
- Vendor cost capture (carrier invoices, THC, trucking)
- Revenue vs. cost reconciliation per shipment
- P&L per shipment, per customer, per trade lane
- Accounts receivable / payable
- Credit limit management
- ERP integration (SAP, Oracle)

**8. D&D (Demurrage & Detention) Tracking**
- Record carrier-specific free time rules per trade lane
- Auto-calculate when free time expires per container
- Accrue daily D&D charges
- Alert before free time expires
- D&D dispute management

**9. Reporting & Analytics**
- Shipment volume by customer / lane / period
- Revenue and profitability reports
- Carrier performance scorecards (on-time %, cost)
- D&D exposure dashboard
- KPI dashboards for operations team

**10. Inland / Intermodal**
- Trucking order management and dispatch
- Pre-carriage and on-carriage coordination
- Driver and vehicle assignment
- Gate appointment scheduling at terminals

**11. Warehouse / CFS Management (WMS)**
- LCL consolidation and deconsolidation tracking
- CFS inventory per shipment
- Stuffing/destuffing planning

### Industry Platforms
| System | Type | Notes |
|--------|------|-------|
| **CargoWise** | Full TMS for freight forwarders | WiseTech Global. Market leader. |
| **Navis N4** | Terminal Operating System (TOS) | Manages port terminal operations (yard, cranes, gates) |
| **INTTRA / E2open** | Carrier connectivity platform | Booking and documentation exchange between forwarders and carriers |
| **Descartes** | Customs + routing | Strong in customs filing and regulatory compliance |

### Integration Standards
- **EDI / EDIFACT**: Legacy B2B messaging still widely used. Key messages: IFTMIN (booking), BAPLIE (stowage plan), CUSCAR (cargo manifest)
- **REST APIs / JSON**: Modern approach. DCSA is defining open API standards for the container shipping industry
- **ERP Integration**: SAP / Oracle for finance
