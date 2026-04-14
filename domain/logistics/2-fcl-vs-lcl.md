## D2: What is FCL vs LCL? What are the operational differences?

**Why asked:** FCL and LCL are the two fundamental cargo modes in container shipping. Every system at CLT will distinguish between them.

**Answer:**

| | FCL (Full Container Load) | LCL (Less than Container Load) |
|---|---|---|
| **Definition** | One shipper uses the entire container | Multiple shippers share one container |
| **Stuffing** | Done by shipper at their warehouse | Done by CFS operator (consolidator) |
| **Seal** | Shipper seals the container | CFS operator seals after consolidation |
| **Pricing** | Per container (e.g., $2,000 per 40HC) | Per CBM or W/M (weight/measure, whichever is greater) |
| **Transit time** | Faster — container goes direct | Slower — must consolidate at origin CFS, deconsolidate at destination CFS |
| **Cost** | Higher base cost, but better for large volumes | Cheaper for small shipments |
| **Documentation** | One B/L per container | Multiple B/Ls for each individual shipper (HBLs from NVOCC) |
| **Risk** | Shipper responsible for cargo integrity inside container | Multiple parties' cargo mixed — higher risk of cross-contamination or damage claims |
| **When to use** | Cargo fills 12+ CBM or >5 tons | Small shipments, samples, low-volume cargo |

**Operational flow difference:**

**FCL:**
```
Shipper's warehouse → Stuff & seal → Trucking → Origin CY → Vessel → Destination CY → Trucking → Consignee
```

**LCL:**
```
Multiple shippers → Origin CFS (consolidation) → One container → Vessel → Destination CFS (deconsolidation) → Individual consignees
```

**CFS charges** apply for LCL — typically charged per CBM or Revenue Ton (RT), whichever is greater. Revenue Ton = 1,000 kg = 1 CBM (carrier uses whichever gives higher revenue).

**Software implication:** A TMS must handle both modes — different document flows, charge structures, and tracking events.
