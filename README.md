# WeatherAPI Test Suite

Postman collections for testing the WeatherAPI `/current.json` endpoint. Two separate collections — current weather and air quality — so you can run either one independently.

## Repo

```
collections/
    Feature1_CurrentWeather.postman_collection.json
    Feature2_AirQuality.postman_collection.json
environments/
    WeatherAPI_Shared.postman_environment.json
data/
    cities.json
.circleci/
    config.yml
README.md
TEST_CASES.md
```

---

## Setup



In `environments/WeatherAPI_Shared.postman_environment.json` replace `YOUR_API_KEY_HERE` with your key. The `city` variable defaults to `Sydney` — change it there too if you want a different default.

---

## Running from Postman

### Import the environment

1. Open Postman
2. Click **Environments** in the left sidebar
3. Click **Import** and select `WeatherAPI_Shared.postman_environment.json`
4. Open the environment, find `api_key` and set your key if you haven't already
5. Select the environment from the dropdown in the top right corner of Postman

### Import a collection

1. Click **Collections** in the left sidebar
2. Click **Import**
3. Select whichever collection you want — `Feature1_CurrentWeather` or `Feature2_AirQuality` (or both)

### Run Feature 1 — Current Weather

1. Click the three dots next to **Feature 1 - Current Weather**
2. Click **Run collection**
3. Make sure the environment is set to **WeatherAPI - Shared Environment**
4. Click **Run Feature 1 - Current Weather**

### Run Feature 2 — Air Quality

Same steps as above but select **Feature 2 - Air Quality**.

### Changing the city

Go to **Environments → WeatherAPI - Shared Environment** and update the `city` variable before running. You can set it to any city name — Melbourne, Brisbane, London, etc.

### Viewing results

Results appear in the Collection Runner after the run finishes. Green = passed, red = failed. Click any failed test to see what the actual vs expected values were. Check the **Console** (bottom left) for the `console.log` output from each request — it prints a short summary of the response data.

---

## Running from CircleCI

### One-time setup

1. Push this repo to GitHub
2. Go to https://app.circleci.com and click **Add Project**
3. Connect  GitHub repo
4. Go to **Project Settings → Environment Variables**
5. Add a variable called `WEATHERAPI_KEY` and set it to your API key

That's all the setup needed. Your key lives in CircleCI and never touches any file in the repo.

### Trigger a run from the CircleCI UI

1. Go to project in CircleCI
2. Click **Pipelines** in the left sidebar
3. Click **Trigger Pipeline** (top right)
4. Click **Add Parameters** to choose what to run

The available parameters are:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `run_current_weather` | boolean | true | Runs Feature 1 |
| `run_air_quality` | boolean | true | Runs Feature 2 |
| `city` | string | Sydney | City to test |
| `use_data_file` | boolean | false | Runs all cities in data/cities.json |

5. Set your parameters and click **Trigger Pipeline**

Some examples of what you can do:

**Run only Feature 1 for Melbourne**
```
run_current_weather  → true
run_air_quality      → false
city                 → Melbourne
```

**Run only Feature 2 for Brisbane**
```
run_current_weather  → false
run_air_quality      → true
city                 → Brisbane
```

**Run both features against all cities in the data file**
```
run_current_weather  → true
run_air_quality      → true
use_data_file        → true
```

### Trigger a run from the command line

If you want to kick off a pipeline without going into the UI:

```bash
curl --request POST \
  --url https://circleci.com/api/v2/project/gh/YOUR_ORG/YOUR_REPO/pipeline \
  --header "Circle-Token: $CIRCLECI_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "branch": "main",
    "parameters": {
      "run_current_weather": true,
      "run_air_quality": false,
      "city": "Melbourne"
    }
  }'
```

Replace `YOUR_ORG` and `YOUR_REPO` with your GitHub org and repo name. `CIRCLECI_TOKEN` is your personal CircleCI API token — generate one from https://app.circleci.com/settings/user/tokens.

### Viewing results in CircleCI

1. Go to **Pipelines** and click on the pipeline that just ran
2. Click on a job (e.g. `feature1_current_weather`)
3. Expand the Newman step to see the full test output in the console
4. Click the **Artifacts** tab to download the HTML report

---

## Adding more cities to the data file

Open `data/cities.json` and add entries:

```json
[
  { "city": "Sydney" },
  { "city": "Melbourne" },
  { "city": "Brisbane" },
  { "city": "Perth" }
]
```

Next time you run with `use_data_file=true` it will pick them up automatically.

---

## Converting to Australia standards NEPM AQI

WeatherAPI doesn't return an Australian AQI — it returns raw pollutant values. Feature 2 calculates the Australian NEPM AQI from those values using:

```
sub-index = (concentration / NEPM standard) × 100
overall AQI = highest sub-index across all pollutants
```

Since the API gives instantaneous readings rather than time-averaged ones, the number is indicative rather than an official figure. 