# umbrella-procurement-weather
Regression-based umbrella demand forecast translated into ROP/Safety Stock and ordering decision in Excel

# umbrella-procurement-weather

Regression-based umbrella demand forecast translated into ROP/Safety Stock and ordering decision in Excel.

## What’s inside
- `Umbrella_Company_Procurement_Model_Weather.xlsx` — Excel workbook with:
  - multiple linear regression (Rainfall, Temperature, Wind) → demand forecast
  - procurement logic: Safety Stock → DLT → ROP → Order Qty → MOQ rounding
  - reorder flag (ORDER / OK)

## How to use (Excel)
1. Open the file and go to **Procurement_Plan_Weather**.
2. Update inputs (Jan25W):
   - `Forecast_Rain_mm_Jan25`
   - `Forecast_Temp_C_Jan25`
   - `Forecast_Wind_kmh_Jan25`
   - `StockOnHandW`, `OnOrderW`, `MOQW`
   - `ServiceLevel`, `LeadTime_days` (from the base sheet)
3. Read the decision:
   - `Reorder_FlagW`
   - `Order_Q_MOQ_Jan25W` (final order quantity)

## Notes
- Wind Speed is not statistically significant in this sample (p-value high), but kept in the model for demonstration.
- This is a portfolio / learning case study with synthetic data.
