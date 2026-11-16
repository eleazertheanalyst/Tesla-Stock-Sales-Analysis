# Tesla Stock Sales Analysis Dashboard (Power BI & DAX)

### Dashboard : [Insert Power BI Service Link Here]
<img width="1267" height="712" alt="desktop" src="https://github.com/user-attachments/assets/70035905-6374-41bf-aeea-6d40f160da55" />

## Project Overview
This project analyzes Tesla used vehicle inventory and stock sales data across models, geographic states, mileage tiers, accident histories, and driver assistance packages.
The dataset required structured data modeling, calculated columns, and disconnected slicer parameter engineering in Power BI to allow stakeholders to dynamically switch between vehicle models (`Cyber Truck`, `Model 3`, `Model S`, `Model X`, `Model Y`)—including dynamic 3D Sketchfab model rendering—and filter by vehicle condition (`No Accidents` vs. `Previously Repaired`)
Using Power BI and DAX, an interactive dark-mode executive dashboard was built to uncover pricing drivers, regional valuation differences, and feature-based price premiums.

## Business Objective
The primary objectives of this analysis were to:
-	Analyze average vehicle pricing, median odometer readings, and average monthly EMI across Tesla models
-	Evaluate vehicle condition distribution (`% Without Accident` vs. `% With Accident`)
-	Identify geographic pricing variations and average mileage across US states
-	Quantify the pricing impact of Driver Assistance Systems (`Full Self-Driving`, `Autopilot`, `Enhanced Autopilot`)
-	Provide actionable insights for inventory pricing, regional stock allocation, and financing strategies

## Steps followed 
### Phase 1: Data Preparation & Modeling (Power BI)
Step 1 : **Data Validation & Table Setup**
- Imported the `Tesla Stock Dataset` into Power BI
- Verified data types across pricing, odometer mileage, monthly EMI, state geography, accident history, and driver assistance columns
- Ensured clean categorical values for model selection and accident status filtering
  
Step 2: **Dynamic Slicer & 3D Model Integration**
- Created disconnected parameter/selector tables (`ModelSelector` and `ModelSelector_Slicer`) to drive the left-hand model slicer navigation (`Cyber Truck`, `Model 3`, `Model S`, `Model X`, `Model Y`)
- Linked `ModelSelector[3D Url]` dynamically via DAX (`Selected3DUrl`) so selecting any vehicle model updates both the 3D Sketchfab vehicle display and all KPI metrics simultaneously

### Phase 2: Calculated Columns & DAX Measures 
Developed calculated columns and dynamic key performance indicators:
- **Odometer Bin (Calculated Column)**
```dax
OdometerBin = 
SWITCH(
    TRUE(),
    'Tesla Stock Dataset'[odometer] <= 5000, "0-5k",
    'Tesla Stock Dataset'[odometer] <= 10000, "5k-10k",
    'Tesla Stock Dataset'[odometer] <= 15000, "10k-15k",
    'Tesla Stock Dataset'[odometer] <= 20000, "15k-20k",
    'Tesla Stock Dataset'[odometer] <= 25000, "20k-25k",
    'Tesla Stock Dataset'[odometer] <= 30000, "25k-30k",
    'Tesla Stock Dataset'[odometer] <= 35000, "30k-35k",
    'Tesla Stock Dataset'[odometer] <= 40000, "35k-40k",
    "40k+"
)
```
- **Selected Model**
```dax
SelectedModel = SELECTEDVALUE(ModelSelector_Slicer[ModelCode], "ALL")
```
- **Selected 3D Url**
```dax
Selected3DUrl = 
IF(
    [SelectedModel] = "ALL",
    // Default Sketchfab embed URL when nothing is selected
    "[https://sketchfab.com/models/587a0833e60f465090145b139f6c1bfc/embed](https://sketchfab.com/models/587a0833e60f465090145b139f6c1bfc/embed)",
    // Get the 3D URL from ModelSelector table for the selected model
    CALCULATE(
        SELECTEDVALUE(ModelSelector[3D Url]),
        FILTER(
            ModelSelector,
            ModelSelector[ModelCode] = [SelectedModel]
        )
    )
)
```
- **Average Price**
```dax
AveragePrice = 
VAR selected = [SelectedModel]
RETURN
IF(
    selected = "ALL",
    AVERAGE('Tesla Stock Dataset'[price]),
    CALCULATE(
        AVERAGE('Tesla Stock Dataset'[price]),
        'Tesla Stock Dataset'[model] = selected
    )
)
```
- **Median Odometer**
```dax
MedianOdometer = 
VAR selected = [SelectedModel]
RETURN
IF(
    selected = "ALL",
    MEDIAN('Tesla Stock Dataset'[odometer]),
    CALCULATE(
        MEDIAN('Tesla Stock Dataset'[odometer]),
        'Tesla Stock Dataset'[model] = selected
    )
)
```
- **Average EMI**
```dax
AverageEMI = 
VAR selected = [SelectedModel]
RETURN
IF(
    selected = "ALL",
    AVERAGE('Tesla Stock Dataset'[EMI]),
    CALCULATE(
        AVERAGE('Tesla Stock Dataset'[EMI]),
        'Tesla Stock Dataset'[model] = selected
    )
)
```
- **Percent Without Accident**
```dax
PercentNoAccident = 
VAR selected = [SelectedModel]
RETURN
IF(
    selected = "ALL",
    DIVIDE(
        CALCULATE(COUNTROWS('Tesla Stock Dataset'), 'Tesla Stock Dataset'[accident_history] = "No Accidents"),
        COUNTROWS('Tesla Stock Dataset'),
        0
    ),
    DIVIDE(
        CALCULATE(
            COUNTROWS('Tesla Stock Dataset'),
            'Tesla Stock Dataset'[model] = selected,
            'Tesla Stock Dataset'[accident_history] = "No Accidents"
        ),
        CALCULATE(
            COUNTROWS('Tesla Stock Dataset'),
            'Tesla Stock Dataset'[model] = selected
        ),
        0
    )
)
```
- **Percent With Accident**
```dax
PercentWithAccident = 
VAR selected = [SelectedModel]
RETURN
IF(
    selected = "ALL",
    DIVIDE(
        CALCULATE(COUNTROWS('Tesla Stock Dataset'), 'Tesla Stock Dataset'[accident_history] <> "No Accidents"),
        COUNTROWS('Tesla Stock Dataset'),
        0
    ),
    DIVIDE(
        CALCULATE(
            COUNTROWS('Tesla Stock Dataset'),
            'Tesla Stock Dataset'[model] = selected,
            'Tesla Stock Dataset'[accident_history] <> "No Accidents"
        ),
        CALCULATE(
            COUNTROWS('Tesla Stock Dataset'),
            'Tesla Stock Dataset'[model] = selected
        ),
        0
    )
)
```

### Phase 3: Dashboard Development
#### Developed an interactive dark-themed analytical dashboard consisting of:
-	**KPI Summary Ribbon** – Displays `Average Price` ($45.11K), `Median Odometer` (28.67K), `Average EMI` ($710.42), `% Without Accident` (89.78%), and `% With Accident` (10.22%) with embedded trend sparklines and custom icons
-	**Regional Valuation & Usage Analysis** – Compares `Price by State` against `Average Mileage by State` across 18 US states
-	**Feature & Trim Valuation** – Breaks down `Price by Driver Assistance System` alongside a central interactive `3D VIEW` Sketchfab vehicle display

#### Visualizations Used
-	**New Card Visuals with Sparklines & Icons** → Average Price, Median Odometer, Average EMI, % Without Accident, % With Accident
-	**Stacked Bar Charts** → Price by State & Average Mileage by State
-	**Donut Chart** → Price by Driver Assistance System (`Full Self-Driving`, `Autopilot`, `Enhanced Autopilot`)
-	**Web Viewer Visual** → Dynamic Sketchfab 3D model showcase (`3D VIEW`)
-	**Button & Tile Slicers** → Model navigation sidebar (`Cyber Truck`, `Model 3`, `Model S`, `Model X`, `Model Y`) and top condition filter (`No Accidents`, `Previously Repaired`)

#### Design & Layout
-	High-contrast dark black and charcoal canvas with electric purple/violet accents for modern executive readability
-	Structured layout: 
    -	Model slicer navigation rail on the far left
    -	Condition slicers at the top right and 5 KPI cards across the top ribbon
    -	State-by-state price rankings on the left, 3D model view + Driver Assistance donut chart in the center, and state-by-state mileage rankings on the right


## Snapshot of Dashboard (Power BI Service)
<img width="1844" height="923" alt="Service" src="https://github.com/user-attachments/assets/76f3b502-85eb-4d89-b74a-2f31fd5fa960" />

 
# Report Snapshot (Power BI DESKTOP)
## Tesla Stock Sales Analysis — Models & Price Insights
<img width="1913" height="973" alt="power bi desktop" src="https://github.com/user-attachments/assets/f12a24c9-133e-42e6-a53f-bb4db0422c11" />


# Dashboard Insights
### **Pricing, Odometer & Financing Overview (Cyber Truck)**
- **Core Valuation:** Selected inventory averages **$45.11K** in vehicle price with a **Median Odometer** reading of **28.67K** miles and an **Average EMI** of **$710.42/month**
- **Vehicle Condition:** **89.78%** of listed inventory has **No Accidents**, while **10.22%** is categorized as **Previously Repaired**
  
### **Geographic Price & Mileage Analysis**
- **Highest-Priced Markets:** **Massachusetts ($51.65K)** and **Virginia ($51.40K)** command the highest average vehicle prices, followed by **Florida ($48.67K)**, **Minnesota ($48.20K)**, and **Tennessee ($48.03K)**
- **Lowest-Priced Markets:** **Arizona ($37.47K)**, **Pennsylvania ($38.45K)**, and **New York ($40.05K)** exhibit the lowest average listing prices
- **Mileage vs. Price Inverse Trend:** High-mileage states like **New York (46.00K miles)** and **Ohio (38.56K miles)** show significantly lower average prices ($40.05K and $43.60K respectively), whereas lower-mileage states like **Massachusetts (24.59K miles)** and **Tennessee (21.51K miles)** maintain premium pricing
### **Driver Assistance System Valuation**
- **Full Self-Driving (FSD) Premium:** Vehicles equipped with **Full Self-Driving** command the highest average valuation at **$47.03K (35.2%)**
- **Autopilot & Enhanced Autopilot:** **Enhanced Autopilot** vehicles average **$46.68K (34.94%)**, while standard **Autopilot** models average **$39.89K (29.86%)**—demonstrating a ~$7.1K price premium for FSD over base Autopilot


# Strategic Recommendations
- **Regional Arbitrage & Inventory Reallocation:** Source lower-priced, low-mileage inventory from states like **Oregon** (12.16K avg miles / $47.10K) and **Tennessee** (21.51K avg miles / $48.03K) to list in high-demand premium markets like **Massachusetts ($51.65K)** and **Virginia ($51.40K)**
- **FSD & Enhanced Autopilot Merchandising:** Highlight **Full Self-Driving** and **Enhanced Autopilot** capabilities prominently in online listings, as buyers consistently pay a **$6.8K–$7.1K premium** over base Autopilot trims
- **Targeted Financing Promotions:** Leverage the **$710.42 average EMI** benchmark to structure tailored monthly payment plans in price-sensitive states like **Arizona** and **Pennsylvania** to accelerate inventory turnover
- **Accident-History Discount Positioning:** Clearly segment the **10.22% Previously Repaired** vehicles with transparent inspection reports and competitive EMI tiers to attract value-focused buyers without eroding clean-title margins


# Technical Skills Demonstrated
-	**Power BI:** Custom dark-mode UI/UX design, disconnected slicer tables (`ModelSelector`, `ModelSelector_Slicer`), dynamic 3D Sketchfab HTML embed integration, interactive visual filtering
-	**DAX:** Dynamic `SELECTEDVALUE` routing, calculated columns (`SWITCH`), statistical aggregations (`AVERAGE`, `MEDIAN`), conditional ratios (`CALCULATE`, `DIVIDE`, `COUNTROWS`), KPI sparklines
-	**Data Analysis:** Multi-dimensional pricing elasticity analysis, geographic mileage-to-price correlation, trim feature valuation


# Project Status
| Project Component | Status |
| :--- | :--- |
| Data validation & ingestion | Completed |
| Slicer selector tables & 3D URL integration | Completed |
| Calculated columns & DAX KPI measures | Completed |
| Interactive dark-theme dashboard | Completed |
| Business insights and recommendations | Completed |

---
**Project Focus:** End-to-End Data Analytics | Automotive Pricing Intelligence | Business Intelligence  
**Author:** Eleazer Tabi Serebour
