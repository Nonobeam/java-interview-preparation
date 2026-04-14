## D5: Walk through the end-to-end container shipping process from booking to delivery.

**Why asked:** CLT needs developers who understand the full business flow their system supports. This is a foundational question.

**Answer:**

```
Booking → Empty Pickup → Stuffing → Pre-carriage → Documentation
→ Export Customs → Gate-In at Origin Terminal → Vessel Loading → Vessel Departure
→ Ocean Transit → [Transshipment if needed] → Vessel Arrival
→ Vessel Discharge → Import Customs → Delivery Order → Gate-Out → Delivery → Empty Return
```

### Step-by-Step

**1. Booking**
Shipper/forwarder requests space from carrier. Gets booking confirmation number + vessel/voyage assignment.

**2. Empty Container Pickup**
Trucker picks up empty container from carrier's depot. Equipment Interchange Receipt (EIR) issued — records container number, seal, condition.

**3. Stuffing / Loading**
- FCL: Shipper stuffs at their warehouse and seals the container.
- LCL: CFS operator consolidates multiple shippers' cargo.
- VGM (Verified Gross Mass) must be determined and submitted — mandatory since SOLAS 2016.

**4. Pre-carriage (Origin Inland)**
Container transported from shipper/CFS to port by truck, rail, or barge.

**5. Documentation**
Shipper submits Shipping Instructions (SI) → Carrier drafts Bill of Lading → Shipper approves → B/L finalized.

**6. Export Customs Clearance**
Customs declaration submitted. Customs may inspect (physical) or release (green channel). Export release obtained.

**7. Gate-In at Origin Terminal**
Container arrives at port. Gate records: container number, seal, truck, booking reference. Container enters CY (Container Yard).

**8. Vessel Loading**
Terminal plans stowage (bay plan). Quay cranes (STS) load containers onto vessel. Chief officer approves the plan.

**9. Vessel Departure (ETD)**
Cargo manifest finalized. Final B/L issued. Vessel departs. Advance manifests submitted to destination customs (AMS for US, ENS for EU).

**10. Ocean Transit**
Vessel sails — tracked via AIS. Duration: Asia-Europe ~25-35 days, Asia-US West Coast ~12-15 days.

**11. Transshipment (if applicable)**
Container discharged at hub port (e.g., Singapore), waits in CY, then loaded onto connecting vessel to final destination.

**12. Vessel Arrival & Discharge (ETA/ATA)**
Vessel arrives at Port of Discharge (POD). Containers discharged, placed in terminal CY. Arrival notice sent to consignee.

**13. Import Customs Clearance**
Customs declaration filed. Duties/taxes paid. Cargo released (green/yellow/red channel).

**14. Delivery Order & Container Release**
Consignee presents B/L to carrier agent (or telex release). Carrier verifies all charges paid. Carrier issues Delivery Order (D/O) — authorizes terminal to release container.

**15. Gate-Out**
Trucker presents D/O at gate. Terminal releases container. EIR issued.

**16. Destuffing & Delivery**
- FCL: Container to consignee's warehouse. Consignee destuffs.
- LCL: Container to destination CFS. CFS destuffs and separates consignments.

**17. Empty Container Return**
Empty container returned to carrier's depot within the free time period to avoid detention charges.

### Critical Milestones in a TMS
`BOOKING_CONFIRMED → EMPTY_PICKED_UP → GATE_IN → LOADED_ON_VESSEL → VESSEL_DEPARTED → VESSEL_ARRIVED → DISCHARGED → CUSTOMS_RELEASED → GATE_OUT → EMPTY_RETURNED`
