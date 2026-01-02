# Make vs. Buy: Supply Chain Decision Support Dashboard

## 📌 Project Overview
This project presents a Power BI dashboard designed to assist supply chain managers in the critical "Make vs. Buy" decision-making process. By integrating internal manufacturing resource estimates with external Request for Proposal (RFP) responses, the dashboard provides a comparative cost analysis to determine whether products should be manufactured in-house or outsourced to vendors.

## 🚀 Key Business Questions
* **Cost Comparison:** Is it more cost-effective to manufacture specific products internally or purchase them from external suppliers?
* **Vendor Selection:** Among the external options, which vendor offers the most competitive pricing per unit?
* **Resource Allocation:** What are the internal labor and machine hour requirements if production is kept in-house?

## 🛠️ Data Model & Architecture
The dashboard is built upon a star schema relational model connecting four key datasets.

* **Model View:**
    <br><img width="800" alt="Model View" src="https://github.com/user-attachments/assets/e25c43e6-03a3-404a-87d9-f09c90201181" /><br>
* **Table View:**
    <br><img width="800" alt="Table View" src="https://github.com/user-attachments/assets/5b6570ac-9f04-4916-9c7d-c711d7a023e0" /><br>

**Key Tables:**
1.  **Quotes (Fact Table):** Contains RFP responses from vendors, including minimum order quantities and unit prices.
2.  **Internal Mfg Estimates (Fact Table):** Details the internal resource consumption (labor/machine hours) required for each product.
3.  **Mfg Resources (Dimension):** Specific cost rates ($/hr) for different internal resources (e.g., Labor, Machine A, Machine B).
4.  **Product Dimension (Dimension):** Master list of products and their categories.

## 📊 Dashboard Pages & Visualization Logic

### Page 1: Executive Cost Summary
* **Purpose:** High-level overview of the financial impact of making vs. buying across the entire product portfolio.
* **Key Visuals:**
    * **KPI Cards:** Total Estimated Internal Cost vs. Total External Vendor Cost.
    * **Bar Chart (Cost Variance):** Visualizes the delta between internal and external costs by Product Category.
    * **Slicers:** Filter by Product Family or specific SKU.

<img width="1000" alt="Make Versus Buy" src="https://github.com/user-attachments/assets/d7ebdab9-065d-4733-990c-fd721d40ee83" />

### Page 2: Vendor Analysis & Comparison
* **Purpose:** Detailed evaluation of external supplier performance and pricing structures.
* **Key Visuals:**
    * **Matrix Table:** Side-by-side comparison of unit prices from different vendors (e.g., *Vendor A* vs. *Vendor B*) for each product.
    * **Scatter Plot:** Correlation between "Minimum Order Quantity" and "Unit Price" to identify volume discount opportunities.
    * **Highlight:** Conditional formatting marks the lowest bid for each item in green.

<img width="1000" alt="Supplier Selection" src="https://github.com/user-attachments/assets/3082b17b-0a42-4dba-b344-9629be2569e5" />

### Page 3: Internal Resource Feasibility & Scenario Planning
* **Purpose:** Deep dive into the operational requirements of in-house production and scenario modeling.
* **Key Visuals:**
    * **Stacked Bar Chart:** Breakdown of internal costs by resource type (Labor vs. Machine Overhead).
    * **Gauge Charts:** Total required manufacturing hours vs. available internal capacity.
    * **Table:** Itemized list of resource consumption (Hours/Unit) for selected products.

<img width="1000" alt="Scenario Analysis" src="https://github.com/user-attachments/assets/f7d7857d-a584-4c81-a9cc-d444ac35c9f5" />

## 💻 Tech Stack & Key DAX Measures
* **Tool:** Microsoft Power BI
* **Advanced Features:** Field Parameters, What-If Parameters (`GENERATESERIES`).

### Core DAX Logic
The following measures drive the core decision logic, accounting for volume tiers, yield rates, and NRE:

```dax
/* 1. Dynamic Buy Cost Calculation 
   Finds the lowest vendor quote valid for the selected volume, 
   adjusts for Yield Rate, and adds NRE. */
Buy Scenario Full Cost = 
MINX(
    FILTER(Quotes, Quotes[Volume] <= 'Scenario Volume'[Scenario Volume Value]), 
    ([Scenario Volume Value] * Quotes[Unit_Cost] / Quotes[Yield Rate]) + Quotes[Non_recurring_expenses]
)

/* 2. Scenario Volume Generator
   Creates the underlying series for the What-If parameter slider (1k to 100k units). */
Scenario Volume = GENERATESERIES(1000, 100000, 500)

/* 3. Cost Avoidance (Savings)
   Calculates the absolute financial impact of choosing the optimal path. */
Cost Avoidance = ABS([Make Scenario Full Cost] - [Buy Scenario Full Cost])

/* 4. Decision Logic
   Returns the binary recommendation text based on the cost comparison. */
Make vs Buy = 
IF(
    ISBLANK([Make Scenario Full Cost]), 
    BLANK(), 
    IF([Make Scenario Full Cost] >= [Buy Scenario Full Cost], "Buy", "Make")
)



