# Manufacturing Operations and Supply Chain Performance Analytics

**Power BI | Power Query M | DAX | Production, cost, maintenance, fulfilment and supplier analytics**

Nine dirty operational tables cleaned, modeled and turned into a two-page Power BI report with 14 reusable DAX measures across five business objectives.

| | |
|---|---|
| **Tools** | Power BI Desktop, Power Query (M), DAX |
| **Data** | `Manufacturing_Highly_Dirty_Multi_Table_Dataset.xlsx`, 9 related tables |
| **Deliverable** | 2-page interactive report, 10 executive KPI cards, 14 DAX measures |

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Dataset Overview](#2-dataset-overview)
3. [Data Quality Assessment](#3-data-quality-assessment)
4. [Data Cleaning and Error Correction](#4-data-cleaning-and-error-correction)
5. [Feature Engineering](#5-feature-engineering)
6. [Data Modeling and Relational Schema](#6-data-modeling-and-relational-schema)
7. [DAX Measures](#7-dax-measures)
8. [Business Objectives](#8-business-objectives)
9. [Data Visualization and Interpretation](#9-data-visualization-and-interpretation)
10. [Validation and Quality Assurance](#10-validation-and-quality-assurance)
11. [Key Findings and Insights](#11-key-findings-and-insights)
12. [Business Recommendations](#12-business-recommendations)
13. [Conclusion](#13-conclusion)

---

## 1. Project Overview
Decisions on output, cost, downtime, orders and suppliers need one trusted view. This project cleans nine related tables, models them in Power BI and answers five objectives:

| # | Objective |
|---|---|
| 1 | Production efficiency and output performance |
| 2 | Manufacturing cost optimization and material waste |
| 3 | Machine reliability and maintenance performance |
| 4 | Customer fulfilment and order value |
| 5 | Supplier performance and sourcing optimization |

## 2. Dataset Overview

| Table | Contents |
|---|---|
| `Customers` | Customer details for sales and fulfilment |
| `Suppliers` | Material category, lead time, grade, defect rate |
| `Products` | Attributes, standard material cost, standard price |
| `Employees` | Employees linked to operations |
| `Machines` | Equipment information |
| `Production_Orders` | Dates, plant, shift, planned quantity, due date, status |
| `Production_Logs` | Planned and actual output, scrap, downtime, material and labour cost |
| `Sales_Orders` | Ordered and delivered quantity, unit price, delivery dates, status |
| `Maintenance_Logs` | Machine, maintenance type, failure reason, downtime, cost, status |

## 3. Data Quality Assessment

| Check | Risk if unresolved |
|---|---|
| Missing and incomplete values | Understated totals |
| Inconsistent labels and statuses | Split categories in visuals |
| Wrong data types, text inside numeric columns | Failed aggregations |
| Duplicate records and identifier mismatches | Broken relationships |
| Inconsistent date formats | Faulty time-series grouping |
| Referential integrity across tables | Incomplete cross-table analysis |

## 4. Data Cleaning and Error Correction

All steps run in Power Query M, so cleaning is repeatable on every refresh. Order of execution: replace placeholders, convert types, remove error rows.

**Replace text placeholders** such as "Not Available" with nulls before type conversion.
```powerquery
= Table.ReplaceValue(
    #"Previous Step",
    "Not Available",
    null,
    Replacer.ReplaceValue,
    {
        "material_qty_used",
        "material_cost",
        "labor_cost"
    }
)
```

**Convert to numeric types** with a culture-invariant setting for aggregation and DAX.
```powerquery
= Table.TransformColumnTypes(
    #"Previous Step",
    {
        {"material_qty_used", type number},
        {"material_cost", type number},
        {"labor_cost", type number},
        {"maintenance_cost", type number},
        {"defect_rate", type number}
    },
    "en-US"
)
```

**Remove rows that failed conversion.**
```powerquery
= Table.RemoveRowsWithErrors(
    #"Changed Type",
    {
        "material_qty_used",
        "material_cost",
        "labor_cost",
        "maintenance_cost",
        "defect_rate"
    }
)
```

## 5. Feature Engineering

**Production status:** classifies each record against its planned quantity.
```powerquery
= Table.AddColumn(
    #"Changed Type",
    "Production_Status",
    each
        if [actual_qty] = null or [planned_qty] = null then null
        else if [actual_qty] > [planned_qty] then "Surplus"
        else if [actual_qty] < [planned_qty] then "Deficit"
        else "Target Met",
    type text
)
```

**Quarter name:** reporting quarter from the order date.
```powerquery
= Table.AddColumn(
    #"Previous Step",
    "Quarter_Name",
    each
        if [order_date] = null then null
        else "Qtr " & Text.From(Date.QuarterOfYear([order_date])),
    type text
)
```

**Expense tier:** labour cost threshold of 50,000.
```powerquery
= Table.AddColumn(
    #"Previous Step",
    "Expense_Tier",
    each
        if [labor_cost] = null then null
        else if [labor_cost] > 50000 then "High Expense"
        else "Standard Expense",
    type text
)
```

## 6. Data Modeling and Relational Schema
Nine tables are related on customer, supplier, product, production order, machine and employee keys. Cardinality, key uniqueness and filter direction were checked so cross-table measures return the intended results.

<img width="1920" height="1080" alt="Power BI model view of the nine manufacturing tables and their relationships" src="https://github.com/user-attachments/assets/4921308a-ef32-4fcc-b793-4e61572ee0cd" />

## 7. DAX Measures

### Production efficiency
```powerquery
Total Planned Quantity = SUM(Production_Logs[planned_qty])

Total Actual Quantity = SUM(Production_Logs[actual_qty])

Production Gap = [Total Actual Quantity] - [Total Planned Quantity]

Production Achievement % =
DIVIDE([Total Actual Quantity], [Total Planned Quantity], 0)
```

### Cost and material waste
```powerquery
Total Scrap Quantity = SUM(Production_Logs[scrap_qty])

Scrap Rate % =
DIVIDE([Total Scrap Quantity], [Total Actual Quantity], 0)

Total Material Cost = SUM(Production_Logs[material_cost])

Total Labour Cost = SUM(Production_Logs[labor_cost])
```

### Machine reliability
```powerquery
Total Maintenance Cost = SUM(Maintenance_Logs[maintenance_cost])

Total Downtime = SUM(Maintenance_Logs[downtime_hours])
```

### Customer fulfilment
```powerquery
Total Order Value =
SUMX(Sales_Orders, Sales_Orders[ordered_qty] * Sales_Orders[unit_price])

Total Quantity Delivered = SUM(Sales_Orders[delivered_qty])
```

### Supplier performance
```powerquery
Average Supplier Lead Time = AVERAGE(Suppliers[avg_lead_time_days])

Average Supplier Defect Rate = AVERAGE(Suppliers[defect_rate])
```

Format `Production Achievement %` and `Scrap Rate %` as percentages. `DIVIDE` with a default of 0 prevents divide-by-zero errors. `Total Order Value` is ordered quantity at unit price, not revenue collected or profit.

## 8. Business Objectives

| # | Objective | Key question | Measures |
|---|---|---|---|
| 1 | Production efficiency | Is actual output meeting plan, by plant and shift? | Planned, Actual, Production Gap, Achievement % |
| 2 | Cost and material waste | How do material cost, labour cost and scrap vary by shift? | Material Cost, Labour Cost, Scrap Quantity, Scrap Rate % |
| 3 | Machine reliability | Which machines carry the most downtime and maintenance cost? | Total Downtime, Total Maintenance Cost |
| 4 | Customer fulfilment | How do ordered and delivered quantities compare over time? | Total Order Value, Quantity Delivered, order status |
| 5 | Supplier performance | Which suppliers have long lead times or high defect rates? | Average Lead Time, Average Defect Rate, grade |
<img width="587" height="335" alt="Executive KPI&#39;s" src="https://github.com/user-attachments/assets/9ee92e6b-32ae-4ca8-b364-834a667e5bcf" />
<img width="585" height="331" alt="Business Operations Performance Analytics" src="https://github.com/user-attachments/assets/6ac462a6-f564-46bd-892e-7b2bd28ee473" />

Interpretation rules applied:
- Judge cost changes alongside output so lower spend is not mistaken for efficiency.
- High maintenance spend alone does not prove a poor machine. Age, usage and criticality matter.
- Order value is not profit.
- Confirm the supplier defect-rate definition before ranking suppliers.

## 9. Data Visualization and Interpretation

**Page 1, Executive KPIs:** 10 cards covering Total Planned Quantity, Total Actual Quantity, Production Achievement %, Production Gap, Total Scrap Quantity, Scrap Rate %, Total Material Cost, Total Labour Cost, Total Maintenance Cost and Total Order Value.

**Page 2, Business Operations Performance:**

| Objective | Visual | Comparison |
|---|---|---|
| Production efficiency | Line and clustered column | Planned vs actual by plant and shift |
| Cost and material waste | Stacked column | Material and labour cost by shift, with scrap rate |
| Machine reliability | Clustered bar | Downtime and maintenance cost by machine |
| Customer fulfilment | Line and clustered column | Ordered, delivered and order value over time |
| Supplier performance | Line and clustered column | Lead time and defect rate by supplier |

Axis scales, percentage formats, titles and tooltips are set explicitly because measures use different units.

## 10. Validation and Quality Assurance
- DAX totals reconciled with source-table totals
- Percentage measures formatted correctly
- Relationships and filters tested across all nine tables
- Date fields group correctly in time visuals
- Impact of removed error rows reviewed
- Planned quantity source (`Production_Logs` vs `Production_Orders`) clearly identified
- Order value never presented as profit

## 11. Key Findings and Insights
The report is a validated analytical framework. No quantified improvement is claimed without verified results. It surfaces:

| Area | What it reveals |
|---|---|
| Production | Planned vs actual gaps by plant and shift |
| Cost and waste | Link between volume, cost and scrap |
| Maintenance | Machines with high downtime or maintenance spend |
| Fulfilment | Ordered vs delivered gaps and order value trend |
| Suppliers | Longer lead times and higher defect rates |

## 12. Business Recommendations
1. **Improve production planning:** investigate recurring gaps across plants and shifts.
2. **Monitor manufacturing costs:** read spend alongside volume.
3. **Reduce scrap:** verify causes in high-scrap shifts or products before changing processes.
4. **Strengthen maintenance planning:** prioritize by downtime, history and criticality.
5. **Improve customer fulfilment:** follow up on incomplete orders and delivery gaps.
6. **Review supplier performance:** use validated lead-time and defect-rate measures for sourcing reviews.
7. **Maintain data quality:** investigate recurring conversion errors and document excluded records.

## 13. Conclusion
Power Query cleaning, a relational model and explicit DAX measures turned nine raw tables into a validated report that supports decisions on production, cost, maintenance, fulfilment and sourcing.

