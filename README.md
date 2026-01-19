# umbrella-procurement-weather

Regression-based umbrella demand forecasting translated into a practical procurement decision (Safety Stock, DLT, ROP, MOQ order qty) in Excel.

This repo contains an Excel case study where **weather signals** (Rainfall, Temperature, Wind) are used to **predict demand**, and the forecast is then converted into **inventory policy** and a concrete **order recommendation**.

---

## File
- `Umbrella_Company_Procurement_Model_Weather.xlsx`

---

## What we built (end-to-end)
### 1) Data setup
We created a monthly dataset (19 observations) with:
- Rainfall (mm)
- Temperature (°C)
- Wind Speed (km/h)
- Quantity Sold (units)

Goal: explain and forecast demand using weather as drivers.

---

### 2) Regression model (demand driver model)
We ran **multiple linear regression**:

**Demand = Intercept + b_Rainfall * Rainfall + b_Temperature * Temperature + b_Wind * Wind**

From the Excel regression output:
- **R² ≈ 0.904** → the model explains ~90% of demand variability in this sample.
- Rainfall has strong positive impact (statistically significant).
- Temperature is also significant in this sample.
- Wind is **not significant** (high p-value), but we kept it for demonstration of multi-factor models.

Model parameters were copied into the planning sheet as named cells:
- `Intercept2`
- `b_Rainfall`
- `b_Temperature`
- `b_Wind`
- plus model error estimate: `Sigma_ResidualW` (standard deviation of residuals)

**Why we need Sigma_ResidualW?**  
Because forecasting is never perfect — residuals are the “typical forecast error”. We use this to size **Safety Stock**.

---

### 3) Forecasting a future month (Jan25)
We added forecasted weather inputs for Jan25:
- `Forecast_Rain_mm_Jan25`
- `Forecast_Temp_C_Jan25`
- `Forecast_Wind_kmh_Jan25`

Then we calculated:
- `Demand_Forecast_Jan25` using the regression equation.

This gives a single-number demand forecast for the month based on expected weather.

---

## Procurement logic (how the forecast becomes an order decision)
We implemented a classic reorder-point approach:

### Key inputs (changeable)
- `ServiceLevel` (e.g., 0.95) → desired probability of not stocking out
- `LeadTime_days` (e.g., 30) → supplier lead time
- `StockOnHandW` → inventory on hand now
- `OnOrderW` → inventory already ordered but not received yet
- `MOQW` → minimum order quantity (or order multiple)

### 4) Safety Stock (buffer for uncertainty)
We compute a Z-score from service level:
- `z = NORM.S.INV(ServiceLevel)`

Then Safety Stock (monthly lead time version):
- `SafetyStock_Jan25W = z * Sigma_ResidualW * SQRT(LeadTime_months)`

**Why this works:**  
If forecast errors are roughly normal, Z translates your service level into the buffer needed to cover forecast uncertainty during lead time.

---

### 5) Demand during Lead Time (DLT)
Because our lead time is ~1 month here:
- `DLT_Jan25W = Demand_Forecast_Jan25 * LeadTime_months`

If lead time was e.g. 45 days, we’d convert to months accordingly.

---

### 6) Reorder Point (ROP)
ROP is the threshold where you should trigger replenishment:

- `ROP_Jan25W = DLT_Jan25W + SafetyStock_Jan25W`

Interpretation:
- If your **inventory position** drops below ROP → you risk stockout before the next replenishment arrives.

---

### 7) Inventory Position
We use “Inventory Position” because procurement decisions depend not only on what is in the warehouse, but also what is already coming:

- `InventoryPositionW = StockOnHandW + OnOrderW`

---

### 8) Order quantity (base)
Base replenishment quantity:

- `Order_Q_Jan25W = MAX(0; ROP_Jan25W - InventoryPositionW)`

**Why MAX(0)?**  
To avoid negative orders. If you are above ROP, you order 0.

---

### 9) MOQ rounding (final order quantity)
Suppliers often require MOQ or ordering in multiples. We round up to meet MOQ:

- `Order_Q_MOQ_Jan25W = CEILING.MATH(Order_Q_Jan25W; MOQW)`

Interpretation:
- If the calculation says order 1024 but MOQ is 200, final order becomes 1200.

---

### 10) Reorder flag (simple decision label)
We add a visible decision:

- `Reorder_FlagW = IF(InventoryPositionW < ROP_Jan25W; "ORDER"; "OK")`

This was also highlighted using conditional formatting (red for ORDER, green for OK).

---

## Where to look in the workbook (sheets)
- **Data**: the historical dataset used for regression
- **Regression_Output_Weather**: Excel output table (R², coefficients, p-values)
- **Procurement_Plan_Weather**: planning & decision sheet (inputs → forecast → SS/DLT/ROP → order)

---

## How to use (quick)
1. Open the workbook and go to **Procurement_Plan_Weather**.
2. Update inputs:
   - Weather forecasts (Rain/Temp/Wind for Jan25)
   - `StockOnHandW`, `OnOrderW`, `MOQW`
   - `ServiceLevel`, `LeadTime_days`
3. Read the result:
   - `Reorder_FlagW` (ORDER/OK)
   - `Order_Q_MOQ_Jan25W` (final order quantity)

---

## Notes / limitations
- Small sample (19 months) → good for learning & portfolio, not production-grade.
- Weather can explain a lot here, but real demand also depends on price, promotions, distribution, competition, stockouts, etc.
- Wind was not significant in this sample; in real work you’d likely remove or re-test it.

---

## Portfolio statement
This project demonstrates the full chain:
**driver-based demand forecasting → forecast uncertainty → safety stock → reorder point → MOQ-compliant order recommendation.**

