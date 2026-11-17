# Warehouse Logistics & WMS Synthetic Data Generator (Python)

A deterministic, domain-driven Python simulation engine that generates a complete, relationally consistent multi-year **Warehouse Management System (WMS) & Logistics dataset** modeled after an industrial electrical-components distribution center in **Baden-Württemberg, Germany**.

Designed for **Data Engineers, Analytics Engineers, and BI Developers** who need realistic operational supply-chain data—complete with shift schedules, hall-level hourly picking bottlenecks, physical box vs. pallet packing rules, and carrier SLA performance—to build data warehouses, dbt/SQL pipelines, and Power BI / Tableau Control Tower dashboards.

---

## Key Simulation Highlights

- **Real-World German Industrial Context:** Calendar engine (`business_calendar.py`) models official Baden-Württemberg public holidays, school holidays (_Ferien_ capacity dips to `0.90`), weekend closures, and seasonal autumn demand peaks (`2024–2027`).
- **3-Hall, 12-Zone Physical Topology (`24,180` Storage Locations):**
  - **Hall 1 (Halle 4):** `18,500` locations across Zones 1–7 (`FORKLIFT`, `PICKING_CART`, `VERTICAL_LIFT`). Houses 5 _Kleinpacken_ (`H1-KP-01..05`) and 9 _Großpacken_ (`H1-GP-01..09`) stations.
  - **Hall 2 (Halle 4A):** `4,240` locations across Zones 8–10 (`VERTICAL_LIFT`, `FORKLIFT`, `PICKING_CART`).
  - **Hall 3 (Halle 3):** `1,440` locations across Zones 11–12 (`PICKING_CART`, `STAPLER`). Houses 2 _Großpacken_ (`H3-GP-01..02`) stations.
- **Physical Weight & Packaging Rules (`<= 30 kg` Cartons vs. `> 30 kg` Pallets):**
  - Orders `<= 30.0 kg` (`84.5%`) are packed into standardized cartons (`BOX_201`, `BOX_202`, `BOX_203`) across **Kleinpacken** (`1–4` lines & `<= 10 kg`) and **Großpacken** (`<= 30 kg`).
  - Single-piece postal/parcel order lines (`requested_quantity = 1`) represent `~40.5%` of all line items.
  - Heavy B2B orders `> 30.0 kg` (`15.5%`, up to `2,500 kg`) are routed to **Pallet Packing** (`PALLET_SHIPMENT` / `PALLET_FREIGHT`).
- **Hourly Hall Capacity Queue & Bottleneck Simulation:**
  - `capacity_queue.py` enforces strict shift operating hours (`06:00–21:00` across _Frühschicht_ and _Spätschicht_) and hourly hall pick limits (`Hall 1: 115/h`, `Hall 2: 24/h`, `Hall 3: 10/h`), naturally producing realistic SLA delays during peak demand and holiday staffing dips (`~93.3%` On-Time Delivery rate).
- **Zero-Discrepancy Referential Integrity:**
  - Built-in assertions and `src/pipeline/validation.py` verify 100% cross-table consistency across all 10 generated CSV datasets (`0` orphan keys, `0` capacity violations, exact weight and quantity reconciliation).

---

## Generated Datasets (`data/processed/`)

| File                 |   Records | Description                                                                                                          |
| :------------------- | --------: | :------------------------------------------------------------------------------------------------------------------- |
| `calendar.csv`       |   `1,186` | Daily Baden-Württemberg business calendar (`2024-01-01` to `2027-03-31`) with demand & capacity multipliers          |
| `locations.csv`      |  `24,180` | 3 warehouse halls, 12 storage zones, storage types (`BOX_STORAGE`, `PALLET_STORAGE`), and picking methods            |
| `product_master.csv` |  `37,500` | Standardized industrial electrical product catalog with unit weights (`kg`), prices (`EUR`), and EAN/tariff codes    |
| `inventory.csv`      |  `16,926` | Opening warehouse stock snapshot (`1.79M` units across box and pallet locations)                                     |
| `customers.csv`      |     `500` | Active B2B (`80%`) and B2C (`20%`) customer master records                                                           |
| `orders.csv`         | `110,000` | Multi-year order headers (`2024`: 34k, `2025`: 40k, `2026`: 36k) with priorities and requested delivery deadlines    |
| `order_items.csv`    | `259,425` | Order line items (`avg 2.36 lines/order`, `40.5%` single-unit lines)                                                 |
| `picking.csv`        | `261,053` | Shift- and capacity-scheduled picking operations across Hall 1, Hall 2, and Hall 3                                   |
| `packing.csv`        | `110,000` | Station-level packing logs (`H1-KP`, `H1-GP`, `H3-GP`), workload units, and package types (`BOX_201..203`, `PALLET`) |
| `shipping.csv`       | `110,000` | Carrier dispatch (`DHL`, `GLS`, `AIR_FREIGHT`, `OCEAN_FREIGHT`), transit times, and On-Time Delivery (`93.30%`)      |

_(Plus 6 diagnostic CSV reports in `data/processed/diagnostics/` auditing hourly hall utilization and monthly timeliness)._

---

## Project Structure

```text
├── data/
│   ├── raw/                                  # Source product catalog input
│   └── processed/                            # Generated CSV datasets & diagnostics
├── src/
│   ├── generators/
│   │   ├── business_calendar.py              # Step 1: BW holiday & shift capacity calendar
│   │   ├── location.py                       # Step 2: 3-hall, 12-zone storage topology
│   │   ├── products.py                       # Step 3: Product master standardization
│   │   ├── inventory.py                      # Step 4: Opening stock allocation
│   │   ├── orders.py                         # Step 5: Customer, order & line-item generator
│   │   ├── picking.py                        # Step 6: Picking & hall capacity queue simulation
│   │   ├── packing.py                        # Step 7: Packing station & carton/pallet logic
│   │   ├── shipping.py                       # Step 8: Carrier dispatch & SLA tracking
│   │   ├── picking_capacity_diagnostics.py   # Bonus: Hourly/daily capacity utilization audit
│   │   ├── capacity_queue.py                 # Hourly hall capacity reservation engine
│   │   ├── containers.py                     # Warehouse container ID & type logic
│   │   └── working_time.py                   # Two-shift schedule (06:30-22:45) with shift breaks & working-day math
│   └── pipeline/
│       └── validation.py                     # Step 9: End-to-end cross-dataset QA validation
├── requirements.txt
└── run_pipeline.py                           # Single-command master orchestrator
```
