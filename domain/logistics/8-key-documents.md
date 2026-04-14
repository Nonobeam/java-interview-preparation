## D8: What are the key shipping documents and what is each used for?

**Why asked:** Document management is a core function of logistics software. CLT's system generates, validates, and archives these documents.

**Answer:**

### The Core Set

| Document | Who Prepares | Purpose |
|----------|-------------|---------|
| **Bill of Lading (B/L)** | Carrier (MBL) or Forwarder/NVOCC (HBL) | Contract of carriage + receipt of goods + document of title. Most important. |
| **Shipping Instructions (SI)** | Shipper | Shipper's instructions to carrier/forwarder for how to prepare the B/L. |
| **Commercial Invoice** | Seller/Exporter | Value of goods — used for customs valuation and duty calculation. |
| **Packing List** | Seller/Exporter | Detailed contents of each package/container. Used by customs, consignee, and for claims. |
| **Certificate of Origin (CO)** | Exporter / Trade Authority | Certifies the country of manufacture. Required for customs and FTA preferential duty rates. |
| **Customs Declaration (Export/Import)** | Customs Broker | Filed with customs authority. Declares HS code, value, origin, quantity. |
| **Delivery Order (D/O)** | Carrier (local agent) | Authorizes the terminal to release the container to the consignee. |
| **Arrival Notice** | Carrier (local agent) | Notifies consignee that vessel has arrived or is approaching. |

### Specialized Documents

| Document | Purpose |
|----------|---------|
| **VGM Declaration** | Verified Gross Mass of packed container. SOLAS mandatory since 2016. Must be submitted before vessel cutoff. |
| **Dangerous Goods Declaration (DGD)** | Required for hazardous cargo. States IMDG class, UN number, proper shipping name, flash point. |
| **Phytosanitary Certificate** | For plant-based products — certifies freedom from pests/diseases. Required for agricultural goods, timber. |
| **Fumigation Certificate** | Confirms cargo/container was fumigated. Required by many countries for wooden packaging. |
| **Marine Cargo Insurance Policy** | Covers cargo against loss/damage. Required under CIF Incoterm. |
| **Letter of Credit (L/C)** | Bank guarantee of payment. Requires specific documents to be presented to the bank within deadlines. Requires a clean B/L. |
| **AMS / ENS / ISF Filing** | Advance customs manifest filings: US (AMS, ISF 10+2), EU (ENS). Must be submitted before cargo is loaded. |

### Document Flow (Export Side)
```
Shipper submits SI
  → Carrier prepares draft B/L
  → Shipper approves B/L
  → Customs declaration submitted
  → Export customs releases cargo
  → Vessel loaded, final B/L issued
  → Arrival Notice sent to consignee
  → Consignee presents B/L to agent
  → Agent issues Delivery Order
  → Terminal releases container
```

### Common Documentation Errors (and their consequences)
- **Wrong HS code** → incorrect duty, potential penalty, customs hold
- **Weight mismatch** (B/L vs actual VGM) → vessel loading denied
- **Missing/late AMS filing** → "Do Not Load" order from US customs
- **Dirty B/L** → bank rejects documents under L/C transaction
- **B/L not arrived before vessel** → consignee cannot pick up cargo (telex release needed)
