## D11: What are the common business challenges in container shipping?

**Why asked:** CLT wants developers who understand the problems their software solves. This shows business awareness.

**Answer:**

### 1. Container Shortage / Equipment Imbalance
- More goods flow Asia → US/Europe than the reverse → empty containers accumulate at import regions
- Carriers must reposition empties at $300-$800 per unit cost
- During peak season: bookings rejected, equipment scarce
- **Software help:** Equipment inventory tracking, repositioning optimization

### 2. Port Congestion
- Terminal overwhelmed → vessels queue at anchor → delayed cargo pickup → D&D charges pile up
- Causes: labor strikes, equipment breakdowns, sudden volume surges, chassis shortages
- Notable: US West Coast 2021-2022, Shanghai lockdowns 2022, Red Sea rerouting 2024 (all ships going around Africa via Cape of Good Hope)
- **Software help:** Congestion alerts, ETA re-forecasting, D&D exposure alerts, port call monitoring

### 3. Blank Sailings & Rolled Cargo
- **Blank sailing**: Carrier cancels a scheduled voyage to manage capacity
- **Rolling**: Booked container NOT loaded on intended vessel → moved to next sailing
- Impact: unexpected delays, warehouse demurrage, customer SLA violations
- **Software help:** Sailing schedule monitoring, proactive rebooking workflow, customer notification

### 4. Rate Volatility
- Ocean freight rates can change dramatically:
  - Pre-COVID Asia-USWC: ~$1,500-$2,000/40ft
  - COVID peak 2021: >$20,000/40ft
  - Post-COVID 2023: back to ~$1,500
  - Red Sea disruptions 2024: $4,000-$8,000
- Rate indices: SCFI (Shanghai Containerized Freight Index), Drewry WCI, Freightos FBX
- **Software help:** Rate validity tracking, contract vs spot comparison, alert when spot exceeds contract threshold

### 5. Documentation Errors
Most common errors and consequences:
| Error | Consequence |
|-------|-------------|
| Wrong HS code | Incorrect duty, customs penalty, seizure |
| B/L details mismatch vs cargo | Customs hold, amendment costs |
| Late AMS/ENS/ISF filing | "Do Not Load" order |
| Missing phytosanitary certificate | Cargo refused at destination |
| Wrong consignee info | B/L amendment required, delays |
- **Software help:** Validation rules, mandatory field checks, automated customs filings, template reuse

### 6. Schedule Unreliability
- Global carrier on-time performance: 50-70% normally, dropped to ~30% during COVID
- Cascading delays: one late vessel pushes subsequent port calls
- **Software help:** Dynamic ETA updates via AIS integration, predictive delay models, customer proactive notifications

### 7. D&D Charge Disputes
- Carriers apply charges that customers dispute as incorrect or unfair
- Without accurate records, customers cannot challenge charges
- **Software help:** Precise free time tracking per carrier/port/container, accrual calculations, audit trail for disputes

### 8. Compliance & Sanctions
- Every shipment must pass denied party screening (shipper, consignee, vessel, port)
- Violation = massive fines and criminal liability
- **Software help:** Real-time screening against OFAC, EU sanctions, BIS Entity List

### 9. Visibility Gap
- Shippers often don't know where their cargo is until it's late
- The "holy grail" of modern logistics software = real-time visibility
- **Software help:** Multi-carrier tracking integration, event-driven notifications, predictive ETA

### Key Insight for Interview
> "Almost every feature in a logistics TMS exists to solve one of these challenges. When I understand the problem, I can build better solutions — validation rules for documentation, D&D tracking systems, ETA alert pipelines."
