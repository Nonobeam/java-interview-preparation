## D9: What container types exist and when is each used?

**Why asked:** Container type is a key field in every booking and shipment record in CLT's system. Wrong container type causes failed bookings.

**Answer:**

### Standard Dry Containers

| Type | External Size | Internal Volume | Max Payload | Use Case |
|------|--------------|-----------------|-------------|----------|
| **20ft GP** | 20' x 8' x 8'6" | ~33 CBM | ~28,000 kg | Heavy cargo — steel, machinery, chemicals. Weight limit reached before volume. |
| **40ft GP** | 40' x 8' x 8'6" | ~67 CBM | ~28,500 kg | General cargo — most versatile option. |
| **40ft HC (High Cube)** | 40' x 8' x 9'6" | ~76 CBM | ~28,500 kg | **Most popular globally.** Extra 1 foot of height. Voluminous/lightweight cargo: furniture, electronics, garments, bikes. |
| **45ft HC** | 45' x 8' x 9'6" | ~86 CBM | ~27,600 kg | Mainly North American and intra-Asian trades. |

### Specialized Containers

**Reefer (Refrigerated)**
- Has built-in refrigeration unit
- Temperature range: -35°C to +30°C
- Needs electrical power (genset on truck, reefer plugs at terminal/vessel)
- Used for: perishable food (fruits, meat, seafood), pharmaceuticals, chemicals
- Available: 20ft reefer, 40ft HC reefer
- **Software implication:** Reefer bookings require power plug availability tracking and temperature log management

**Open Top**
- No solid roof — covered by tarpaulin
- Allows crane top-loading
- Used for: heavy machinery, marble slabs, timber, large industrial equipment
- Available: 20ft OT, 40ft OT

**Flat Rack**
- No side walls or roof — floor only with end walls (fixed or collapsible)
- Used for: OOG (Out of Gauge) cargo — machinery, vehicles, boats, industrial pipes
- Cargo may extend beyond container dimensions → requires "OOG stowage" on vessel, extra planning and surcharges
- Available: 20ft FR, 40ft FR

**Tank Container (ISO Tank)**
- Cylindrical tank inside a standard frame
- Capacity: 21,000–26,000 liters
- Used for: liquid chemicals, food-grade liquids (wine, juice, oils), hazardous liquids
- Must follow IMDG dangerous goods regulations if hazardous

**Other Specialized Types**
| Type | Use Case |
|------|----------|
| **Ventilated** | Coffee, cocoa, onions — cargo that needs airflow to prevent moisture buildup |
| **Insulated (Thermal)** | Temperature-sensitive but no active cooling needed |
| **Garment (GOH)** | Fitted with hanging rails for garments on hangers |
| **Bulk** | Roof hatches for top-loading bulk cargo (grain, powder, seeds) |

### Choosing Container Type
| Cargo | Recommended Container |
|-------|-----------------------|
| General dry goods, electronics | 40ft HC |
| Frozen meat, pharmaceuticals | 40ft HC Reefer |
| Heavy machinery (OOG) | Flat Rack |
| Chemicals (liquid) | ISO Tank |
| Steel coils | 20ft GP (weight-constrained) |
| Garments on hangers | GOH |
| Marble blocks | Open Top or Flat Rack |
