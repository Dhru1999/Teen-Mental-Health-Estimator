# Mental Health Signal

### A student's daily rhythm, read as a signal—not a diagnosis.

**Mental Health Signal** is a student-wellness learning project that explores how social-media habits, study routines, sleep, physical activity, and perceived stress relate to a predicted mental-health score. It brings together a scikit-learn regression model, a FastAPI prediction service, and a responsive web interface.

> **Important:** This project is for learning and informational purposes only. Its score is a model estimate—not a diagnosis, screening result, or substitute for professional care. If you are concerned about your wellbeing, talk with someone you trust or a qualified professional.

## At a glance

- **Try the web app:** [https://teen-mental-health-estimator-eqoaivrhx-dhrupti.vercel.app/](https://teen-mental-health-estimator-eqoaivrhx-dhrupti.vercel.app/)
- **Interactive model notebook:** [`ML_Project.ipynb`](./ML_Project.ipynb)
- **Model artifact:** [`Mental_Health_Model.pkl`](./Mental_Health_Model.pkl)
- **Dataset:** [`Student Social Media And Mental Health Impact.csv`](./Student%20Social%20Media%20And%20Mental%20Health%20Impact.csv)

## What it does

1. Collects a small profile and daily-habit snapshot in the browser.
2. Checks the form values and sends them to the prediction API.
3. Runs the submitted values through a saved preprocessing-and-model pipeline.
4. Displays the estimated score on a 0–10 gauge, with a plain-language signal band.

The interface describes results as **strained** (below 4), **balanced** (4 to below 7), or **strong** (7 and above). These labels are presentation cues in the app, not clinically validated categories.

## Project structure

| File | Purpose |
| --- | --- |
| `public/index.html` | Form, result panel, and page structure |
| `public/style.css` | Responsive visual design and gauge styling |
| `public/script.js` | Client-side validation, API request, and result rendering |
| `main.py` | FastAPI app, request validation, and prediction endpoint |
| `Mental_Health_Model.pkl` | Serialized scikit-learn pipeline used by the API |
| `ML_Project.ipynb` | Data exploration, model training, evaluation, and export workflow |
| `Student Social Media And Mental Health Impact.csv` | Dataset used in the notebook |
| `requirements.txt` | Python dependencies for running the API |
| `ML Project.html` | Project build/implementation blueprint |

## Run it locally

### 1. Start the API

From the project directory, create and activate a virtual environment, then install the API dependencies:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Start the service from the same directory as `main.py` and the model file:

```powershell
python -m uvicorn main:app --reload --port 2200
```

The API will be available at `http://127.0.0.1:2200`. Interactive API documentation is at `http://127.0.0.1:2200/docs`.

### 2. Serve the web page

In a second terminal, from the project directory:

```powershell
py -m http.server 5500 --directory public
```

Open `http://127.0.0.1:5500` in your browser. When the page is opened from `localhost`/`127.0.0.1`, `script.js` sends predictions to the local API at `http://127.0.0.1:2200`; anywhere else (e.g. on Vercel) it calls the API on the same origin.

## Deploy to Vercel

The repo is set up for Vercel's zero-config FastAPI support: `main.py` exposes `app`, the frontend in `public/` is served as static files, and `.vercelignore` keeps the notebook and dataset out of the function bundle. `scikit-learn` is pinned to `1.9.0` because the saved model was trained with that version and does not load with others.

1. Import the GitHub repo at [vercel.com/new](https://vercel.com/new) (framework preset: **FastAPI**), or run `npx vercel` from the project directory.
2. Deploy. The web app is served at `/`, predictions at `/predict`, and API docs at `/docs`.

### Re-run the training notebook (optional)

The checked-in `requirements.txt` covers the API, not the full notebook environment. To work through the notebook, install Jupyter and its analysis/visualization dependencies in your environment:

```powershell
python -m pip install jupyter numpy matplotlib seaborn
```

Then launch Jupyter from the project directory and open `ML_Project.ipynb`. The notebook reads the CSV by its checked-in filename and saves the fitted pipeline as `Mental_Health_Model.pkl`.

## API

### `GET /health`

Returns a small welcome response.

### `POST /predict`

Send a JSON object with these fields:

| Field | Type | Accepted values / range |
| --- | --- | --- |
| `age` | integer | 10–100 |
| `gender` | string | `Male`, `Female` |
| `country` | string | Country name; unrecognized values are grouped as `Other` for the model |
| `academic_level` | string | `High School`, `Undergraduate`, `Graduate` |
| `most_used_platform` | string | `Facebook`, `LinkedIn`, `Instagram`, `Snapchat`, `Twitter`, `YouTube`, `TikTok`, `LINE`, `KakaoTalk`, `VKontakte`, `WhatsApp`, `WeChat` |
| `purpose_of_use` | string | `Networking`, `Education`, `Entertainment`, `News` |
| `avg_daily_usage_hours` | number | 0–24 |
| `daily_unlocks` | integer | 0 or greater |
| `study_hours` | number | 0–24 |
| `physical_activity_hours` | number | 0–24 |
| `sleep_hours_per_night` | number | 0–24 |
| `stress_level` | string | `Low`, `Medium`, `High`, `Very High` |

Example request:

```json
{
  "age": 21,
  "gender": "Female",
  "country": "India",
  "academic_level": "Undergraduate",
  "most_used_platform": "Instagram",
  "purpose_of_use": "Networking",
  "avg_daily_usage_hours": 4.5,
  "daily_unlocks": 60,
  "study_hours": 5,
  "physical_activity_hours": 1,
  "sleep_hours_per_night": 7.5,
  "stress_level": "Medium"
}
```

Successful responses have this shape:

```json
{
  "predicted_mental_health_score": 6.42
}
```

The numeric result is rounded to two decimal places. Invalid request values are rejected by FastAPI/Pydantic with a `422` response.

## How the model was built

The notebook frames the task as **regression**: estimate `Mental_Health_Score` from the available student and daily-routine features. It explores the data, prepares a 70/30 train/test split with `random_state=42`, compares a Linear Regression baseline with Random Forest models, and tunes the Random Forest using randomized cross-validation.

The exported object is a complete scikit-learn pipeline, so the API can provide raw feature columns and let the saved pipeline perform the same transformations used during training. The notebook's preprocessing includes:

- A log transform and scaling for `Study_Hours`.
- Scaling for the other numeric features.
- Ordinal encoding for stress (`Low` → `Medium` → `High` → `Very High`).
- One-hot encoding for non-ordinal categories.
- Country grouping to limit high cardinality: selected countries are retained and remaining countries become `Other`.

The API performs the same country grouping before prediction. If you regenerate or replace the pickle, use a scikit-learn version compatible with the version used to create that artifact; pickle-based model files are not guaranteed to be portable across arbitrary library versions.

## Data and feature inputs

The supplied CSV contains 5,000 rows and a target column named `Mental_Health_Score`. The prediction inputs cover:

- **Profile:** age, gender, country, academic level.
- **Digital habits:** most-used platform, primary purpose, average daily usage, daily unlocks.
- **Daily routine:** study time, physical activity, sleep, perceived stress.

## Privacy and responsible use

Submitted form values are sent to the API host configured in `script.js`. Review that host's policies before entering personal information. This repository does not include a clinical validation study or establish that the model is appropriate for real-world decisions. Do not use its output to diagnose, rank, or make consequential decisions about a person.

## Built with

- Python, FastAPI, Pydantic, pandas, joblib, and scikit-learn
- HTML, CSS, and vanilla JavaScript
- Jupyter Notebook for exploration and model training
