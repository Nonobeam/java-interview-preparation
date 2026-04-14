# Container Shipping Logistics Domain Knowledge

## 1. Core Business Concepts

### What is Container Shipping?

Container shipping is the transportation of goods in standardized intermodal containers via ocean vessels. Containerization revolutionized global trade after Malcolm McLean introduced it in 1956. Before containers, cargo was loaded piece-by-piece (break-bulk), which was slow, expensive, and prone to theft/damage. Today, approximately 90% of global trade by volume moves via container ships.

The container shipping industry operates on a hub-and-spoke model where large mother vessels call at major hub ports, and smaller feeder vessels distribute containers to regional ports.

### Key Players

**Shipper (Exporter)**
- The party that sends/exports the goods. They are the origin of the supply chain.
- Responsible for preparing goods, packaging, and often arranging initial transport to the port.
- Named on the Bill of Lading as the party shipping the goods.

**Consignee (Importer)**
- The party that receives the goods at destination.
- Named on the Bill of Lading as the party to whom goods are delivered.
- Responsible for import customs clearance and taking delivery of goods.

**Freight Forwarder**
- An intermediary that arranges transportation on behalf of shippers/consignees.
- Does NOT own vessels or transport assets — they coordinate logistics.
- Provides door-to-door service by combining multiple carriers (ocean, rail, truck).
- Handles documentation, customs brokerage, cargo insurance, warehousing.
- Examples: Kuehne+Nagel, DHL Global Forwarding, DB Schenker, Expeditors.

**Carrier (Shipping Line / Ocean Carrier)**
- The company that owns or operates the vessel that physically transports containers.
- Issues the Master Bill of Lading (MBL).
- Sets sailing schedules, routes, and base freight rates.
- Major carriers: Maersk, MSC, CMA CGM, COSCO, Hapag-Lloyd, Evergreen, ONE, Yang Ming, HMM, ZIM.
- Carriers form alliances to share vessel capacity:
  - **2M Alliance**: Maersk + MSC (ending 2025)
  - **Ocean Alliance**: CMA CGM, COSCO, Evergreen, OOCL
  - **THE Alliance**: Hapag-Lloyd, ONE, Yang Ming, HMM

**NVOCC (Non-Vessel Operating Common Carrier)**
- Acts as a carrier to shippers but does NOT own vessels.
- Books space on actual carriers and issues their own House Bill of Lading (HBL).
- Consolidates LCL (Less than Container Load) shipments from multiple shippers into one FCL container.
- The key difference from a freight forwarder: NVOCC issues their own B/L and takes carrier liability; a freight forwarder acts as an agent.

**Customs Broker**
- Licensed professional who handles customs clearance on behalf of importers/exporters.
- Prepares and submits customs declarations.
- Classifies goods under HS (Harmonized System) codes for tariff determination.
- Ensures compliance with import/export regulations.
- Calculates duties, taxes, and fees.

**Terminal Operator**
- Manages the port terminal where containers are loaded/discharged from vessels.
- Operates cranes, yard equipment, and gate operations.
- Major terminal operators: PSA International, Hutchison Ports, DP World, APM Terminals, COSCO Shipping Ports.

**Trucker / Haulier**
- Provides inland transportation between the port and shipper/consignee premises.
- Pre-carriage: transport from shipper to port of loading.
- On-carriage: transport from port of discharge to consignee.

**CFS Operator (Container Freight Station)**
- Operates a warehouse where LCL cargo is consolidated (stuffed) or deconsolidated (destuffed).
- Receives loose cargo from multiple shippers and packs into containers, or vice versa.

### Incoterms (International Commercial Terms)

Incoterms are standardized trade terms published by the International Chamber of Commerce (ICC). They define responsibilities between buyer and seller regarding cost, risk, and insurance. Current version: Incoterms 2020.

**Group E — Departure**

| Term | Name | Description |
|------|------|-------------|
| EXW | Ex Works | Seller makes goods available at their premises. Buyer bears all costs and risks from that point. Minimum obligation for seller. |

**Group F — Main Carriage Unpaid (by seller)**

| Term | Name | Description |
|------|------|-------------|
| FCA | Free Carrier | Seller delivers goods to carrier at a named place. Risk transfers when goods are handed to the carrier. |
| FAS | Free Alongside Ship | Seller delivers goods alongside the vessel at the port of loading. Used mainly for bulk cargo. |
| FOB | Free On Board | Seller delivers goods on board the vessel at the port of loading. Risk transfers when goods pass the ship's rail. Very commonly used for ocean freight. |

**Group C — Main Carriage Paid (by seller)**

| Term | Name | Description |
|------|------|-------------|
| CFR | Cost and Freight | Seller pays freight to the port of destination but risk transfers at the port of loading (when goods are on board). |
| CIF | Cost, Insurance, and Freight | Same as CFR but seller also arranges and pays for marine cargo insurance. Most common Incoterm in international trade. |
| CPT | Carriage Paid To | Seller pays freight to named destination. Risk transfers when goods are handed to the first carrier. For any mode of transport. |
| CIP | Carriage and Insurance Paid To | Same as CPT but seller also arranges insurance. For any mode of transport. |

**Group D — Arrival**

| Term | Name | Description |
|------|------|-------------|
| DAP | Delivered at Place | Seller delivers goods to a named destination, ready for unloading. Seller bears all risks until destination. |
| DPU | Delivered at Place Unloaded | Seller delivers and unloads goods at the named destination. Only Incoterm requiring seller to unload. |
| DDP | Delivered Duty Paid | Seller bears all costs and risks including import customs clearance and duties. Maximum obligation for seller. |

**Key interview point**: FOB and CIF are the most commonly used Incoterms in container shipping. Under FOB, the buyer arranges and pays for ocean freight. Under CIF, the seller arranges and pays for ocean freight plus insurance.

---

## 2. Container Types

### Standard Dry Containers

| Type | External Dimensions (L x W x H) | Internal Volume | Max Payload | TEU |
|------|----------------------------------|-----------------|-------------|-----|
| 20ft (20'GP) | 20' x 8' x 8'6" | ~33 CBM | ~28,000 kg | 1 TEU |
| 40ft (40'GP) | 40' x 8' x 8'6" | ~67 CBM | ~28,500 kg | 2 TEU |
| 40ft High Cube (40'HC) | 40' x 8' x 9'6" | ~76 CBM | ~28,500 kg | 2 TEU |

- **20ft GP**: Standard for heavy cargo (steel, machinery) since weight limit is reached before volume.
- **40ft GP**: Standard for general cargo.
- **40ft HC**: Extra 1 foot of height. Most popular container type globally. Used for voluminous/lightweight cargo (furniture, electronics, garments).
- **45ft HC**: Used mainly in North American and intra-Asian trades.

### Specialized Containers

**Reefer (Refrigerated) Container**
- Temperature-controlled container with built-in refrigeration unit.
- Temperature range: -35C to +30C (some models to -60C for pharma).
- Requires electrical power supply (genset or vessel/terminal reefer plugs).
- Used for: perishable food (fruits, vegetables, meat, seafood), pharmaceuticals, chemicals.
- Available in 20ft and 40ft HC sizes.
- Reefer monitoring is critical — temperature logs must be maintained.

**Open Top Container**
- Standard container without a solid roof — covered by a removable tarpaulin.
- Allows top-loading of oversized cargo via crane.
- Used for: heavy machinery, timber, marble slabs, large industrial equipment.
- Available in 20ft and 40ft.

**Flat Rack Container**
- Has no side walls or roof — only a floor with collapsible or fixed end walls.
- Used for oversized/over-dimensional cargo (OOG — Out of Gauge).
- Common for: heavy machinery, vehicles, boats, industrial equipment, pipes.
- Available in 20ft and 40ft.
- Cargo may extend beyond container dimensions (called "overstow" or "over-height/over-width").

**Tank Container (ISO Tank)**
- Cylindrical tank inside a standard container frame.
- Used for liquid bulk cargo: chemicals, food-grade liquids (wine, juice), hazardous liquids.
- Capacity: typically 21,000-26,000 liters.
- Must comply with IMO regulations for dangerous goods if applicable.

**Other Types**
- **Ventilated Container**: Has passive or active ventilation for cargo like coffee, cocoa, onions.
- **Insulated Container**: Thermal insulation without active cooling. For cargo needing protection from temperature extremes.
- **Garment Container (GOH)**: Fitted with hanging rails for garments on hangers.
- **Bulk Container**: Has roof hatches for top-loading bulk cargo (grain, powder).

---

## 3. Shipping Process Flow

### End-to-End Container Shipping Process

```
Booking -> Stuffing/Loading -> Documentation -> Export Customs ->
Port Operations (Origin) -> Vessel Loading -> Ocean Transit ->
(Transshipment if applicable) -> Vessel Discharge ->
Port Operations (Destination) -> Import Customs Clearance ->
Delivery to Consignee
```

### Detailed Steps

**Step 1: Booking**
- Shipper or freight forwarder requests a booking with the carrier.
- Booking details include: origin, destination, commodity, container type/quantity, preferred sailing date, cargo weight/volume.
- Carrier confirms booking and issues a **Booking Confirmation Number**.
- Carrier assigns a specific vessel and voyage number.
- Shipper receives empty container pickup instructions (container release reference).

**Step 2: Empty Container Pickup**
- Shipper or trucker picks up empty container(s) from the carrier's designated **container depot** or **container yard (CY)**.
- An **Equipment Interchange Receipt (EIR)** is issued documenting container condition at pickup.
- Container number, seal number, and condition are recorded.

**Step 3: Stuffing / Loading**
- Cargo is loaded (stuffed) into the container at the shipper's warehouse/factory or at a CFS.
- **FCL (Full Container Load)**: Shipper stuffs the entire container. Sealed by the shipper.
- **LCL (Less than Container Load)**: Multiple shippers' cargo consolidated at a CFS. CFS operator stuffs and seals.
- Container is sealed with a unique **seal number** which must match documentation.
- Verified Gross Mass (VGM) is determined (SOLAS regulation, mandatory since 2016):
  - **Method 1**: Weigh the packed container.
  - **Method 2**: Weigh all cargo items + dunnage + tare weight of container.

**Step 4: Pre-carriage / Inland Transport (Origin)**
- Loaded container transported from shipper's premises to the port of loading.
- Modes: truck, rail, or barge.
- Trucking company receives a **container haulage order**.

**Step 5: Documentation Preparation**
- Shipper prepares: Commercial Invoice, Packing List, Shipping Instructions (SI).
- Freight forwarder/NVOCC submits Shipping Instructions to carrier.
- Carrier prepares draft Bill of Lading based on SI.
- Shipper reviews and approves the draft B/L.
- Export customs declaration is prepared and submitted.

**Step 6: Export Customs Clearance**
- Customs broker submits export declaration to customs authority.
- Customs may inspect the cargo (physical inspection or documentary check).
- Customs releases the cargo for export — **customs release** is obtained.

**Step 7: Gate-In at Origin Terminal**
- Container arrives at the port terminal via truck.
- Terminal gate records: container number, seal number, truck details, booking reference.
- Container is inspected (visual check, seal integrity).
- Terminal issues gate-in receipt.
- Container is placed in the **container yard (CY)** by yard equipment (RTG, straddle carrier).

**Step 8: Vessel Loading**
- Terminal plans the **stowage plan** (bay plan) — determining where each container goes on the vessel.
- Factors: weight distribution, destination port (discharge sequence), hazardous cargo segregation, reefer plug availability, over-dimensional cargo placement.
- Quay cranes (STS — Ship to Shore) load containers onto the vessel.
- The vessel's chief officer approves the stowage plan.
- Loading sequence follows the bay plan from bottom to top, fore to aft.

**Step 9: Vessel Departure (ETD)**
- All containers loaded, cargo manifest finalized.
- Carrier issues the final Bill of Lading.
- Vessel departs the port of loading (POL).
- **Advance Manifest** (AMS for US, ENS for EU, AFR for Japan) must be submitted to destination country customs before departure or within required timeframes.

**Step 10: Ocean Transit**
- Vessel sails along the scheduled route (trade lane).
- Voyage duration varies: Asia to US West Coast ~12-15 days, Asia to Europe ~25-35 days, Asia to US East Coast ~30-40 days.
- Vessel position tracked via AIS (Automatic Identification System).
- In-transit events may include: weather delays, port omission, vessel breakdown, piracy risk zones.

**Step 11: Transshipment (if applicable)**
- If no direct sailing, container is discharged at a **hub port** and loaded onto another vessel.
- Example: Container from Ho Chi Minh City to Rotterdam may transship at Singapore.
- Container waits in the transshipment port's CY until the connecting vessel arrives.
- Transshipment adds time and cost but allows carriers to serve more routes.

**Step 12: Vessel Arrival & Discharge (ETA/ATA)**
- Vessel arrives at the port of discharge (POD).
- Quay cranes discharge containers from the vessel.
- Containers placed in the terminal's CY.
- Terminal sends an **arrival notice** to the consignee/freight forwarder.

**Step 13: Import Customs Clearance**
- Customs broker submits import declaration with all required documents.
- Customs authority assesses duties and taxes based on HS code classification and customs value.
- Cargo may be selected for: documentary check, physical inspection, or green channel (automatic clearance).
- Duties and taxes paid. Customs releases the cargo.

**Step 14: Delivery Order & Container Release**
- Consignee or their agent presents the original B/L (or telex release/sea waybill) to the carrier's local agent.
- Carrier verifies all freight charges are paid.
- Carrier issues a **Delivery Order (D/O)** authorizing the terminal to release the container.

**Step 15: Gate-Out at Destination Terminal**
- Trucker presents the D/O at the terminal gate.
- Terminal releases the container.
- EIR (Equipment Interchange Receipt) issued at gate-out.

**Step 16: Destuffing & Delivery**
- **FCL**: Container delivered to consignee's warehouse. Consignee destuffs the container.
- **LCL**: Container goes to a CFS. CFS operator destuffs and separates individual consignments.
- Consignee inspects the cargo and confirms receipt.

**Step 17: Empty Container Return**
- After destuffing, the empty container must be returned to the carrier's designated depot/yard.
- Must be returned within the **free time** period to avoid **detention** charges.
- EIR issued at return documenting container condition.

---

## 4. Key Documents

### Bill of Lading (B/L)

The most critical document in container shipping. It serves three functions:

1. **Evidence of contract of carriage** between shipper and carrier.
2. **Receipt for goods** — confirms the carrier received the described cargo.
3. **Document of title** — the holder of the original B/L has title to the goods.

**Types of B/L:**

| Type | Description |
|------|-------------|
| **Master Bill of Lading (MBL)** | Issued by the ocean carrier (shipping line). Covers the actual vessel transport. |
| **House Bill of Lading (HBL)** | Issued by a freight forwarder or NVOCC to their customer (shipper). The carrier sees only the MBL. |
| **Original B/L** | Printed on carrier's paper, issued in sets of 3 originals. Must be physically surrendered for cargo release. |
| **Telex Release / Express Release** | Carrier sends electronic message to destination agent to release cargo without original B/L. Faster and cheaper. |
| **Surrendered B/L** | Original B/L returned to carrier at origin. "SURRENDERED" stamped on it. Cargo released at destination without original. |
| **Clean B/L** | No notation of defective cargo condition. Required for letter of credit transactions. |
| **Claused / Dirty B/L** | Contains remarks about damaged or defective cargo condition. |
| **Straight B/L** | Non-negotiable. Named consignee only — cannot be transferred by endorsement. |
| **Order B/L** | Negotiable. Made out "to order" or "to order of [bank]". Can be transferred by endorsement. Used in letter of credit transactions. |
| **Through B/L** | Covers multimodal transport (e.g., truck + ocean + rail). |
| **Combined Transport B/L** | Similar to Through B/L, covers door-to-door movement. |

**Key B/L fields**: Shipper, Consignee, Notify Party, Vessel/Voyage, POL, POD, Container No., Seal No., Description of Goods, Gross Weight, Measurement (CBM), Number of Packages, Freight (Prepaid/Collect), Place of Receipt, Place of Delivery, Date of Issue.

### Sea Waybill

- Non-negotiable transport document.
- NOT a document of title (unlike B/L).
- Cargo released to named consignee upon proof of identity — no original documents needed.
- Faster release at destination. Increasingly used for trusted shipper-consignee relationships.
- Cannot be used for letter of credit requiring original B/L.

### Commercial Invoice

- Issued by seller (exporter) to buyer (importer).
- States the value of goods for customs valuation and duty calculation.
- Contains: buyer/seller details, description of goods, quantity, unit price, total value, currency, Incoterms, payment terms.
- Used by customs to determine import duties.

### Packing List

- Detailed list of the contents of each package/container.
- Contains: item description, quantity, weight (net/gross), dimensions, package markings, container number.
- Used by customs, consignee for cargo verification, and for claims in case of shortage.

### Certificate of Origin (CO)

- Certifies the country of manufacture/origin of the goods.
- Required for: preferential tariff rates under Free Trade Agreements (FTA), customs clearance, quota administration.
- Types:
  - **Non-preferential CO**: States the origin but does not grant tariff benefits.
  - **Preferential CO (e.g., Form A, Form D, Form E)**: Grants reduced or zero tariff rates under FTAs.
  - **EUR.1**: Used in EU trade agreements.

### Customs Declaration

- **Export Declaration**: Filed by the exporter's customs broker at the origin country.
- **Import Declaration**: Filed by the importer's customs broker at the destination country.
- Declares: goods description, HS code, customs value, origin, quantity.
- HS Code (Harmonized System): 6-digit international classification code. Countries add 2-4 more digits for national tariff lines.

### Delivery Order (D/O)

- Issued by the carrier (or their agent) to authorize the terminal to release the container to the consignee.
- Issued after the consignee presents the B/L and pays all charges.
- Presented at the terminal gate for container pickup.

### Other Important Documents

| Document | Purpose |
|----------|---------|
| **Shipping Instructions (SI)** | Shipper's instructions to the carrier for preparing the B/L. |
| **Arrival Notice** | Carrier notifies consignee that the vessel has arrived or is arriving. |
| **Cargo Manifest** | Complete list of all cargo on a vessel. Submitted to port and customs authorities. |
| **Dangerous Goods Declaration (DGD)** | Required for hazardous cargo. Includes IMDG class, UN number, proper shipping name. |
| **Phytosanitary Certificate** | Required for plant-based products (wood, agricultural goods). |
| **Fumigation Certificate** | Proof that cargo/container was fumigated (pest control). |
| **Marine Cargo Insurance Policy** | Covers cargo against loss or damage during transit. |
| **Letter of Credit (L/C)** | Bank guarantee of payment. Requires specific documents to be presented. |
| **VGM Declaration** | Verified Gross Mass of the packed container (SOLAS requirement). |

---

## 5. Port Operations

### Terminal Layout

```
                    BERTH (Quayside)
    ========================================
    |  Quay Cranes (STS Cranes)           |
    ========================================
    |                                      |
    |  Transfer Zone (Prime Movers/AGVs)   |
    |                                      |
    ========================================
    |                                      |
    |  Container Yard (CY)                 |
    |  [RTG / RMG / Straddle Carriers]     |
    |  [Container stacks, 4-6 high]        |
    |                                      |
    ========================================
    |  Gate Complex                        |
    |  [Truck entry/exit, inspection]      |
    ========================================
    |                                      |
    |  CFS (Container Freight Station)     |
    |  [LCL cargo consolidation/           |
    |   deconsolidation warehouse]         |
    |                                      |
    ========================================
    |  Reefer Yard (with power plugs)      |
    |  Empty Container Depot               |
    |  Maintenance & Repair (M&R)          |
    ========================================
```

### Container Yard (CY)

- Open-air area where containers are stored before loading or after discharge.
- Containers stacked in **blocks**, **rows**, **tiers** (heights), and **bays**.
- Location coded: e.g., Block A, Row 03, Bay 12, Tier 3 = A-03-12-3.
- Import containers: stored until consignee picks up.
- Export containers: stored until vessel loading.
- Transshipment containers: stored between connecting vessels.

**Yard Equipment:**
- **RTG (Rubber Tyred Gantry Crane)**: Mobile crane that stacks containers. Most common.
- **RMG (Rail Mounted Gantry Crane)**: Fixed crane on rails. Used in automated terminals.
- **Straddle Carrier**: Lifts and transports containers. Common in smaller terminals.
- **Reach Stacker**: Mobile crane on wheels. Used in depots and smaller yards.
- **AGV (Automated Guided Vehicle)**: Driverless vehicles moving containers between quay and yard. Used in automated terminals.

### Container Freight Station (CFS)

- Warehouse facility for handling LCL cargo.
- **Export CFS**: Receives loose cargo from multiple shippers, consolidates into containers.
- **Import CFS**: Receives full containers, destuffs and separates individual consignments for pickup.
- CFS charges apply per CBM or per revenue ton (whichever is greater).

### Berth Allocation

- Terminal assigns a berth (docking position) to each vessel.
- Factors: vessel size (LOA — Length Overall), draft requirements, crane availability, cargo volume, schedule priority.
- Berth window: the scheduled time slot for a vessel to dock.
- Berth productivity measured in: moves per hour (MPH), crane intensity (cranes per vessel).

### Crane Operations

**Quay Cranes (STS — Ship to Shore)**
- Massive gantry cranes straddling the berth.
- Load/discharge containers between vessel and quayside.
- Modern STS cranes handle 25-40 moves per hour per crane.
- Large vessels may have 5-8 cranes working simultaneously.
- Twin-lift cranes can handle two 20ft containers at once.
- Tandem-lift cranes handle two 40ft containers.

**Crane Split / Crane Allocation**
- Terminal decides how many cranes to deploy per vessel based on: container volume, vessel size, berth time available, terminal workload.

### Gate Operations

- **Gate-in (receiving)**: Truck arrives with export container. Security check, document verification, container inspection (damage, seal), VGM check. Container assigned a yard position.
- **Gate-out (delivery)**: Truck arrives to pick up import container. Document verification (D/O), container released, EIR issued.
- Modern gates use: OCR (Optical Character Recognition) for automatic container number reading, RFID for truck identification, radiation scanning for security.

### Vessel Planning

- **Stowage Plan (Bay Plan)**: Diagram showing container placement on the vessel.
- Vessel divided into: **bays** (longitudinal), **rows** (transverse), **tiers** (vertical).
- Numbering convention: Bays numbered fore to aft (01, 03, 05... for 40ft bays; 02, 04, 06... for 20ft bays). Rows numbered from center outward. Tiers: 02, 04, 06... below deck; 82, 84, 86... above deck.
- Key planning considerations:
  - Weight distribution (heavier containers lower and center).
  - Stability (GM — metacentric height).
  - Discharge port sequence (first discharge port on top).
  - Hazardous cargo segregation (IMDG rules).
  - Reefer proximity to power plugs.
  - Over-height/over-width container placement.

---

## 6. Pricing & Charges

### Ocean Freight

- The base cost of transporting a container from port to port.
- Quoted per container (FCL) or per CBM/revenue ton (LCL).
- Varies by: trade lane, season, supply/demand, contract vs. spot rate.
- **Contract Rate (Named Account / NAC)**: Long-term rate (typically quarterly or annual) negotiated between shipper and carrier.
- **Spot Rate**: One-time rate for a single shipment. More volatile.
- **FAK (Freight All Kinds)**: A single rate regardless of commodity type.

### Surcharges

| Charge | Full Name | Description |
|--------|-----------|-------------|
| **THC** | Terminal Handling Charge | Cost of handling the container at the terminal (loading/discharge). Charged at both origin and destination. Typically $100-$300 per container. |
| **BAF** | Bunker Adjustment Factor | Surcharge to cover fuel (bunker) cost fluctuations. Also called **EBS** (Emergency Bunker Surcharge) or **LSS** (Low Sulphur Surcharge) post-IMO 2020. |
| **CAF** | Currency Adjustment Factor | Surcharge to offset exchange rate fluctuations. |
| **PSS** | Peak Season Surcharge | Applied during high-demand periods (typically July-October for Asia-US/Europe trade). |
| **GRI** | General Rate Increase | Carrier-announced rate increase, usually monthly. |
| **WRS** | War Risk Surcharge | Applied for transits through high-risk areas (e.g., Gulf of Aden, Red Sea). |
| **ISPS** | International Ship and Port Facility Security charge | Security-related surcharge post-9/11. |
| **Suez/Panama Canal Surcharge** | Canal transit fees passed to shippers. |
| **Congestion Surcharge** | Applied when a port experiences significant congestion. |
| **Reefer Surcharge** | Additional cost for reefer containers (power supply, monitoring). |
| **OOG Surcharge** | Out of Gauge surcharge for oversized cargo on flat racks / open tops. |
| **IMO 2020 / LSS** | Low Sulphur Surcharge for IMO 2020 low-sulphur fuel compliance. |
| **Inland Haulage Charge (IHC)** | Cost of truck/rail transport between port and shipper/consignee premises. |

### Demurrage, Detention, and Storage

These are critical charges in the industry and a common source of disputes:

**Demurrage**
- Charged by the **carrier** for keeping a full import container inside the terminal beyond the allowed **free time** period.
- Free time: typically 3-7 days after vessel discharge (varies by carrier and port).
- Accrues daily. Rates escalate over time (e.g., $50/day for days 1-5, $100/day for days 6-10, $150/day thereafter).
- Incentivizes quick pickup of containers from the terminal.

**Detention**
- Charged by the **carrier** for keeping the container outside the terminal (at the consignee's warehouse) beyond the allowed free time.
- Applies from gate-out (pickup) to empty return to depot.
- Free time: typically 4-7 days.
- Combined free time or separate free time may apply.

**Storage**
- Charged by the **terminal operator** (not the carrier) for storing containers in the terminal beyond free time.
- Similar to demurrage but paid to a different party.
- Often overlaps with demurrage — shipper may pay both.

**Combined Demurrage & Detention (D&D)**
- Some carriers quote combined free time. E.g., 10 days combined free time means 10 days total from vessel discharge to empty return.

**Interview tip**: D&D disputes are a major pain point in the industry. The FMC (Federal Maritime Commission) in the US has introduced rules to improve transparency and fairness of D&D charges. Software systems that help track and manage D&D free time are highly valued.

### Other Charges

| Charge | Description |
|--------|-------------|
| **Documentation Fee (DOC Fee)** | Carrier's fee for B/L preparation. ~$25-$75. |
| **Seal Fee** | Cost of the container seal. |
| **VGM Fee** | Fee for weighing the container (VGM compliance). |
| **CFS Charges** | Handling fees at the CFS for LCL cargo. Charged per CBM or revenue ton. |
| **Wharfage** | Port authority charge for using port facilities. |
| **Port Dues** | Vessel-related charges paid to the port authority. |
| **Agency Fee** | Carrier's local agent fee for handling documentation/release. |
| **Chassis Fee** | In the US, fee for using a truck chassis to transport containers. |
| **Pre-pull / Dual Transaction Fee** | Terminal charge for moving containers to a staging area before vessel loading. |

---

## 7. Container Tracking & Status

### Common Container Status Codes / Events

| Status Code | Milestone | Description |
|-------------|-----------|-------------|
| **EMPTY PICKUP** | Equipment | Empty container picked up from depot |
| **GATE IN (EXPORT)** | Origin Terminal | Full container received at origin terminal |
| **LOADED ON VESSEL** | Origin Terminal | Container loaded onto the vessel |
| **VESSEL DEPARTURE** | Origin Port | Vessel departed from port of loading |
| **TRANSSHIPMENT DISCHARGE** | Hub Port | Container discharged at transshipment port |
| **TRANSSHIPMENT LOAD** | Hub Port | Container loaded on connecting vessel |
| **VESSEL ARRIVAL** | Destination Port | Vessel arrived at port of discharge |
| **DISCHARGED FROM VESSEL** | Destination Terminal | Container unloaded from vessel |
| **CUSTOMS HOLD** | Destination | Container held by customs for inspection |
| **CUSTOMS RELEASED** | Destination | Customs clearance completed |
| **GATE OUT (IMPORT)** | Destination Terminal | Container released from terminal to trucker |
| **EMPTY RETURN** | Equipment | Empty container returned to depot |
| **RAIL DEPARTURE** | Inland | Container loaded on rail at origin |
| **RAIL ARRIVAL** | Inland | Container arrived at inland rail terminal |

### Vessel Tracking

- **AIS (Automatic Identification System)**: Ships broadcast their position, speed, course, and identity via VHF radio. Tracked by satellite and coastal stations.
- Vessel tracking platforms: MarineTraffic, VesselFinder, FleetMon.
- Carriers provide vessel tracking on their websites.
- Key vessel information: IMO number (unique vessel ID), MMSI number, vessel name, flag, call sign, current position, speed, draft, destination, ETA.

### ETA Management

- **ETA (Estimated Time of Arrival)**: The projected arrival time at port.
- ETA is dynamic — updated throughout the voyage based on: weather, port congestion at upstream ports, vessel speed changes, schedule recovery efforts.
- **ETD (Estimated Time of Departure)**: Projected departure time.
- **ATA (Actual Time of Arrival)**: Confirmed arrival time.
- **ATD (Actual Time of Departure)**: Confirmed departure time.
- **ETB (Estimated Time of Berthing)**: When vessel will actually dock (may differ from ETA if there's a queue).
- **ETC (Estimated Time of Completion)**: When vessel operations will be completed.

ETA accuracy is a major challenge. Carriers and logistics platforms invest heavily in predictive ETA algorithms using historical data, AIS data, weather, and port congestion data.

### Schedule Reliability

- Measured as the percentage of vessels arriving within a defined window (e.g., +/- 1 day of scheduled ETA).
- Industry average schedule reliability: typically 50-70% (it dropped to ~30% during COVID disruptions in 2021-2022).
- Sea-Intelligence and Drewry publish regular schedule reliability reports.

---

## 8. Common Software Systems

### Transportation Management System (TMS)

A TMS is the core software platform for managing logistics operations. In the container shipping context, a TMS typically covers:

**Core Modules:**

1. **Booking Management**
   - Booking creation and confirmation with carriers.
   - Booking amendment and cancellation.
   - Space allocation and capacity management.
   - Integration with carrier booking APIs (INTTRA, CargoSmart).

2. **Documentation Management**
   - Bill of Lading creation and management.
   - Shipping Instructions (SI) preparation and submission.
   - Document templates and compliance checking.
   - Electronic document sharing and archival.

3. **Container Tracking & Visibility**
   - Real-time container and vessel tracking.
   - Milestone event tracking (gate-in, loaded, departed, arrived, etc.).
   - Exception management and alerts.
   - Predictive ETA.
   - Integration with AIS data, carrier tracking APIs.

4. **Rate Management / Tariff Management**
   - Carrier contract rate management.
   - Spot rate quotation.
   - Surcharge management.
   - Rate comparison and carrier selection.
   - Auto-rating of shipments.
   - Validity period management.

5. **Quotation Management**
   - Customer quotation creation.
   - Profit/margin calculation.
   - Quotation approval workflow.
   - Quote-to-booking conversion.

6. **Customs & Compliance**
   - Customs declaration preparation and submission (EDI/API with customs systems).
   - HS code classification.
   - Trade compliance (denied party screening, embargo checks).
   - AMS/ENS/AFR filing.

7. **Financial / Accounting Module**
   - Invoice generation (to customers).
   - Cost management (vendor invoices from carriers, truckers, terminals).
   - Revenue/cost reconciliation.
   - Profit & loss per shipment.
   - Credit management.
   - Accounts receivable / payable.
   - Integration with ERP systems (SAP, Oracle).

8. **Customer Relationship Management (CRM)**
   - Customer master data.
   - Sales pipeline management.
   - Customer interaction history.

9. **Reporting & Analytics**
   - Shipment volume reports.
   - Revenue and profit analysis.
   - Carrier performance scorecards.
   - D&D exposure reporting.
   - KPI dashboards.

10. **Inland / Intermodal Management**
    - Trucking dispatch and management.
    - Rail bookings.
    - Multi-leg shipment planning.
    - Driver/vehicle management.

11. **Warehouse Management (WMS)**
    - CFS operations management.
    - Inventory tracking for LCL cargo.
    - Stuffing/destuffing planning.

### Industry-Specific Platforms

| System | Description |
|--------|-------------|
| **CargoWise** | Leading TMS for freight forwarders. By WiseTech Global. Comprehensive end-to-end platform. |
| **INTTRA** | Now part of E2open. Platform for carrier booking, tracking, documentation. Industry standard for carrier EDI. |
| **CargoSmart** | Container tracking and analytics platform. |
| **IQAX** | Digital platform for container tracking and eBL. |
| **Descartes** | Customs compliance, routing, tracking solutions. |
| **BluJay Solutions** | TMS and supply chain platform (now part of E2open). |
| **Navis (N4)** | Terminal Operating System (TOS) — manages terminal/yard operations. Market leader. |
| **TOPS** | Terminal Operating System by RBS. |
| **TradeLens** | Blockchain-based shipping platform by Maersk/IBM (discontinued 2022, but historically significant). |
| **DCSA** | Digital Container Shipping Association — developing industry standards for digital APIs. |

### Integration Standards

- **EDI (Electronic Data Interchange)**: Traditional B2B messaging format. Common messages: IFTMIN (booking), COPARN (container release), BAPLIE (bay plan), CUSCAR (customs cargo report).
- **API (REST/JSON)**: Modern integration approach. DCSA is defining standard APIs.
- **EDIFACT**: UN standard for EDI messages in international trade.
- **XML**: Used in customs declarations (e.g., ASYCUDA systems).

---

## 9. Industry-Specific Terms / Glossary

### Essential Abbreviations

| Term | Full Form | Definition |
|------|-----------|------------|
| **TEU** | Twenty-foot Equivalent Unit | Standard unit of measurement. 1 x 20ft container = 1 TEU. 1 x 40ft container = 2 TEU. Vessel capacity measured in TEU (e.g., 24,000 TEU mega vessel). |
| **FEU** | Forty-foot Equivalent Unit | 1 x 40ft container = 1 FEU = 2 TEU. |
| **FCL** | Full Container Load | Shipper uses the entire container. Door-to-door seal integrity. |
| **LCL** | Less than Container Load | Cargo from multiple shippers consolidated into one container. Charged per CBM or W/M (weight/measure). |
| **POL** | Port of Loading | The port where cargo is loaded onto the ocean vessel. |
| **POD** | Port of Discharge | The port where cargo is unloaded from the ocean vessel. |
| **ETD** | Estimated Time of Departure | Projected departure time of the vessel from a port. |
| **ETA** | Estimated Time of Arrival | Projected arrival time of the vessel at a port. |
| **ATD** | Actual Time of Departure | Confirmed departure time. |
| **ATA** | Actual Time of Arrival | Confirmed arrival time. |
| **CY** | Container Yard | Area in the terminal where containers are stored. |
| **CFS** | Container Freight Station | Warehouse for LCL cargo consolidation/deconsolidation. |
| **CY/CY** | Container Yard to Container Yard | Carrier's responsibility is port-to-port (CY at origin to CY at destination). |
| **CFS/CFS** | CFS to CFS | For LCL: carrier responsible from CFS at origin to CFS at destination. |
| **IMO** | International Maritime Organization | UN agency responsible for maritime safety and environmental regulations. |
| **IMDG** | International Maritime Dangerous Goods Code | Regulations for transporting hazardous materials by sea. |
| **SOLAS** | Safety of Life at Sea | International maritime safety treaty. Includes VGM requirement. |
| **VGM** | Verified Gross Mass | Mandatory verified weight of packed container (SOLAS 2016). |
| **OOG** | Out of Gauge | Cargo that exceeds standard container dimensions. |
| **DG** | Dangerous Goods | Hazardous cargo classified under IMDG code. |
| **GP** | General Purpose | Standard dry container. |
| **HC** | High Cube | Container with extra height (9'6" instead of 8'6"). |
| **SOC** | Shipper Owned Container | Container owned by the shipper, not the carrier. |
| **COC** | Carrier Owned Container | Container owned by the shipping line. Most common. |
| **D&D** | Demurrage and Detention | Combined term for port storage and equipment usage charges. |
| **BL / B/L** | Bill of Lading | Key shipping document — receipt, contract, title. |
| **SI** | Shipping Instructions | Shipper's instructions to prepare the B/L. |
| **DO / D/O** | Delivery Order | Authorization to release container from terminal. |
| **HS Code** | Harmonized System Code | International product classification for customs tariffs. |
| **AMS** | Automated Manifest System | US Customs advance manifest filing requirement (24-hour rule). |
| **ENS** | Entry Summary Declaration | EU advance manifest filing requirement. |
| **ISF** | Importer Security Filing | US requirement (also called "10+2"). Must be filed 24 hours before loading. |
| **LOA** | Length Overall | Total length of a vessel. |
| **DWT** | Deadweight Tonnage | Maximum weight a vessel can carry (cargo + fuel + stores). |
| **GRT / GT** | Gross Tonnage | Measure of vessel's internal volume. |
| **NRT / NT** | Net Tonnage | Measure of vessel's cargo-carrying capacity. |

### Transshipment, Feeder, and Mother Vessels

**Transshipment**
- The transfer of a container from one vessel to another at an intermediate port (hub port) en route to its final destination.
- Common because carriers cannot provide direct services between every port pair.
- Major transshipment hubs: Singapore, Port Klang, Colombo, Tanjung Pelepas, Dubai (Jebel Ali), Piraeus, Algeciras, Panama (Balboa/Cristobal).
- Transshipment incidence rate: ~30% of global container movements are transshipped.

**Mother Vessel (Main Line Vessel / Trunk Vessel)**
- Large container vessel operating on major trade lanes (e.g., Asia-Europe, Asia-US).
- Capacity: 10,000-24,000+ TEU.
- Calls only at major hub ports.
- Examples of mega vessels: Maersk Triple-E class (18,000 TEU), HMM Algeciras class (24,000 TEU), MSC Irina (24,346 TEU — largest as of 2024).

**Feeder Vessel**
- Smaller vessel that connects regional ports to hub ports.
- Capacity: typically 500-3,000 TEU.
- Collects containers from smaller ports and delivers them to the hub for transfer to mother vessels, and vice versa.
- Example: A feeder service might collect containers from ports in Vietnam and Thailand, delivering them to Singapore hub for onward carriage to Europe on a mother vessel.

### Trade Lanes

| Trade Lane | Route | Key Characteristics |
|------------|-------|---------------------|
| **Asia - Europe** | Far East to North Europe / Mediterranean | Longest major trade lane. ~25-35 days transit. Suez Canal route. |
| **Transpacific (TP)** | Asia to US West Coast / East Coast | Largest trade lane by volume. USWC ~12-15 days, USEC ~25-35 days (via Panama or Suez). |
| **Transatlantic (TA)** | Europe to US East Coast | Mature trade lane. ~10-14 days transit. |
| **Intra-Asia** | Within Asia | Highest growth. Short sea shipping. Feeder services. |
| **Asia - Middle East / ISC** | Asia to Indian Subcontinent / Middle East | Growing trade lane. |
| **North-South** | Europe/Asia to Africa / South America | Emerging markets. |

---

## 10. Common Business Challenges

### Container Shortage / Equipment Imbalance

- Global trade is imbalanced: more goods flow from Asia to US/Europe than the reverse.
- This creates **equipment imbalance** — empty containers accumulate in import-heavy regions and are scarce in export-heavy regions.
- Carriers must **reposition** empty containers at significant cost ($300-$800 per container).
- During peak seasons, container shortages drive up prices and cause booking rejections.
- COVID-19 pandemic severely exacerbated this: containers stuck at inland locations, ports, and warehouses globally.

### Port Congestion

- Occurs when terminal capacity is overwhelmed by vessel arrivals and container volumes.
- Causes: labor shortages, equipment breakdowns, sudden volume surges, chassis shortages (US), weather events.
- Effects: vessel waiting time (at anchor), delayed cargo, increased D&D charges, schedule disruptions.
- Notable congestion events: US West Coast (2021-2022), Shanghai lockdowns (2022), Red Sea disruptions (2024) causing rerouting via Cape of Good Hope.
- Software can help by: providing congestion visibility, optimizing yard operations, managing appointment systems.

### Blank Sailings

- A **blank sailing** is when a carrier cancels a scheduled voyage.
- Carriers blank sailings to manage capacity and prevent rate erosion during low-demand periods.
- Impact on shippers: cargo must be rolled to the next available vessel, causing delays.
- **Rolling** cargo: when a booked container is not loaded on the intended vessel and "rolled" to the next sailing.
- Shippers need visibility into blank sailing announcements to plan alternatives.

### Rate Volatility

- Ocean freight rates are highly volatile, driven by supply (vessel capacity) and demand (cargo volumes).
- Pre-COVID: Shanghai to US West Coast spot rate ~$1,500-$2,000 per 40ft.
- Peak COVID (2021): Same route exceeded $20,000 per 40ft.
- Post-COVID (2023): Rates crashed back to ~$1,500.
- 2024: Red Sea disruptions pushed rates up again ($4,000-$8,000).
- Rate indices: Shanghai Containerized Freight Index (SCFI), Drewry World Container Index (WCI), Freightos Baltic Index (FBX).
- Challenge for software: managing contract vs. spot rates, rate validity periods, surcharge volatility.

### Documentation Errors

- Documentation errors cause shipment delays, customs holds, and financial penalties.
- Common errors:
  - Incorrect HS code classification (wrong duties, potential penalties).
  - Mismatch between B/L details and actual cargo.
  - Wrong consignee/notify party information.
  - Missing or incorrect certificates of origin.
  - VGM discrepancies.
  - Late or incorrect AMS/ENS/ISF filings (can result in "Do Not Load" holds).
- Software solutions: automated validation rules, data reuse from templates, integration with customs systems, AI-based HS code suggestion.

### Customs Compliance

- Each country has different import regulations, restricted goods lists, and documentation requirements.
- Penalties for non-compliance: fines, cargo seizure, criminal prosecution, trade sanctions.
- Free Trade Agreement utilization: many shippers fail to claim preferential tariff rates they're entitled to.
- Denied party screening: must check all parties against government denied party lists (OFAC, BIS Entity List, EU Sanctions).

### Schedule Unreliability

- Vessels rarely arrive exactly on schedule.
- Cascading delays: a delay at one port pushes back arrivals at subsequent ports.
- Shippers struggle with: inventory planning, warehouse scheduling, customer commitments.
- Carriers implement: schedule recovery (increasing speed, skipping ports), vessel swaps.
- **Port omission**: When a carrier decides to skip a scheduled port call to recover schedule time. Cargo for that port must be transshipped or delivered overland.

### Cybersecurity & Digitalization

- The industry is rapidly digitizing but faces cybersecurity threats.
- Notable attack: Maersk's NotPetya ransomware attack (2017) — shut down operations for weeks, cost ~$300 million.
- Electronic Bill of Lading (eBL) adoption is growing but still limited (~2-3% of global B/Ls are electronic).
- DCSA (Digital Container Shipping Association) is developing industry standards for APIs and data exchange.
- Key digital transformation areas: electronic documentation, blockchain, IoT sensors, AI-based demand forecasting, automated terminals.

### Environmental Regulations

- **IMO 2020**: Vessels must use fuel with max 0.5% sulphur content (previously 3.5%). Carriers comply via: low-sulphur fuel (VLSFO), scrubbers, or LNG-powered vessels.
- **IMO 2023 (EEXI/CII)**: Energy Efficiency Existing Ship Index and Carbon Intensity Indicator. Vessels rated A-E. Below D requires improvement plans.
- **EU ETS (Emissions Trading System)**: From 2024, shipping included in EU carbon trading. Increases costs for EU-related voyages.
- **Green corridors**: Pilot routes for zero-emission vessels.
- Software impact: carbon footprint calculation per shipment, emissions reporting, fuel optimization.

### Container Damage & Claims

- Cargo damage during transit: water damage, physical damage, temperature excursion (reefer), theft, contamination.
- Claims process: consignee must inspect cargo upon receipt, file damage notice (within 3 days for apparent damage, 15 days for non-apparent).
- Surveys and documentation required for insurance claims.
- Software helps with: damage photo documentation, claims management workflow, carrier liability tracking.

---

## Summary: Key Points for Interview

1. **Understand the end-to-end flow**: From booking to empty container return. Know every step.
2. **Know the key players and their relationships**: Especially shipper, freight forwarder, NVOCC, carrier, customs broker, terminal operator.
3. **B/L is king**: Understand the difference between MBL and HBL, original vs. telex release, negotiable vs. non-negotiable.
4. **D&D is a pain point**: One of the biggest sources of disputes and unexpected costs. Software that manages free time tracking is valuable.
5. **Rate management is complex**: Multiple surcharges, volatile rates, contract vs. spot pricing.
6. **Visibility is the holy grail**: Real-time tracking, predictive ETA, exception alerts — this is what modern logistics software focuses on.
7. **Compliance is non-negotiable**: Customs regulations, AMS/ENS filings, VGM, dangerous goods — errors have serious consequences.
8. **The industry is digitalizing rapidly**: eBL, APIs (DCSA standards), IoT, AI — but legacy systems and paper-based processes still dominate.
9. **Schedule reliability matters**: Shippers need to know when their cargo will arrive. Predictive analytics are increasingly important.
10. **LCL vs. FCL**: Understand the different operational flows and how they affect CFS operations, documentation, and charges.
