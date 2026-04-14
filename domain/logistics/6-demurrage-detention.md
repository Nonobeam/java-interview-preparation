## D6: What is demurrage vs detention? Why does it matter?

**Why asked:** D&D is one of the biggest pain points and disputes in logistics. CLT's system almost certainly needs to track and calculate these charges.

**Answer:**

These are charges imposed by the carrier when containers are held beyond the allowed **free time** period. Both escalate daily and are a major source of unexpected costs and disputes.

### Three Distinct Charges

| Charge | Charged by | When it applies | What triggers it |
|--------|-----------|-----------------|-----------------|
| **Demurrage** | Carrier | Full container sitting inside the terminal beyond free time | Container not picked up from terminal after vessel discharge |
| **Detention** | Carrier | Container outside the terminal beyond free time | Container not returned empty to depot after delivery |
| **Storage** | Terminal Operator | Full container in terminal beyond terminal free time | Similar to demurrage, but paid to the terminal, not the carrier |

### Timeline of Charges (Import Side)

```
Vessel Discharge
      |
      |--- FREE TIME starts (e.g., 5 days)
      |
      |--- Day 5: Free time expires
      |
      |--- DEMURRAGE starts accruing (e.g., $50/day days 6-10, $100/day days 11-15, $150/day after)
      |
     Gate-Out (container leaves terminal)
      |
      |--- DEMURRAGE stops
      |--- DETENTION free time starts (e.g., 7 days)
      |
      |--- DETENTION starts accruing if not returned by then
      |
     Empty Return to Depot → DETENTION stops
```

### Example Calculation
- Free time: 5 days demurrage, 7 days detention
- Container discharged Day 1
- Consignee picks up container on Day 8 → **3 days demurrage** (days 6, 7, 8)
- Consignee returns empty on Day 18 → **4 days detention** (days 15, 16, 17, 18 after the 7-day free time from gate-out)

### Combined Free Time
Some carriers offer "combined D&D free time" — e.g., 14 days combined from vessel discharge to empty return. This simplifies calculation but total exposure is the same.

### Why It Matters for Software
- The system must track each container's discharge date and automatically calculate when free time expires
- Alert users before free time runs out
- Calculate accrued D&D per shipment for dispute resolution
- Integrate with carrier invoices to validate charges
- D&D disputes are extremely common — carriers sometimes apply charges incorrectly, and having accurate system records is the only defense

### Industry Context
- During COVID port congestion (2021-2022), many consignees faced D&D charges they couldn't avoid due to port delays outside their control
- The US Federal Maritime Commission (FMC) introduced rules in 2022 to limit unfair D&D practices
- D&D management is a high-value feature in any logistics TMS
