
## 🗺️ Data Model Architecture (Star Schema)
The data model uses a central **Fact Table** (Master Task) connected to specialized **Dimension Tables** (Hospitality, Event, Budget Variance, and Outstanding Payment) using unique relational keys.

                  ┌───────────────────────────────┐
                  │          Master Task          │ 
                  │         (Central Fact)        │
                  └───────────────┬───────────────┘
                                  │
         ┌────────────────────────┼────────────────────────┐
         ▼                        ▼                        ▼
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│   Hospitality   │      │     Events      │      │ Budget Variance │
│  (Hotel/Travel) │      │  (Food & Guests)│      │  (Financials)   │
└─────────────────┘      └─────────────────┘      └─────────────────┘
                                  │
                                  ▼
                        ┌───────────────────┐
                        │Outstanding Payment│
                        │ (Cash Flow Risk)  │
                        └───────────────────┘

------------------------------
## 📑 Detailed Table Schemas 
1. **Table:** Master Task **(Fact Table)**

**Description:** The operational backbone of the project. It tracks every task, milestone, deadline, and high-level expenditure across the entire event lifecycle.

| Column Name | Data Type | Key Type | Sample Value | Business Rules / Constraints |
|---|---|---|---|---|
| Task ID | Alphanumeric | Primary Key | T001, R001 | Unique identifier for each operational task or resource row. |
| Vendor Name | Text | Foreign Key | Grand Palace | Tracks the vendor assigned. Links to procurement accounts. |
| Category/Event | Text | Foreign Key | Catering | Used to map expenses back to the Event and Budget Variance sheets. |
| Due Date | Date | - | 2026-11-15 | Target deadline formatted as YYYY-MM-DD. |
| Estimated Cost | Currency | - | ₹50,000 | Initial budget allocation. Must be a numeric value (cannot be text like "NA"). |
| Actual Cost | Currency | - | ₹45,000 | Final contract value. Populated via lookups from dimensional tables where applicable. |
| Payment Made | Currency | - | ₹20,000 | Total capital already transferred to the vendor. |
| Payment Pending | Currency | - | ₹25,000 | Calculated field: Actual Cost - Payment Made. Represents remaining liability. |
| Days Pending | Integer | - | 56 | Calculated field: Due Date - TODAY(). Tracking window to deadline. |

------------------------------
 2. **Table:** Hospitality **(Dimension Table)**
 
**Description:** Tracks travel logistics, room blocks, and specific accommodations for outstation family members and VIP guests.

| Column Name | Data Type | Key Type | Sample Value | Business Rules / Constraints |
|---|---|---|---|---|
| Task ID | Alphanumeric | Foreign Key | R001 | Links directly to the accommodation rows in the Master Task table. |
| Relative Name | Text | - | Uncle Ahmed Khan | Primary guest contact identifier. |
| Check-in Date | Date | - | 2026-12-03 | Target arrival date. |
| Check-out Date | Date | - | 2026-12-07 | Target departure date. |
| No of Days | Integer | - | 4 | Calculated field: Check-out Date - Check-in Date. |
| Room Rate/Night | Currency | - | ₹2,500 | Standard negotiated price per hotel room night. |
| Actual Acc. Cost | Currency | - | ₹10,000 | Calculated field: No of Days * Room Rate/Night. Feeds into Master Task. |

------------------------------
3. **Table:** Event **(Dimension Table)**

**Description:** Handles catering metrics, RSVP conversion percentages, and variable costs for sub-events.

| Column Name | Data Type | Key Type | Sample Value | Business Rules / Constraints |
|---|---|---|---|---|
| Event Name | Text | Foreign Key | Nikah Lunch | Must exactly match the category names used in the Master Task table. |
| Estimated Guests | Integer | - | 200 | Baseline capacity planning number. |
| Confirmed Guests | Integer | - | 190 | Active RSVP head-count used to lock final vendor contracts. |
| Cost Per Plate | Currency | - | ₹700 | Fixed variable cost item per attending guest. |
| Total Food Cost | Currency | - | ₹1,33,000 | Calculated field: Confirmed Guests * Cost Per Plate. |

------------------------------
4. **Table:** Budget Variance **(Dimension Summary Table)**

**Description:** High-level executive reporting sheet used to display financial control metrics and pinpoint budget leakages.

| Column Name | Data Type | Key Type | Sample Value | Business Rules / Constraints |
|---|---|---|---|---|
| Category | Text | Foreign Key | Catering | Unique category groupings. |
| Estimated Cost | Currency | - | ₹2,00,000 | Dynamic aggregation: Calculated using SUMIF from Master Task. |
| Actual Cost | Currency | - | ₹2,10,000 | Dynamic aggregation: Calculated using SUMIF from Master Task. |
| Variance | Currency | - | -₹10,000 | Calculated field: Estimated Cost - Actual Cost. |
| Status | Text | - | Over Budget | Evaluated logic: Positive = Under Budget, Negative = Over Budget, 0 = On Track. |

------------------------------
5. **Table:** Outstanding Payment **(Dimension / Dynamic View)**

**Description:** A liquidity risk-management ledger highlighting active debt obligations, payment deadlines, and cash flow threats.

| Column Name | Data Type | Key Type | Sample Value | Business Rules / Constraints |
|---|---|---|---|---|
| Vendor Name | Text | - | Grand Palace | Filtered array pull using =FILTER() where pending balance > 0. |
| Total Owed | Currency | - | ₹25,000 | Matches outstanding balances remaining. |
| Due Date | Date | - | 2026-10-15 | Target financial deadline. |
| Days Remaining | Integer | - | -5 | Calculated countdown. Negative metrics flag immediate Overdue risks. |

------------------------------
## 📈 Core Analytical Formulas & Business Logic 
**1. Cost Variance Analysis**

Budget Variance = Estimated Cost - Actual Cost 

* **Business Translation:** A positive value indicates cost-saving performance. A negative value indicates a cost overrun that requires immediate executive attention or budget re-allocation from other categories.

## 2. Variable Supply Chain Forecasting

Total Event Food Cost = Confirmed RSVPs x Per-Plate Rate 

* **Business Translation**: Eliminates material procurement waste by tying final variable vendor costs dynamically to shifting customer demand (RSVP counts) instead of rigid initial estimations.

## 3. Aging Debt Calculation (Cash Flow Risk)

Days Remaining = Due Date - TODAY() 

* **Business Translation:** Segregates liabilities into risk tiers (Overdue, Due within 7 Days, Safe) to manage short-term working capital effectively.

------------------------------
