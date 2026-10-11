
# Manufacturing Operations & Supply Chain Performance Analytics

### A Multi-Table Data Analytics Project for Production Efficiency, Cost Optimization, Machine Reliability, Customer Fulfilment, and Supplier Performance

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Dataset Overview](#2-dataset-overview)
3. [Data Quality Assessment](#3-data-quality-assessment)
4. [Data Cleaning and Error Correction](#4-data-cleaning-and-error-correction)
5. [Feature Engineering](#5-feature-engineering)
6. [Data Modeling and Relational Schema](#6-data-modeling-and-relational-schema)
7. [DAX Measures](#7-dax-measures)
8. [Exploratory Data Analysis](#8-exploratory-data-analysis)
9. [Core Business Analysis](#9-core-business-analysis)
   - [Production Efficiency and Output Performance](#91-production-efficiency-and-output-performance)
   - [Manufacturing Cost Optimization and Material Waste](#92-manufacturing-cost-optimization-and-material-waste)
   - [Machine Reliability and Maintenance Performance](#93-machine-reliability-and-maintenance-performance)
   - [Customer Fulfilment and Order Value](#94-customer-fulfilment-and-order-value)
   - [Supplier Performance and Sourcing Optimization](#95-supplier-performance-and-sourcing-optimization)
10. [Data Visualization and Interpretation](#10-data-visualization-and-interpretation)
11. [Validation and Quality Assurance](#11-validation-and-quality-assurance)
12. [Key Findings and Insights](#12-key-findings-and-insights)
13. [Business Recommendations](#13-business-recommendations)
14. [Conclusion](#14-conclusion)

---

## 1. Project Overview

Manufacturing businesses need reliable information about production output, operating costs, equipment downtime, customer orders, and supplier performance to make effective operational decisions.

This project uses **Microsoft Power BI** to analyse manufacturing and supply chain operations across nine related tables. The objective is to organise operational data into a structured analytical model, develop business-focused DAX measures, and present performance indicators through interactive dashboards.

The analysis is organised around five business objectives:

1. Production efficiency and output performance.
2. Manufacturing cost optimization and material waste.
3. Machine reliability and maintenance performance.
4. Customer fulfilment and order value.
5. Supplier performance and sourcing optimization.

The project demonstrates practical application of Power Query, relational data modeling, DAX calculations, and interactive data visualization to support evidence-based operational decision-making.

### Tools and Technologies

- Microsoft Power BI Desktop
- Power Query
- DAX (Data Analysis Expressions)
- Data modeling and table relationships
- Interactive reports, KPI cards, charts, and tooltips

## 2. Dataset Overview

The project uses the `Manufacturing_Highly_Dirty_Multi_Table_Dataset.xlsx` dataset, which contains nine related tables representing manufacturing, inventory-related material usage, maintenance, sales, customer, employee, machine, product, and supplier information.

| Table | Purpose |
|---|---|
| `Customers` | Customer information for analysing sales and fulfilment. |
| `Suppliers` | Supplier details, material categories, lead times, grades, and defect rates. |
| `Products` | Product attributes, standard material costs, and standard selling prices. |
| `Employees` | Employee information associated with operational activities. |
| `Machines` | Machine information for equipment and operational analysis. |
| `Production_Orders` | Production order dates, products, plants, shifts, planned quantities, targets, due dates, and statuses. |
| `Production_Logs` | Planned and actual output, scrap, downtime, cycle time, material usage, material costs, labour costs, and quality results. |
| `Sales_Orders` | Customer orders, products, quantities, unit prices, requested delivery dates, actual delivery dates, delivered quantities, and order statuses. |
| `Maintenance_Logs` | Maintenance dates, machine references, maintenance types, failure reasons, downtime, costs, technicians, and statuses. |

The tables provide complementary perspectives on operational performance. Their relationships enable analysis across production, maintenance, sales, customers, products, employees, machines, and suppliers where the data model supports the required filter paths.

## 3. Data Quality Assessment

Before developing the report, the dataset was assessed for data quality issues that could affect calculations, relationships, and visual interpretation.

The assessment focused on:

- Missing and incomplete values.
- Inconsistent categorical labels and status values.
- Incorrect or inconsistent data types.
- Numeric columns containing text values.
- Potential duplicate records and identifier consistency.
- Date fields requiring consistent date formats.
- Referential integrity between related tables.
- Values that could produce calculation or aggregation errors.

This assessment helped identify data preparation requirements before developing the Power BI analytical model.



## 4. Data Cleaning & Error Correction

**Power Query | M Code**

* **Numeric Data Type Conversion:** Converted cost and quantity columns to numeric types for aggregation and DAX calculations.
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

* **Removal of Numeric Conversion Errors:** Removed rows that returned errors after numeric type conversion in the specified fields.
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

* **Replacement of Text Placeholders:** Where text placeholders such as `"Not Available"` occur in numeric columns, replace them with `null` before conversion.
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

## 5. Feature Engineering

**Power Query | M Code | Calculated Columns**

* **Production Status Classification:** Created a column to classify production output against planned quantity.
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

* **Quarter Extraction:** Derived a reporting quarter from the order date.
  ```powerquery
  = Table.AddColumn(
      #"Previous Step",
      "Quarter_Name",
      each
          if [order_date] = null then null
          else "Qtr " &
              Text.From(Date.QuarterOfYear([order_date])),
      type text
  )
  ```

* **Labour Expense Classification:** Categorized labour cost using a threshold of 50,000.
  ```powerquery
  = Table.AddColumn(
      #"Previous Step",
      "Expense_Tier",
      each
          if [labor_cost] = null then null
          else if [labor_cost] > 50000
              then "High Expense"
              else "Standard Expense",
      type text
  )
  ```

## 6. Data Modeling and Relational Schema

The Power BI data model brings together nine related tables to support analysis across different operational areas.

The model includes:

- Customer and sales order information.
- Product and production information.
- Production orders and production logs.
- Machine and maintenance records.
- Employee information associated with production activities.
- Supplier information for sourcing performance analysis.

Relationships use relevant identifiers, including customer, supplier, product, production order, machine, and employee identifiers, where the corresponding keys are available.

A well-structured model helps Power BI apply filters consistently and reduces the need to repeat calculations across individual report visuals.

Relationship direction, cardinality, key uniqueness, and filter propagation are important to the accuracy of cross-table analysis. Measures that combine fields from different tables should be validated to ensure that the model returns the intended results.


## 7. DAX Measures

DAX measures provide reusable calculations for evaluating production, costs, maintenance, sales, and supplier performance.

The following examples illustrate the principal calculations used in the project. They should match the final measures in the Power BI model.

### Production Performance

* **Total Planned Quantity (DAX Measure / Aggregation):** Aggregates master production targets directly from the operational planning dataset to establish baseline volume milestones:

  ```powerquery
  Total Planned Quantity = SUM(Production_Orders_tbl[planned_qty])
  ```

* **Total Actual Quantity (DAX Measure / Aggregation):** Sums up the true physical volume output recorded directly on the factory floor logs across production lines:

  ```powerquery
  Total Actual Quantity = SUM(Production_Logs_tbl[actual_qty])
  ```

* **Production Gap (DAX Measure / Math Logic):** Calculates the exact volumetric variance surplus generated by subtracting planned milestones from actual floor outputs:

  ```powerquery
  Production Gap = [Total Actual Quantity] - [Total Planned Quantity]
  ```

* **Production Achievement % (DAX Measure / Divide Logic):** Evaluates the structural target fulfillment rate using the safe `DIVIDE` function to completely prevent runtime divide-by-zero errors:

  ```powerquery
  Production Achievement % = 
  DIVIDE(
      [Total Actual Quantity],
      [Total Planned Quantity],
      0
  )
  ```
  Format `Production Achievement %` as a percentage in Power BI.

---

### Quality and Material Waste

* **Total Scrap Quantity (DAX Measure / Aggregation):** Tracks the cumulative physical volume of material discarded during raw manufacturing runs:

  ```powerquery
  Total Scrap Quantity = SUM(Production_Logs_tbl[scrap_qty])
  ```

* **Scrap Rate % (DAX Measure / Divide Logic):** Measures the exact proportion of wasted material relative to total gross floor volume output to evaluate quality drift:

  ```powerquery
  Scrap Rate % = 
  DIVIDE(
      [Total Scrap Quantity],
      [Total Actual Quantity],
      0
  )
  ```
  Format `Scrap Rate %` as a percentage.

---

### Manufacturing Costs

* **Total Material Cost (DAX Measure / Aggregation):** Aggregates the gross financial capital deployed toward physical raw material inputs across all operations:

  ```powerquery
  Total Material Cost = SUM(Production_Logs_tbl[material_cost])
  ```

* **Total Labour Cost (DAX Measure / Aggregation):** Calculates total labor expenditures, leveraging the newly unblocked numeric column structure to evaluate shift costs:

  ```powerquery
  Total Labour Cost = SUM(Production_Logs_tbl[labor_cost])
  ```

---

### Machine Reliability and Maintenance

* **Total Maintenance Cost (DAX Measure / Aggregation):** Sums up all corrective and preventative financial repair investments across individual machinery assets:

  ```powerquery
  Total Maintenance Cost = SUM(Maintenance_Logs_tbl[maintenance_cost])
  ```

* **Total Downtime (DAX Measure / Aggregation):** Tracks the cumulative operational breakdown hours stalling floor throughput to pinpoint mechanical vulnerabilities:

  ```powerquery
  Total Downtime = SUM(Maintenance_Logs_tbl[downtime_hours])
  ```

---

### Customer Fulfilment and Order Value

* **Total Order Value (DAX Measure / Iterator Logic):** Executes row-by-row matrix math across the sales database to establish the total gross commercial demand footprint at active unit prices:

  ```powerquery
  Total Order Value = 
  SUMX(
      Sales_Orders_tbl,
      Sales_Orders_tbl[ordered_qty] * Sales_Orders_tbl[unit_price]
  )
  ```
  Total Order Value represents the value of ordered quantities at the recorded unit prices. It should not be interpreted as revenue actually collected or as profit without additional supporting data.

* **Total Quantity Delivered (DAX Measure / Aggregation):** Aggregates the physical product volumes successfully shipped out and accepted by distribution channels:

  ```powerquery
  Total Quantity Delivered = SUM(Sales_Orders_tbl[delivered_qty])
  ```

---

### Supplier Performance

* **Average Supplier Lead Time (DAX Measure / Average Logic):** Computes the mean procurement time window (in days) spanning raw order placement to inbound vendor dock delivery:

  ```powerquery
  Average Supplier Lead Time = AVERAGE(Suppliers_tbl[avg_lead_time_days])
  ```

* **Average Supplier Defect Rate (DAX Measure / Average Logic):** Evaluates incoming component quality by tracking the mean structural defect rates across third-party vendors, utilizing the clean numeric rows unblocked in Power Query:

  ```powerquery
  Average Supplier Defect Rate = AVERAGE(Suppliers_tbl[defect_rate])
  ```
  The supplier defect-rate calculation assumes that the source field is numeric and consistently represented. Its aggregation and percentage formatting should reflect the field's actual definition and scale.

## 8. Exploratory Data Analysis

Exploratory data analysis was used to examine operational performance from multiple perspectives before drawing business conclusions.

The analysis focuses on:

- Comparing planned production with actual output.
- Identifying production gaps across available plants and shifts.
- Examining scrap quantities and material and labour costs.
- Comparing machine downtime with maintenance expenditure.
- Tracking order quantities, delivered quantities, and order value over time.
- Comparing supplier lead times and recorded defect rates.

These comparisons help identify patterns, potential bottlenecks, and areas requiring further investigation. They also provide context for interpreting the KPIs displayed in the dashboards.

## 9. Core Business Analysis

The report addresses five business objectives. Each objective connects operational questions to relevant metrics and visual analysis.

### 9.1 Production Efficiency and Output Performance

**Business objective:** Evaluate whether production operations are meeting planned output expectations.

Key questions:

- How does actual production compare with planned production?
- What is the production achievement percentage?
- Which plants and shifts show the largest production gaps?
- Where should production performance be investigated further?

Relevant metrics:

- Total Planned Quantity
- Total Actual Quantity
- Production Gap
- Production Achievement %

The analysis compares planned and actual quantities across available plant and shift categories. It supports investigation of production shortfalls and identification of operations that may require scheduling, resource, or process improvements.

### 9.2 Manufacturing Cost Optimization and Material Waste

**Business objective:** Understand manufacturing expenditure and assess the relationship between production activity and scrap.

Key questions:

- How much has been recorded as material cost?
- How much has been recorded as labour cost?
- How do material and labour costs vary across shifts?
- What scrap quantities and scrap rates are recorded?

Relevant metrics:

- Total Material Cost
- Total Labour Cost
- Total Scrap Quantity
- Scrap Rate %

Comparing costs and scrap across shifts can help identify areas for investigation. Cost reductions should be evaluated alongside output and quality so that lower expenditure is not mistaken for improved efficiency when production has also declined.

### 9.3 Machine Reliability and Maintenance Performance

**Business objective:** Examine equipment downtime and maintenance expenditure to identify machines that may require attention.

Key questions:

- Which machines account for the most recorded downtime?
- How does maintenance expenditure vary across machines?
- Which machines may warrant further reliability investigation?
- What do maintenance types and failure reasons indicate about recurring issues?

Relevant metrics:

- Total Downtime
- Total Maintenance Cost

The analysis compares downtime and maintenance cost by machine. These metrics provide a starting point for maintenance prioritisation, but higher maintenance spending alone does not prove poor performance. Equipment age, usage, production criticality, and failure history may also be relevant.

### 9.4 Customer Fulfilment and Order Value

**Business objective:** Compare customer order quantities with delivered quantities and monitor the value of orders over time.

Key questions:

- How do ordered quantities compare with delivered quantities?
- How does order value change over time?
- Which order statuses require follow-up?
- Where are delivery dates or quantities indicative of fulfilment delays or shortfalls?

Relevant metrics and fields:

- Ordered Quantity
- Total Quantity Delivered
- Total Order Value
- Order Status
- Requested Delivery Date
- Actual Delivery Date

The analysis supports monitoring of order fulfilment and order value trends. Ordered quantity and delivered quantity should be compared using compatible filters and clearly defined aggregation logic.

Order value is not the same as profit. A reliable profitability analysis would require relevant cost attribution, such as product costs and any additional expenses needed to calculate profit.

### 9.5 Supplier Performance and Sourcing Optimization

**Business objective:** Compare supplier lead times and recorded defect rates to support sourcing decisions.

Key questions:

- How does average supplier lead time vary by supplier?
- How do recorded defect rates compare across suppliers?
- Which suppliers may require a quality or delivery review?
- How can supplier performance inform sourcing and follow-up priorities?

Relevant metrics:

- Average Supplier Lead Time
- Average Supplier Defect Rate
- Supplier Grade
- Material Category

The analysis compares supplier lead time with recorded defect rates and uses supplier grades and material categories as supporting context. Before ranking suppliers, the underlying defect-rate definition, scale, and aggregation method should be confirmed.

## 10. Data Visualization and Interpretation

The Power BI report contains two pages designed for executive review and operational investigation.

### Page 1: Executive KPI's

The executive page presents KPI cards for:

- Total Planned Quantity
- Total Actual Quantity
- Production Achievement %
- Production Gap
- Total Scrap Quantity
- Scrap Rate %
- Total Material Cost
- Total Labour Cost
- Total Maintenance Cost
- Total Order Value

These indicators provide a high-level overview of production output, quality, expenditure, and order value.

### Page 2: Business Operations Performance Analysis

The business operations page brings together five primary visuals:

| Business Objective | Primary Visual | Main Comparison |
|---|---|---|
| Production Efficiency | Line and clustered column chart | Planned and actual production by plant and shift. |
| Manufacturing Cost and Material Waste | Stacked column chart | Material and labour costs by shift, with scrap rate available for context. |
| Machine Reliability | Clustered bar chart | Downtime and maintenance cost by machine. |
| Customer Fulfilment | Line and clustered column chart | Ordered quantity, delivered quantity, and order value over time. |
| Supplier Performance | Line and clustered column chart | Average lead time and recorded defect rate by supplier. |

The visuals are intended to help users identify differences, trends, and areas for further investigation. Because measures can use different units and scales, axis formatting, percentage formatting, tooltips, and titles should be clear enough to prevent misleading comparisons.

## 11. Validation and Quality Assurance

Validation is essential to ensure that the report communicates the data accurately.

The project validation process should cover:

- Checking that numeric measures aggregate the intended columns.
- Confirming that percentage measures are formatted as percentages.
- Comparing DAX totals against corresponding source-table totals.
- Checking relationships and filter behaviour across the nine tables.
- Confirming that date fields group correctly in time-based visuals.
- Reviewing whether removed conversion-error rows materially affect reported results.
- Ensuring that visual titles, axes, units, and tooltips accurately describe the metrics.
- Confirming that order value is not presented as profit.
- Checking that supplier defect rates are interpreted according to their source definition.

Particular attention should be given to cross-table visuals. For example, production logs and production orders contain related but distinct planned-quantity fields; the selected field must be clearly identified when interpreting production achievement.

## 12. Key Findings and Insights

The report is designed to surface evidence-based findings across the five business objectives.

The analysis can help identify:

- Gaps between planned and actual production.
- Differences in production performance across plants and shifts.
- Relationships between production activity, recorded costs, and scrap.
- Machines with comparatively high downtime or maintenance expenditure.
- Differences between ordered and delivered quantities.
- Changes in order value over time.
- Suppliers with comparatively long lead times or higher recorded defect rates.

The significance of each finding depends on the validated results and the context of the operation. No numerical performance improvement or quantified business impact is claimed here without verified analytical results.

## 13. Business Recommendations

The findings from the report can support the following evidence-led actions:

1. **Improve production planning:** Investigate recurring production gaps and compare planned output with actual results across plants and shifts.
2. **Monitor manufacturing costs:** Review material and labour expenditure alongside production volume to distinguish genuine efficiency gains from reduced output.
3. **Reduce scrap:** Investigate shifts, products, or production activities associated with higher scrap rates and verify potential causes before changing processes.
4. **Strengthen maintenance planning:** Review machines with high downtime, maintenance history, and operational importance to help prioritise maintenance resources.
5. **Improve customer fulfilment:** Monitor incomplete orders, delivery dates, and differences between ordered and delivered quantities.
6. **Review supplier performance:** Use validated lead-time and defect-rate measures to identify suppliers for follow-up and sourcing review.
7. **Maintain data quality:** Investigate recurring numeric conversion errors and document excluded records so that future reporting remains transparent and consistent.

These recommendations should be prioritised according to the actual findings, operational constraints, and business impact demonstrated by the report.

## 14. Conclusion

Manufacturing Operations & Supply Chain Performance Analytics demonstrates how Power BI can organise multi-table operational data into a structured analytical model and turn it into business-focused performance reporting.

By combining Power Query transformations, relational data modeling, DAX measures, and interactive visualizations, the project provides a framework for examining production efficiency, manufacturing costs, machine reliability, customer fulfilment, and supplier performance.

The project reflects a practical approach to data analytics: prepare the data, model the relationships, define meaningful metrics, validate the calculations, and present the results in a format that supports business decisions.

The resulting report is intended to help operational stakeholders identify areas for further investigation and make better-informed decisions using manufacturing and supply chain data.
