# What Drives Delhi's Air Pollution? A Multi-Year PM2.5 Analysis

**Question:** Which factors (season, weather, day of week, festivals, location) best explain PM2.5 spikes across Delhi, and when and where should authorities act first?

**Stack:** Python (pandas, SciPy, statsmodels) · SQL Server (star schema, window functions) · Power BI (DAX)

## Important: demo data vs real data
`python src/01_ingest_clean.py --demo` generates **synthetic** data with realistic faults (outages, stuck sensors, error codes, duplicates) so you can run the full pipeline immediately. **Numbers produced from demo data are not findings.** Before publishing anything, replace it with real CPCB data (below) and rewrite the findings section from your own results.

## Real data
1. **Air quality:** download hourly station data for 8-10 Delhi stations, 2021-2025, from the CPCB CAAQMS portal (airquality.cpcb.gov.in) or the CPCB data download pages. Save one CSV per station in `data/raw/stations/` with columns `datetime, PM2.5, PM10, NO2, SO2, CO, O3`. If the export uses other column names, edit `COLUMN_MAP` in `01_ingest_clean.py`.
2. **Weather:** daily Delhi temperature, humidity, wind speed and rainfall from Open-Meteo's historical API (free). Save as `data/raw/weather_daily.csv` with columns `date, temp_c, humidity_pct, wind_kmh, rain_mm`.
3. **Diwali dates** are in `DIWALI` at the top of the cleaning script; check them against a calendar.

## Run it
```bash
pip install -r requirements.txt
python src/01_ingest_clean.py          # or add --demo
python src/02_eda_stats.py
```
Then load `data/processed/*.csv` into SQL Server (`sql/01_schema_and_load.sql`), run `sql/02_analysis_queries.sql`, and build the dashboard from `powerbi/dax_and_dashboard_spec.md`.

## Method highlights
- **Cleaning log:** every step records how many values it touched (`data/processed/cleaning_log.json`). Impossible values become NaN rather than being dropped; only gaps of 3 hours or less are interpolated; stuck sensors (identical reading for 24+ hours) are removed; a day needs 16+ valid hours.
- **No pseudo-replication:** tests use the city-day mean, because stations on the same day are not independent.
- **Confounding handled explicitly:** the Diwali effect is tested against a local 21-day baseline. A naive comparison with the rest of the season gave a misleading answer in testing, and that contrast is worth showing in your write-up.
- **Autocorrelation:** daily PM2.5 is strongly autocorrelated (Durbin-Watson well below 2), so the regression uses HAC (Newey-West) standard errors.

## Findings (fill in from real data)
1. Seasonal pattern: ___ (worst month, magnitude vs best month)
2. Weather: each +1 km/h wind is associated with ___% change in PM2.5; weather alone explains ___% of variance, season and events raise this to ___%
3. Festival effect: Diwali window ___% above local baseline (p = ___)
4. Where to act first: stations ___ and ___ exceed the Severe threshold on ___% of days

## Limitations
- Observational data: associations, not causes. Diwali overlaps with stubble burning and temperature inversions.
- Sensor quality varies by station; missing days are not random (outages cluster in bad-air periods in some datasets).
- Daily averaging hides hourly peaks. Station-level spatial dependence is not modelled.
- Weather is city-level, not station-level.

## Stretch goal
Forecast next-day PM2.5 category with a baseline (persistence) and a gradient-boosted model using lagged PM2.5 and weather; evaluate with a time-based split, never a random one.
