# Earthquake Classifier: ETL & Dynamic EDA Pipeline 🌋

An automated, production-grade Python telemetry pipeline that extracts real-time seismic event records from the public **USGS API**, validates incoming schemas with **Pydantic**, transforms and engineers machine-learning target labels with **Pandas**, and loads cleaned data into a **Cloud SQL Database (PostgreSQL via Neon / Supabase)**.

A lightweight **FastAPI** web application serves dynamic Exploratory Data Analysis (EDA) visualizations and live ETL sync controls, designed for deployment on **Vercel**.

---

## 🏗️ Architecture & Modular Project Structure

The project separates every pipeline stage and application responsibility into dedicated folders:

```
pai_demo_1/
│
├── .env.example                  # Template for Cloud SQL (Neon/Supabase) credentials
├── .env                          # Local environment variables
├── requirements.txt              # Production Python dependencies
├── vercel.json                   # Vercel serverless deployment configuration
├── README.md                     # Documentation & operational guide
│
├── etl/                          # ETL Pipeline (Separated by function)
│   ├── __init__.py
│   ├── extract/                  # EXTRACTION MODULE
│   │   ├── __init__.py
│   │   └── usgs_client.py        # requests client for USGS GeoJSON feeds & query API
│   ├── validate/                 # VALIDATION MODULE
│   │   ├── __init__.py
│   │   └── schema.py             # Pydantic schema validation & boundary checking
│   ├── transform/                # TRANSFORMATION & FEATURE ENGINEERING
│   │   ├── __init__.py
│   │   └── cleaner.py            # Pandas data cleaning & target label engineering (mag >= 5.0)
│   ├── load/                     # CLOUD DATABASE LOADER
│   │   ├── __init__.py
│   │   └── db_loader.py          # SQLAlchemy PostgreSQL loader (Neon/Supabase) + local fallback
│   └── run_pipeline.py           # Standalone orchestrator CLI for scheduled/manual ETL runs
│
├── app/                          # FastAPI Web Application & EDA Backend
│   ├── __init__.py
│   ├── main.py                   # FastAPI ASGI app & REST endpoints
│   ├── database.py               # Cloud DB connection provider & pooling
│   ├── services/                 # EDA SERVICES
│   │   ├── __init__.py
│   │   └── eda_service.py        # SQL queries for Class Balance, Missingness, and Spatial points
│   └── templates/                # FRONTEND
│       └── index.html            # Responsive Tailwind CSS dashboard (Chart.js + Leaflet.js)
│
└── tests/                        # Automated Test Suite
    ├── test_etl.py               # Unit tests for extract, validate, transform, and load
    └── test_api.py               # Integration tests for FastAPI endpoints
```

---

## 🔬 The 3 Core EDA Objectives

The FastAPI dashboard dynamically queries Cloud SQL to provide deep telemetry insights:

### 1. Class Balance Analysis
- **Problem**: Natural seismic events follow the Gutenberg-Richter power-law distribution. Minor tremors (`magnitude < 5.0`) vastly outnumber significant earthquakes (`magnitude >= 5.0`), creating severe class imbalance (typically ~98% minor vs. ~2% significant).
- **Visualization**: Interactive Chart.js donut chart showing exact event counts, percentages, and imbalance ratios (e.g., `1 : 48`).

### 2. Missingness Analysis across Sensor Attributes
- **Problem**: Sensor coverage is not uniform globally. Local micro-tremors are recorded by dense local networks, whereas teleseismic quakes rely on global networks. Attributes like station count (`nst`), azimuthal gap (`gap`), distance to nearest station (`dmin`), travel time error (`rms`), PAGER alerts (`alert`), and felt reports (`felt`) exhibit distinct missingness profiles.
- **Visualization**: Grouped bar chart comparing missingness percentages overall vs. within the Minor class vs. within the Significant class.

### 3. Spatial Distribution & Tectonic Plate Clusters
- **Problem**: Earthquakes cluster primarily along tectonic plate boundaries (the Pacific "Ring of Fire", the Mid-Atlantic Ridge, and the Alpine-Himalayan belt).
- **Visualization**: Interactive Leaflet.js dark-matter map plotting global coordinates with magnitude-scaled circle markers (Coral/Red for Significant `≥ 5.0`, Cyan/Teal for Minor `< 5.0`). Click any epicenter for rich station telemetry and hypocenter depth.

---

## ⚡ Quick Start & Local Execution

### 1. Prerequisites & Installation
Ensure Python 3.10+ is installed:

```bash
git clone <your-repo>
cd pai_demo_1

# Install dependencies
pip install -r requirements.txt
```

### 2. Configure Cloud Database (Neon / Supabase)
Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Set your Cloud SQL PostgreSQL connection string:

```ini
# Neon example:
DATABASE_URL=postgresql://neondb_owner:<password>@ep-sample-123.us-east-2.aws.neon.tech/neondb?sslmode=require

# Supabase example:
DATABASE_URL=postgresql://postgres:<password>@db.xxxx.supabase.co:5432/postgres?sslmode=require

# (Default if omitted: local SQLite sqlite:///./earthquakes.db)
```

### 3. Run the ETL Ingestion Pipeline
To fetch live telemetry from USGS, validate, clean, and load into your database:

```bash
# Ingest past 7 days of global telemetry
python etl/run_pipeline.py --feed all_week

# Or ingest past 24 hours
python etl/run_pipeline.py --feed all_day

# Or past 30 days
python etl/run_pipeline.py --feed all_month
```

### 4. Launch the Web Application & Dashboard
Start the FastAPI server:

```bash
uvicorn app.main:app --reload --port 8000
```

Open your browser at: **`http://localhost:8000`**

- Use the **"Sync Live Telemetry"** button in the dashboard to trigger live ETL updates directly from the web interface.
- View interactive Swagger API documentation at: **`http://localhost:8000/docs`**

---

## 🧪 Running Automated Tests

Run the test suite verifying schemas, data transformations, idempotent database loads, and API endpoints:

```bash
python -m unittest discover tests
```

---

## 🚀 Vercel Deployment Guide

Because the pipeline writes to Cloud SQL (Neon or Supabase), the application completely bypasses Vercel's read-only file system restriction.

1. Push your repository to GitHub.
2. In the [Vercel Dashboard](https://vercel.com), click **Add New Project** and select your repository.
3. In **Environment Variables**, add:
   - `DATABASE_URL`: Your Neon or Supabase PostgreSQL connection string.
   - `USGS_FEED`: `all_week`
4. Click **Deploy**. Vercel will build using `vercel.json` and serve the FastAPI application seamlessly!
