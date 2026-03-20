# Helpdesk Performance & SLA Analytics

**End-to-end IT support operations dashboard built with SQL Server, Power BI, and Python**

![Dashboard Preview](Screenshots/Page1_Executive_Overview.png)

---

## Business Problem

Large IT companies receive hundreds of support tickets daily across departments — hardware failures, software issues, access requests, network problems. Without structured analysis, IT managers have no visibility into:

- Which ticket categories are breaching SLA deadlines repeatedly
- Which agents are overloaded vs underutilised
- Whether high-priority tickets are actually being resolved faster than low-priority ones
- What time of day and week sees the highest ticket load
- Which departments generate the most unresolved tickets

The result is reactive IT support instead of proactive — problems keep recurring, SLAs get missed, and leadership has no data to make staffing or process decisions.

---

## Solution

An end-to-end analytics dashboard that gives IT managers a single view of support operations — tracking ticket volumes, SLA compliance, agent performance, and department-wise burden across 26 months of data (January 2024 – February 2026).

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python (Faker, Pandas, NumPy) | Synthetic dataset generation |
| SQL Server | Data storage, cleaning, transformation |
| Power BI (DAX) | Interactive dashboard — 3 pages |

---

## Dataset

Synthetic dataset designed to mirror real IT helpdesk operations at a mid-large Indian IT company. All data generated using Python with realistic patterns deliberately embedded.

**4 tables — 8,000+ records:**

| Table | Rows | Description |
|-------|------|-------------|
| tickets | 8,000 | Core fact table — one row per support ticket |
| agents | 20 | Support agent details — L1/L2/L3 teams |
| departments | 8 | Requesting departments with headcount |
| sla_policy | 20 | SLA thresholds by category and priority |

**Realistic patterns embedded in the data:**
- Monday and Friday have higher ticket volumes (post-weekend issues, pre-weekend rushes)
- Network and Hardware categories breach SLA at higher rates (35-37%) vs Access (13%)
- 3 agents (Rajesh Kumar, Rohit Sharma, Amit Verma) carry disproportionate workload — 973, 973, 928 tickets vs team average of 400
- Q1 2025 ticket spike simulating a system migration event
- 23.49% overall SLA breach rate — realistic for a mid-size IT operation

---

## Project Structure

```
Helpdesk-Performance-SLA-Analytics/
│
├── Data/
│   ├── tickets.csv
│   ├── agents.csv
│   ├── departments.csv
│   └── sla_policy.csv
│
├── Python/
│   └── generate_dataset.py
│
├── SQL/
│   └── helpdesk_complete.sql
│
├── PowerBI/
│   └── Helpdesk_Performance_SLA_Analytics.pbix
│
├── Screenshots/
│   ├── Page1_Executive_Overview.png
│   ├── Page2_SLA_Resolution_DeepDive.png
│   └── Page3_Agent_Department_Performance.png
│
└── README.md
```

---

## SQL Scripts

The SQL script (`helpdesk_complete.sql`) is structured in 3 sections:

**Script 01 — Schema Creation & Data Load**
- CREATE TABLE scripts for all 4 tables with PK, FK, CHECK constraints
- BULK INSERT commands for CSV loading
- Referential integrity verification queries

**Script 02 — Data Cleaning & Type Correction**
- Data type fixes (nvarchar → DATETIME using TRY_CONVERT)
- agents.team column fix (money type → VARCHAR L1/L2/L3)
- NULL analysis — distinguishing valid NULLs (open tickets) from invalid ones
- Whitespace trimming on all VARCHAR columns
- Duplicate detection and date logic validation

**Script 03 — Derived Columns & Analytical Queries**
- `resolution_hours` — DATEDIFF in minutes / 60.0 for decimal precision
- `response_hours` — first response time calculation
- `sla_breach_flag` — joined to sla_policy on category + priority to get specific threshold per ticket
- `response_breach_flag` — first response SLA violation
- `day_of_week`, `hour_of_day`, `month_year`, `week_number` — time intelligence columns
- 6 analytical queries covering breach rates, agent rankings (window functions), department burden, peak hour heatmap, monthly trends, FCR rate
- 3 Power BI views — `vw_tickets_full`, `vw_agent_summary`, `vw_dept_summary`

---

## Dashboard — 3 Pages

### Page 1 — Executive Overview

High-level health snapshot for IT leadership.

![Page 1](Screenshots/Page2_SLA_Resolution_DeepDive.png)

**KPI Cards:** Total Tickets · SLA Breach % · Avg Resolution Hrs · Open Backlog · FCR Rate % · Critical Tickets

**Visuals:**
- Monthly Ticket Volume & SLA Breach Trend (dual-axis line chart)
- Ticket Volume by Category (horizontal bar — Network highlighted)
- SLA Compliance vs Breach (donut chart — Compliant/Breached/Open)
- Ticket Volume by Priority Over Time (stacked column)
- Slicers: Date Range · Department · Priority
- Key Insights text box

---

### Page 2 — SLA & Resolution Deep Dive

Where and why SLAs are being breached.

![Page 2](Screenshots/Page2_SLA_Resolution_DeepDive.png)

**KPI Cards:** SLA Breach Rate · Avg Resolution Hrs · Total Breached Tickets · Reopen Rate %

**Visuals:**
- **Peak Hour Heatmap** — Matrix visual with conditional formatting gradient (dark navy → red) showing ticket load by Day × Hour. Monday 11am–1pm is the peak window.
- Avg Resolution Time by Category (amber bar chart — Other category takes longest at 44hrs)
- SLA Breach Rate by Category (red bar — Network 37.5%, Access 13.1%)
- Monthly SLA Breach Rate Trend (line chart with average reference line)
- Top Agents by SLA Breach Rate (bar chart)
- Slicers: Category · Priority

---

### Page 3 — Agent & Department Performance

Staffing decisions — who needs support, who's overloaded.

![Page 3](Screenshots/Page3_Agent_Department_Performance.png)

**KPI Cards:** Total Tickets · Avg Tickets per Agent · Overall Breach Rate · Open Backlog

**Visuals:**
- Agent Performance Summary Table (conditional formatting on Breach % — green to red gradient)
- Ticket Volume by Agent (bar chart — top 3 overloaded agents highlighted in red)
- Total Tickets by Department (teal bar chart — Engineering leads at 1,791)
- Tickets per Employee by Dept (Marketing highlighted — highest burden at 22.97)
- **Agent Workload vs SLA Breach Rate** (scatter chart — agents in top-right quadrant are both overloaded AND high breach rate)
- Slicers: Team (L1/L2/L3) · Department

---

## Key Findings

| Finding | Value | Implication |
|---------|-------|-------------|
| Overall SLA breach rate | 23.49% | Nearly 1 in 4 tickets misses SLA |
| Network category breach rate | 37.5% | Highest — needs priority staffing |
| Access category breach rate | 13.1% | Lowest — process is working well |
| Peak ticket window | Monday 11am–1pm | Staff more agents on Monday mornings |
| Agent overload (top 3) | 973, 973, 928 tickets | vs avg 400 — clear redistribution needed |
| Marketing dept burden | 22.97 tickets/employee | Highest ratio — potential IT training gap |
| Other category resolution | 44 hrs avg | Longest resolution time despite low volume |
| First Call Resolution rate | 95.34% | Strong — minimal ticket reopening |

---

## DAX Measures

Key measures built in Power BI:

```dax
SLA Breach Rate % = 
FORMAT(
    DIVIDE(
        SUMX(vw_tickets_full, IF(vw_tickets_full[sla_breach_flag] = TRUE(), 1, 0)),
        CALCULATE(COUNTROWS(vw_tickets_full), NOT(ISBLANK(vw_tickets_full[sla_breach_flag]))),
        0
    ) * 100, "0.00") & "%"

FCR Rate % = 
FORMAT(
    DIVIDE(
        COUNTROWS(FILTER(vw_tickets_full,
            vw_tickets_full[reopened_flag] = FALSE() &&
            (vw_tickets_full[status] = "Resolved" || vw_tickets_full[status] = "Closed"))),
        COUNTROWS(FILTER(vw_tickets_full,
            vw_tickets_full[status] = "Resolved" || vw_tickets_full[status] = "Closed")),
        0
    ) * 100, "0.00") & "%"

Open Backlog = 
COUNTROWS(FILTER(vw_tickets_full,
    vw_tickets_full[status] = "Open" || vw_tickets_full[status] = "In Progress"))
```

---

## SLA Policy Reference

| Category | Priority | Response SLA | Resolution SLA |
|----------|----------|-------------|----------------|
| Network | Critical | 1 hr | 2 hrs |
| Hardware | Critical | 1 hr | 4 hrs |
| Software | Critical | 1 hr | 6 hrs |
| Access | Medium | 4 hrs | 24 hrs |
| Any | Low | 8 hrs | 72 hrs |

---

## How to Run

**Dataset generation:**
```bash
pip install faker pandas numpy
python Python/generate_dataset.py
```

**SQL setup:**
1. Open SQL Server Management Studio
2. Run `SQL/helpdesk_complete.sql` section by section
3. Import CSVs from `Data/` folder using SSMS Import Flat File wizard
4. Run the views section to create `vw_tickets_full`, `vw_agent_summary`, `vw_dept_summary`

**Power BI:**
1. Open `PowerBI/Helpdesk_Performance_SLA_Analytics.pbix`
2. Update data source to your local SQL Server instance
3. Refresh data

---

## About

Built as part of an independent data analytics portfolio to demonstrate end-to-end DA skills — data design, SQL engineering, Power BI dashboard development, and business storytelling.

**Tools:** SQL Server · Power BI · DAX · Python · Pandas · Faker

**Domain:** IT Operations · Helpdesk Analytics · SLA Management
