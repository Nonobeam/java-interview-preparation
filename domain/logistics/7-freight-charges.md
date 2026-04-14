## D7: What charges and surcharges are involved in a container shipment?

**Why asked:** CLT's system manages pricing, quotation, and invoicing. Understanding the charge structure is essential to building correct rate calculation logic.

**Answer:**

### Base Charge: Ocean Freight
- The core cost of moving a container from POL to POD
- FCL: quoted per container (e.g., $1,500 per 20ft, $2,200 per 40HC)
- LCL: quoted per CBM or per Revenue Ton (W/M), whichever is greater
- Types: **Contract Rate (NAC)** — negotiated quarterly/annually vs **Spot Rate** — one-time, more volatile

### Common Surcharges

| Code | Full Name | Description |
|------|-----------|-------------|
| **THC** | Terminal Handling Charge | Charged at both origin and destination terminal for loading/discharging. Typically $100-$300/container. |
| **BAF / EBS** | Bunker Adjustment Factor / Emergency Bunker Surcharge | Covers fuel (bunker) cost fluctuations. |
| **LSS / IMO 2020** | Low Sulphur Surcharge | IMO 2020 compliance surcharge for low-sulphur fuel requirement. |
| **CAF** | Currency Adjustment Factor | Covers exchange rate fluctuations. |
| **PSS** | Peak Season Surcharge | Applied during high-demand seasons (typically July-October for Asia-US/Europe). |
| **GRI** | General Rate Increase | Carrier-announced rate increase, usually monthly. |
| **WRS** | War Risk Surcharge | For routes through high-risk zones (Red Sea, Gulf of Aden). |
| **ISPS** | Int'l Ship & Port Facility Security | Post-9/11 security surcharge. |
| **Congestion Surcharge** | — | Applied at congested ports. |
| **Reefer Surcharge** | — | Additional cost for refrigerated containers (power + monitoring). |
| **OOG Surcharge** | Out of Gauge | For flat rack or open top containers with oversized cargo. |

### Documentation & Admin Fees

| Fee | Description |
|-----|-------------|
| **DOC Fee** | B/L documentation preparation fee. ~$25-$75. |
| **VGM Fee** | Container weighing fee for SOLAS compliance. |
| **Seal Fee** | Container seal cost. |
| **Agency Fee** | Local agent fee for documentation handling and cargo release. |

### Inland / Other Charges

| Fee | Description |
|-----|-------------|
| **IHC** (Inland Haulage Charge) | Truck/rail from port to consignee or shipper premises. |
| **CFS Charges** | For LCL cargo — consolidation/deconsolidation at CFS. Per CBM or RT. |
| **Wharfage** | Port authority fee for using port facilities. |
| **Chassis Fee** | US-specific — fee for the truck chassis used to move containers. |

### Pricing Terms
- **All-in Rate**: One total price that bundles ocean freight + all surcharges (easier to quote but harder to audit)
- **Base + Surcharges**: Ocean freight quoted separately from surcharges (transparent but complex)
- **FAK** (Freight All Kinds): One rate regardless of commodity type

### Rate Validity
All rates have a validity period. After expiry, the rate is no longer honored. Surcharges (especially BAF/PSS/GRI) may update monthly. A rate management module in TMS must handle validity periods and alert when rates expire.
