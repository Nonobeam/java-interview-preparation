## D12: How does the booking process work in container shipping?

**Why asked:** Booking is the entry point of every shipment in a logistics TMS. CLT developers need to understand the full lifecycle.

---

### What is a Booking?

A booking is a **reservation of container space** on a specific vessel/voyage from a carrier. It is the first formal step — nothing else can happen without a confirmed booking.

---

### Who Books with Whom?

```
Shipper → Freight Forwarder / NVOCC → Carrier (Shipping Line)
   or
Shipper → directly to Carrier (if no forwarder)
```

- The **forwarder/NVOCC** books space from the carrier on behalf of the shipper
- The carrier sees the forwarder as the customer (not the actual shipper)
- For LCL, the NVOCC consolidates multiple shippers' cargo into one FCL booking with the carrier

---

### Key Booking Fields

| Field | Description |
|-------|-------------|
| **Origin (POL)** | Port of Loading — where cargo goes on the vessel |
| **Destination (POD)** | Port of Discharge — where it comes off |
| **Commodity** | Type of goods (affects DG check, reefer needs, restrictions) |
| **Container Type** | 20GP, 40GP, 40HC, reefer, flat rack, etc. |
| **Container Quantity** | How many containers |
| **Cargo Weight / Volume** | Gross weight (kg) and CBM |
| **ETD** | Estimated time of departure — which sailing/voyage |
| **Incoterm** | FOB, CIF, etc. — determines who pays freight |
| **Shipper / Consignee** | Parties for the B/L |
| **Special Instructions** | Reefer temp settings, hazardous goods, OOG dimensions |

---

### Booking Lifecycle / Status Flow

```
DRAFT → SUBMITTED → CONFIRMED → AMENDED → CANCELLED
                                    ↓
                              SI SUBMITTED
                                    ↓
                              B/L DRAFT ISSUED
                                    ↓
                              B/L CONFIRMED
```

**DRAFT** — Forwarder preparing the booking request internally

**SUBMITTED** — Sent to the carrier (via email, INTTRA portal, or API)

**CONFIRMED** — Carrier accepts and issues:
- **Booking Confirmation Number** (carrier reference)
- **Vessel name + Voyage number** (e.g., EVER GIVEN / 0123E)
- **Container release reference** — required to pick up empty container from depot
- **Cut-off dates** (see below)

**AMENDED** — Changes requested after confirmation (cargo weight change, container swap, date change). Carrier must re-confirm. May incur amendment fees.

**CANCELLED** — Booking cancelled. If cancelled late, carrier may apply a **no-show fee** or **cancellation fee**.

---

### Critical Cut-off Dates

These are deadlines the forwarder must meet for the cargo to make the vessel. Missing any = cargo rolled to next sailing.

| Cut-off | What it means |
|---------|--------------|
| **SI Cut-off** (Shipping Instructions) | Deadline to submit B/L draft instructions to the carrier |
| **VGM Cut-off** | Deadline to submit Verified Gross Mass |
| **CY Cut-off** (Gate Cut-off) | Last day/time to bring the full container into the terminal |
| **Documentation Cut-off** | Final deadline for all export documents to customs |
| **Hazmat Cut-off** | Earlier cut-off for dangerous goods bookings |

**Typical timeline before ETD:**
```
ETD - 7 days: SI cut-off
ETD - 5 days: VGM cut-off
ETD - 3 days: CY cut-off (gate closes for that vessel)
ETD - 2 days: Vessel loads and seals
ETD:          Vessel departs
```

---

### Space Allocation & Overbooking

- Carriers allocate space (TEU and weight) per trade lane per vessel
- Carriers often **overbook** intentionally — expecting no-shows and cancellations (similar to airlines)
- When a vessel is full: carrier may **roll** cargo to the next sailing
- **Rolling** = confirmed booking but cargo not loaded — very disruptive for shippers with tight delivery commitments
- Premium bookings (higher-paying or contract customers) get priority space

---

### Booking Amendments

Common amendment reasons:
- Cargo weight changed (must resubmit VGM)
- Container type swap (e.g., 20GP → 40HC)
- Date change (roll to later sailing)
- Consignee info updated
- Add/remove containers

Each amendment goes back to the carrier for re-confirmation. Some carriers charge an **amendment fee** (~$25-$50). After a certain point (close to cut-off), amendments may be rejected.

---

### Booking Cancellation & No-Show Fee

- Cancellations before cut-off: usually free or small admin fee
- **No-show**: container was booked and confirmed but not delivered to the terminal by CY cut-off
- Carriers charge a **no-show fee** or **dead freight** — compensation for unused space
- Varies by carrier: typically $200-$500 per container

---

### Software Implications for TMS

A booking module must handle:

1. **Multi-carrier booking** — same shipment may compare rates across 3-5 carriers before confirming
2. **Cut-off alerts** — auto-remind operations team of upcoming SI/VGM/CY cut-offs
3. **Amendment workflow** — track each amendment version with timestamps and who changed what
4. **Roll management** — when carrier rolls cargo, update vessel/voyage, recalculate ETD/ETA, notify customer
5. **No-show tracking** — flag containers that missed CY cut-off, calculate potential no-show fees
6. **Carrier integration** — INTTRA or DCSA API for electronic booking submission and status updates
7. **Booking reference mapping** — carrier booking number ↔ internal reference ↔ customer reference (all three exist simultaneously)

---

### Key Terms Summary

| Term | Meaning |
|------|---------|
| **Booking Confirmation** | Carrier's acceptance of the space request |
| **Container Release Reference** | Code to pick up empty container from the carrier's depot |
| **CY Cut-off** | Last gate-in time for the vessel — miss this = rolled |
| **Rolling** | Cargo not loaded on intended vessel, moved to next sailing |
| **No-show Fee** | Penalty for confirmed but unused space |
| **Blank Sailing** | Carrier cancels the entire voyage (no cargo loaded at all) |
| **Vessel/Voyage** | Identifies the specific ship and trip (e.g., EVER GIVEN / 0123E) |
