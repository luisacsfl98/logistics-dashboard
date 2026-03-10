# 📊 Last-Mile Logistics Dashboard — Real-Time Order Monitoring

![Dashboard Preview](Dashboard%20de%20monitoramento%20-%20Modelo.xlsx)

> Operational monitoring dashboard tracking 3,200–3,800 daily orders across multiple states and logistics hubs, built entirely in Excel + Power Query. Designed for daily operational prioritization of open orders within a rolling 6-day window.

---

## 🧩 Business Context

A last-mile logistics operation managing deliveries for a major retail group had no centralized way to monitor order status across hubs and regions. Analysts were querying systems manually — making it hard to catch delays early or prioritize critical orders before they breached SLA.

**The problem:** No single view of open orders and their risk level across 3,000+ daily orders.  
**The solution:** A self-updating dashboard focused on open orders approved in the last 6 days, with automated criticality classification to drive daily operational action.

---

## 🎯 Objective

Build a logistics monitoring panel that:

- Pulls orders approved in a rolling **D-6 window** (last 6 days up to analysis date)
- Classifies open orders by delivery risk level to drive daily prioritization
- Surfaces critical orders before they breach the performance indicator threshold
- Updates daily by replacing the base data — no manual rework

---

## 🛠️ Why Excel (not Power BI or Tableau)

Excel was the deliberate choice for two operational reasons:

1. **Data swap simplicity** — the base table is a direct TMS export in `.xlsx` format. Replacing it is a one-step action any team member can do without technical knowledge.
2. **Direct operational filtering** — the team needs to filter and act on individual orders. Excel slicers + pivot tables allow the analyst to go from summary view to specific critical orders in seconds, without switching tools.

---

## ⚙️ How It Was Built

### Step 1 — Data Extraction
Orders exported daily from the TMS as **CSV**, with **approval date between D-6 and D-0** (analysis date).

For example: analysis on March 9 → pulls orders approved between March 3–9.

The CSV is then treated in Power Query and saved as `.xlsx`, which feeds the Excel Data Model.

**Data pipeline architecture:**
```
TMS Export (CSV) → Power Query (treatment + normalization) → .xlsx → Data Model → Pivot Tables → Slicers → Dashboard
```

Base contains:

| Column | Description |
|--------|-------------|
| `Pedido` | Order ID |
| `modelo_pedido` | Order type |
| `Município / UF` | Destination city and state |
| `Prazo Cliente` | Customer deadline |
| `Transportadora` | Carrier |
| `Status` | Raw TMS status |
| `classificacao_status` | Normalized operational status (Power Query) |
| `dt_aprovacao` | Order approval date |
| `dt_ultima_atualizacao` | Last status update |
| `Lead time operacional` | Days since approval |
| `Status interno` | Criticality classification |
| `Loja / Praça` | Origin store and logistics region |
| `Existem Ocorrências` | Whether delivery issues were flagged |

### Step 2 — Status Normalization (Power Query)
Raw TMS exports contain 15–20 granular status codes. A Power Query conditional column maps them into 5 operational categories:

| Classificação Status | Meaning |
|---------------------|---------|
| **Finalizado** | Confirmed delivered or picked up by customer |
| **Em Rota** | In active transit — out for delivery or in movement |
| **Pendente de Rota** | Awaiting dispatch programming |
| **Pendente** | Awaiting processing |
| **Insucesso** | Failed delivery attempt |

> **Note:** Only confirmed final statuses ("Delivered", "Picked up") are classified as Finalizado. Statuses like "Out for delivery" or "Collected" remain as **Em Rota** since the order can still return as Insucesso.

### Step 3 — Store × Region Mapping (Power Query)
Base merged with a reference table (`Referência Praças`) mapping each store code to its logistics region — enabling regional segmentation in the dashboard.

### Step 4 — Lead Time Calculation
```
Lead Time Operacional = Last Status Update − Approval Date (in days)
```

### Step 5 — Criticality Classification (open orders only)
Applied only to orders **not yet finalized**. Classifies proximity to losing the performance indicator threshold:

| Lead Time | Status Interno | Meaning |
|-----------|---------------|---------|
| ≤ 3 days | ✅ No prazo | On track |
| 4–5 days | ⚠️ Atenção | At risk — approaching limit |
| ≥ 6 days | 🔴 Crítico | Critical — likely to breach indicator |

> **Finalizado** orders appear in the base as volume context only — they are not subject to criticality classification.

### Step 6 — Dashboard (Excel)
- KPI cards: total orders in window, finalized, in-route, pending, failed attempts
- Criticality breakdown: No prazo / Atenção / Crítico by region and state
- Interactive slicers: status, region, carrier, occurrence flag
- Auto-refresh: replace base tab → all pivot tables update automatically

---

## 📊 Key Results

| Metric | Value |
|--------|-------|
| Daily orders monitored | **3,200 – 3,800** |
| Monitoring window | **Rolling D-6 (6 days)** |
| States covered | **RN, MA, CE, PB** |
| Criticality tiers | **3 (No prazo / Atenção / Crítico)** |
| Update frequency | **Daily** |

---

## 🔄 How to Update the Data (POP)

The dashboard was designed so that **any team member** can update it without technical knowledge. Two scenarios:

**Scenario 1 — Same folder, new file (recommended)**
1. Save the new CSV in the standard folder, replacing the old file
2. Open the dashboard
3. Go to **Data → Refresh All**

**Scenario 2 — New path or different file**
1. Go to **Data → Queries & Connections**
2. Double-click the final query
3. In the **Source** step, click the gear icon and select the new file
4. Check the **Navigation** step — confirm the correct tab/table is selected
5. Validate that all expected columns are present in `Table.TransformColumnTypes`
6. Click **Close & Load**, then **Data → Refresh All**

**Final validation checklist:**
- [ ] Pivot tables updated correctly
- [ ] No "invalid field" errors
- [ ] Slicers responding
- [ ] KPI cards calculating correctly

> **Common errors:** Column name changed in source file → fix in `TransformColumnTypes` step. Slicer not responding → ensure the original query name was preserved (never create a new query to replace an existing one).

---

## 🗂️ Project Structure

```
logistics-dashboard/
│
├── data/
│   └── sample_orders.csv              # Synthetic dataset mimicking TMS export structure
│
├── excel/
│   └── dashboard_template.xlsx        # Dashboard with Power Query + Pivots (anonymized)
│
└── README.md
```

---

## 🛠️ Tools & Skills

- **Excel** — dashboard design, pivot tables, slicers, KPI cards, data model
- **Power Query** — ETL pipeline, status normalization, table merging, conditional columns

---

## 💡 What I Learned

The most critical design decision was defining what **Finalizado** really means. Initially, statuses like "Out for delivery" and "Collected" were grouped as finalized — but these orders can still return as failed attempts. Restricting Finalizado to only confirmed final outcomes ("Delivered", "Picked up") made the open order monitoring accurate and operationally trustworthy.

The **D-6 rolling window** was also key: monitoring only the last 6 days ensures the dashboard focuses exclusively on orders that can still be acted upon — orders beyond 7 days have already exited the performance indicator window regardless of outcome.

---

## 📌 Note on Data

All data in this repository is **anonymized or synthetically generated** to preserve confidentiality. The dashboard logic, Power Query transformations, classification rules, and status mappings reflect the actual production system.
