# WeatherAPI Test Cases Reference

**Endpoint:** `GET {{base_url}}/current.json`
**Last Updated:** April 2026

---

## Feature 1 — Current Weather (`aqi=no`)

| Test Case ID | Description | Expected Result |
|---|---|---|
| CW-TC01 | Get current weather for `{{city}}` | Status 200, `Content-Type: application/json`, response contains `location` and `current` keys, `location.name` matches requested city, `location.region` / `country` / `lat` / `lon` / `localtime` all present, `current.temp_c` and `current.temp_f` are numbers and consistent (°F ≈ °C×9/5+32 ±1), `current.feelslike_c` / `feelslike_f` are numbers, `current.condition.text` is a non-empty string, `current.condition.code` is a positive number, `current.humidity` is 0–100, `current.wind_kph` ≥ 0, `current.wind_dir` is non-empty, `current.pressure_mb` > 0, `current.precip_mm` ≥ 0, `current.vis_km` ≥ 0, `current.uv` ≥ 0, `current.is_day` is 0 or 1, `current.last_updated` is non-empty, `current.air_quality` block is **absent**, response time < 5000ms. All weather fields stored as environment variables. |
| CW-TC02 | Verify `result_temp_c` environment variable is set and valid after CW-TC01 | Status 200, `result_temp_c` is a finite number, value is within ±2°C of live `temp_c` from a re-query |
| CW-TC03 | Request with an invalid API key | Status 403, `error` object present in response, `error.code` is `2006` or `2008`, `error.message` is a non-empty string |
| CW-TC04 | Request with empty city parameter (`q=`) | Status 400, `error` object present, `error.code` is `1003` (parameter `q` missing) |
| CW-TC05 | Request with an unrecognised city name | Status 400, `error` object present, `error.code` is `1006` (location not found) |

---

## Feature 2 — Air Quality (`aqi=yes`)

| Test Case ID | Description | Expected Result |
|---|---|---|
| AQ-TC01 | Get air quality data for `{{city}}` and calculate Australian NEPM AQI | Status 200, `Content-Type: application/json`, response contains `location` and `current` keys, `location.name` matches requested city, `current.air_quality` block is **present**, all six raw pollutant values are non-negative numbers (`co`, `no2`, `o3`, `so2`, `pm2_5`, `pm10`), all six NEPM sub-indices are non-negative numbers, overall NEPM AQI is a non-negative number, category is one of Good / Fair / Poor / Very Poor / Hazardous, dominant pollutant is identified, response time < 5000ms. All pollutant values, NEPM AQI, category and dominant pollutant stored as environment variables. |
| AQ-TC02 | Verify `result_nepm_aqi`, `result_nepm_category` and `result_nepm_dominant` environment variables are set and valid after AQ-TC01 | Status 200, `result_nepm_aqi` is a non-negative number, `result_nepm_category` is a recognised NEPM category, `result_nepm_dominant` is a recognised pollutant code, stored NEPM AQI is within ±5 of a live re-calculated value |
| AQ-TC03 | Request with an invalid API key | Status 403, `error` object present in response, `error.code` is `2006` or `2008` |
| AQ-TC04 | Request with empty city parameter (`q=`) | Status 400, `error` object present, `error.code` is `1003` (parameter `q` missing) |
| AQ-TC05 | Request with an unrecognised city name | Status 400, `error` object present, `error.code` is `1006` (location not found) |

---

## Summary

| | Feature 1 — Current Weather | Feature 2 — Air Quality | Total |
|---|---|---|---|
| Positive Tests | 2 | 2 | 4 |
| Negative Tests | 3 | 3 | 6 |
| **Total** | **5** | **5** | **10** |

---

## Environment Variables Populated by Tests

### Feature 1 — Current Weather

| Variable | Populated by | Description |
|---|---|---|
| `result_temp_c` | CW-TC01 | Temperature in Celsius |
| `result_temp_f` | CW-TC01 | Temperature in Fahrenheit |
| `result_feelslike_c` | CW-TC01 | Feels-like temperature in Celsius |
| `result_feelslike_f` | CW-TC01 | Feels-like temperature in Fahrenheit |
| `result_condition` | CW-TC01 | Weather condition text (e.g. `Partly cloudy`) |
| `result_humidity` | CW-TC01 | Humidity percentage |
| `result_wind_kph` | CW-TC01 | Wind speed in km/h |
| `result_wind_dir` | CW-TC01 | Wind direction (e.g. `SSW`) |
| `result_pressure_mb` | CW-TC01 | Atmospheric pressure in millibars |
| `result_precip_mm` | CW-TC01 | Precipitation in millimetres |
| `result_vis_km` | CW-TC01 | Visibility in kilometres |
| `result_uv` | CW-TC01 | UV index |
| `result_is_day` | CW-TC01 | `1` = daytime, `0` = night |
| `last_updated` | CW-TC01 | Timestamp of the weather reading |

### Feature 2 — Air Quality (Raw Pollutants)

| Variable | Populated by | Description |
|---|---|---|
| `result_aqi_co` | AQ-TC01 | Carbon Monoxide concentration (μg/m³) |
| `result_aqi_no2` | AQ-TC01 | Nitrogen Dioxide concentration (μg/m³) |
| `result_aqi_o3` | AQ-TC01 | Ozone concentration (μg/m³) |
| `result_aqi_so2` | AQ-TC01 | Sulphur Dioxide concentration (μg/m³) |
| `result_aqi_pm2_5` | AQ-TC01 | PM2.5 fine particulates (μg/m³) |
| `result_aqi_pm10` | AQ-TC01 | PM10 coarse particulates (μg/m³) |

### Feature 2 — Australian NEPM AQI (Calculated)

| Variable | Populated by | Description |
|---|---|---|
| `result_nepm_aqi` | AQ-TC01 | Overall Australian NEPM AQI (highest sub-index across all pollutants) |
| `result_nepm_category` | AQ-TC01 | Human-readable NEPM category (Good / Fair / Poor / Very Poor / Hazardous) |
| `result_nepm_dominant` | AQ-TC01 | The pollutant driving the overall AQI (`co`, `no2`, `o3`, `so2`, `pm2_5`, or `pm10`) |
| `result_nepm_sub_co` | AQ-TC01 | NEPM sub-index for CO — `(CO μg/m³ ÷ 9000) × 100` |
| `result_nepm_sub_no2` | AQ-TC01 | NEPM sub-index for NO₂ — `(NO₂ μg/m³ ÷ 120) × 100` |
| `result_nepm_sub_o3` | AQ-TC01 | NEPM sub-index for O₃ — `(O₃ μg/m³ ÷ 210) × 100` |
| `result_nepm_sub_so2` | AQ-TC01 | NEPM sub-index for SO₂ — `(SO₂ μg/m³ ÷ 570) × 100` |
| `result_nepm_sub_pm2_5` | AQ-TC01 | NEPM sub-index for PM2.5 — `(PM2.5 μg/m³ ÷ 25) × 100` |
| `result_nepm_sub_pm10` | AQ-TC01 | NEPM sub-index for PM10 — `(PM10 μg/m³ ÷ 50) × 100` |

---

## Australian NEPM AQI Reference

> **Standard:** National Environment Protection (Ambient Air Quality) Measure — NEPM AAQ
> **Method:** Each pollutant is expressed as a percentage of its NEPM standard value. The overall AQI is the highest sub-index across all pollutants.

| Category | NEPM AQI Range | Health Implication |
|---|---|---|
| Good | 0 – 33 | Air quality is satisfactory with little or no risk |
| Fair | 34 – 66 | Acceptable quality; may be a concern for a small number of sensitive individuals |
| Poor | 67 – 99 | Sensitive groups may experience health effects; general public unlikely to be affected |
| Very Poor | 100 – 149 | Everyone may begin to experience health effects; sensitive groups more seriously affected |
| Hazardous | 150+ | Health alert — everyone may experience serious health effects |

### NEPM Standard Values Used for Sub-Index Calculation

| Pollutant | NEPM Standard | Averaging Period |
|---|---|---|
| CO (Carbon Monoxide) | 9,000 μg/m³ | 8-hour |
| NO₂ (Nitrogen Dioxide) | 120 μg/m³ | 1-hour |
| O₃ (Ozone) | 210 μg/m³ | 1-hour |
| SO₂ (Sulphur Dioxide) | 570 μg/m³ | 1-hour |
| PM2.5 (Fine Particulates) | 25 μg/m³ | 24-hour |
| PM10 (Coarse Particulates) | 50 μg/m³ | 24-hour |


