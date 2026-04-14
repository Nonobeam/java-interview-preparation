## D4: What are Incoterms? Explain FOB vs CIF — the most common ones.

**Why asked:** Incoterms determine who arranges and pays for each leg of the journey. This affects how CLT's system assigns responsibility and cost.

**Answer:**

**Incoterms** (International Commercial Terms) are standardized trade terms published by the ICC (International Chamber of Commerce). They define:
- **Where** risk transfers from seller to buyer
- **Who** pays for each leg of transport
- **Who** arranges insurance

Current version: **Incoterms 2020**.

### Full List

| Term | Name | Risk Transfer Point | Who Pays Freight |
|------|------|---------------------|-----------------|
| **EXW** | Ex Works | At seller's warehouse | Buyer pays everything |
| **FCA** | Free Carrier | When handed to first carrier at named place | Buyer pays main carriage |
| **FAS** | Free Alongside Ship | When alongside vessel at POL | Buyer |
| **FOB** | Free On Board | When loaded on board vessel at POL | Buyer pays ocean freight |
| **CFR** | Cost and Freight | On board vessel at POL (risk) | Seller pays ocean freight |
| **CIF** | Cost, Insurance and Freight | On board vessel at POL (risk) | Seller pays freight + insurance |
| **CPT** | Carriage Paid To | Handed to first carrier | Seller pays to named destination |
| **CIP** | Carriage and Insurance Paid To | Handed to first carrier | Seller pays to named destination + insurance |
| **DAP** | Delivered at Place | At named destination, ready for unloading | Seller pays everything to destination |
| **DPU** | Delivered at Place Unloaded | After unloading at destination | Seller unloads |
| **DDP** | Delivered Duty Paid | At named destination, duties paid | Seller pays everything including import duties |

### FOB vs CIF — The Most Important Pair

**FOB (Free On Board) — Port of Loading**
- Seller delivers goods loaded on board the vessel at the named port of loading
- Risk transfers to buyer the moment goods are on board
- **Buyer arranges and pays for ocean freight and insurance**
- Most common for Asia export trade (buyer in US/Europe controls the freight)
- Example: "FOB Ho Chi Minh City" — seller's responsibility ends when container is loaded at HCMC port

**CIF (Cost, Insurance, Freight) — Port of Destination**
- Seller delivers on board the vessel (same as FOB for risk transfer!)
- But seller also **pays for** ocean freight AND arranges minimum cargo insurance
- Buyer still bears risk during ocean transit (confusing but correct — risk transfers at POL, even though seller pays freight)
- Common for buyers who want a simpler price quote inclusive of shipping

**The trap:** Under CIF, the seller pays for freight but the risk transfers at the same point as FOB (on board at POL). So if the cargo sinks mid-ocean, the buyer bears the loss even though the seller paid for shipping.

### Key insight for CLT
When a shipment record is created in the system, the Incoterm determines:
- Who is the payer for ocean freight (affects invoicing)
- What documents the seller must provide
- Where the seller's responsibility in the tracking flow ends
