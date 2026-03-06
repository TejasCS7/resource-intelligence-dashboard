# 🖥️ Resource Intelligence Dashboard

![Demo](https://github.com/TejasCS7/Cloud-Cost-Optimization-and-Finops-Dashboard/blob/0375c8548bcda192ceade9d7d6c3393cdc7aa397/chrome-capture-2024-5-23-ezgif.com-resize.gif)

![Python](https://img.shields.io/badge/Python-3.13-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-Web_Framework-000000?style=for-the-badge&logo=flask&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Production_Ready-brightgreen?style=for-the-badge)

> A full-stack web application built with Flask and PostgreSQL that transforms raw infrastructure billing data into actionable intelligence — featuring a backend anomaly detection engine, time-series cost forecasting, and an interactive what-if scenario simulator, all served through a dynamic browser-based dashboard.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Application Architecture](#-application-architecture)
- [Backend Modules](#-backend-modules)
- [Key Engineering Features](#-key-engineering-features)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Database Schema](#-database-schema)
- [Setup & Installation](#-setup--installation)
- [Application Features](#-application-features)
- [Results & Impact](#-results--impact)
- [Testing](#-testing)
- [Future Roadmap](#-future-roadmap)
- [Contact](#-contact)

---

## 🧭 Overview

**Resource Intelligence Dashboard** is a full-stack Python web application that ingests raw cloud billing data, processes and stores it in a structured PostgreSQL database, and exposes it through an interactive Flask-powered dashboard.

The application is structured around a clean, modular backend with clearly separated responsibilities — an ingestion layer, a database layer, an analytics/computation engine, and a web presentation layer. Users can explore spending trends, investigate anomalies flagged by the backend engine, and run what-if simulations to model the financial impact of infrastructure changes — all through a responsive browser interface.

### Why How the System Is Engineered

| Concern | Implementation |
|---|---|
| **Web Application** | Flask backend with route handlers and Jinja2 templates |
| **Database Design** | Normalized PostgreSQL schema with structured ingestion scripts |
| **Backend Logic** | Modular Python services — ingestion, analysis, forecasting |
| **Algorithm Design** | Anomaly detection + time-series forecasting in Python |
| **API Design** | RESTful Flask routes returning JSON and rendered HTML |
| **Code Organization** | MVC-style separation — data layer, logic layer, presentation layer |
| **Reproducibility** | Virtual environment, `requirements.txt`, documented setup |

---

## 🏗 Application Architecture

The application follows a layered backend architecture — each layer has a single job and passes its output cleanly to the next:

```
┌─────────────────────────────────────────────────────┐
│                  WEB LAYER (Flask)                  │
│  code4_finops_dashboard.py + main.py                │
│  ├── Route handlers (GET/POST endpoints)            │
│  ├── Jinja2 template rendering                      │
│  └── JSON API responses for dynamic chart updates  │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│              ANALYTICS ENGINE LAYER                 │
│  code3_cost_analysis.py                             │
│  ├── Anomaly detection (statistical thresholding)  │
│  ├── Time-series forecasting (3-month projection)  │
│  └── What-if scenario computation engine           │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│               DATABASE LAYER (PostgreSQL)           │
│  code2_database_setup.py                            │
│  ├── Schema creation and table management          │
│  ├── SQL aggregation queries (GROUP BY, CTEs)      │
│  └── psycopg2 connection management                │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│              INGESTION LAYER (Python)               │
│  code1_data_ingestion.py                            │
│  ├── CSV parsing and schema validation             │
│  ├── Data cleaning and type normalization          │
│  └── Bulk INSERT into PostgreSQL                   │
└─────────────────────────────────────────────────────┘
        ▲
        │
  raw_dataset.csv  →  processed_dataset.csv
```

---

## 🔧 Backend Modules

The application is composed of **4 backend Python modules** plus a `main.py` entry point, each owning a distinct layer of responsibility.

### `code1_data_ingestion.py`
> **Role:** Data ingestion and cleaning service

- Reads raw billing CSV (`raw_dataset.csv`) into memory
- Validates column types, strips whitespace, normalizes date formats, and handles nulls
- Writes a clean intermediate file (`processed_dataset.csv`) for auditability
- Bulk-inserts validated records into the PostgreSQL `billing_records` table using `psycopg2`'s `execute_batch` for performance

---

### `code2_database_setup.py`
> **Role:** Database schema manager

- Creates and manages the PostgreSQL database schema on first run
- Defines normalized tables for billing records, resource metadata, and aggregated summaries
- Contains reusable query functions (`get_monthly_totals()`, `get_resource_breakdown()`, `get_anomalies()`) that are called by the analytics engine and Flask routes
- Manages connection pooling and cursor lifecycle cleanly

---

### `code3_cost_analysis.py`
> **Role:** Analytics and computation engine

This is the core algorithmic module. It contains three independent computation features:

**Anomaly Detection**
- Computes rolling mean and standard deviation across historical billing records
- Flags entries that deviate beyond a configurable Z-score threshold as spending anomalies
- Returns structured anomaly objects (resource ID, date, actual vs expected spend, deviation score)

**Time-Series Forecasting**
- Fits a trend model on historical monthly cost data
- Projects billing amounts for the next 3 months with confidence intervals
- Designed to be called both on page load and dynamically via Flask API routes

**What-If Scenario Engine**
- Accepts user-defined parameter changes (e.g., "remove idle resources", "downgrade instance tier")
- Recomputes projected spend against the current baseline
- Returns structured before/after comparison objects consumed by the dashboard frontend

---

### `code4_finops_dashboard.py`
> **Role:** Flask web application — routes, views, and API endpoints

- Registers all Flask routes: index, `/anomalies`, `/forecast`, `/scenarios`, `/api/simulate`
- Calls analytics engine functions and passes results to Jinja2 templates as context
- Exposes a `/api/simulate` POST endpoint that accepts JSON parameters and returns recomputed scenario projections — enabling the interactive what-if UI without full page reloads
- Handles application configuration (database URL, debug mode, secret key)

---

### `main.py`
> **Role:** Application entry point

- Bootstraps the application: runs `code2_database_setup.py` to ensure schema exists, then runs the ingestion pipeline if the database is empty, then starts the Flask dev server
- Single command to go from zero to a running application: `python src/main.py`

---

## 🚀 Key Engineering Features

### 1. Modular MVC-Style Backend
The codebase is strictly layered — the Flask routes never touch raw data, the ingestion module never builds queries, and the analytics engine never renders HTML. Each module has a clearly defined interface, making the application easy to extend and test.

### 2. Anomaly Detection Algorithm
The backend statistical engine computes rolling Z-scores across billing time-series data, automatically flagging unusual spending spikes. Anomalies are surfaced in the dashboard with deviation scores and suggested investigation actions.

### 3. Interactive What-If Simulation via REST API
The `/api/simulate` endpoint accepts a JSON payload of parameter overrides and dynamically returns a recomputed cost projection — powering the dashboard's scenario simulator without any page reload. This is a clean example of a backend computation API consumed by a frontend UI.

### 4. Normalized PostgreSQL Schema
Rather than querying CSVs directly, all data lives in a structured relational schema. The analytics queries are written as proper SQL with CTEs and aggregations — leveraging the database engine for performance instead of doing everything in Python memory.

### 5. Self-Bootstrapping Application
Running `python src/main.py` is all it takes — the app checks whether the schema exists, creates it if not, ingests data if the table is empty, and starts the server. No manual setup steps beyond installing dependencies.

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| **Language** | Python 3.13 |
| **Web Framework** | Flask |
| **Database** | PostgreSQL 17 |
| **DB Driver** | psycopg2 |
| **Templating** | Jinja2 |
| **Data Processing** | pandas, csv |
| **Analytics** | Python (statistical computation) |
| **Visualization** | Chart.js (via Flask templates) |
| **Environment** | Python venv |
| **Version Control** | Git / GitHub |

---

## 📁 Project Structure

```
resource-intelligence-dashboard/
│
├── src/
│   ├── code1_data_ingestion.py      # Ingestion & cleaning service
│   ├── code2_database_setup.py      # Schema manager & query functions
│   ├── code3_cost_analysis.py       # Anomaly detection, forecasting, scenarios
│   ├── code4_finops_dashboard.py    # Flask app — routes, views, API endpoints
│   └── main.py                      # Entry point — bootstraps and starts app
│
├── data/
│   ├── raw_dataset.csv              # Source billing data (input)
│   └── processed_dataset.csv        # Cleaned data (generated by ingestion module)
│
├── templates/                       # Jinja2 HTML templates
│   ├── index.html
│   ├── anomalies.html
│   ├── forecast.html
│   └── scenarios.html
│
├── static/                          # CSS and JS assets
│
├── requirements.txt
├── .env.example                     # Environment variable template
└── README.md
```

---

## 🗄 Database Schema

```sql
-- Core billing records table
CREATE TABLE billing_records (
    id              SERIAL PRIMARY KEY,
    resource_id     VARCHAR(100) NOT NULL,
    resource_type   VARCHAR(50),
    service_name    VARCHAR(100),
    billing_date    DATE NOT NULL,
    cost_usd        NUMERIC(12, 4) NOT NULL,
    usage_quantity  NUMERIC(12, 4),
    usage_unit      VARCHAR(50),
    project_id      VARCHAR(100),
    region          VARCHAR(50),
    ingested_at     TIMESTAMP DEFAULT NOW()
);

-- Monthly aggregation view (used by forecast and dashboard routes)
CREATE VIEW monthly_cost_summary AS
SELECT
    DATE_TRUNC('month', billing_date) AS month,
    service_name,
    SUM(cost_usd)                     AS total_cost,
    COUNT(*)                          AS record_count
FROM billing_records
GROUP BY 1, 2
ORDER BY 1;
```

---

## ⚙️ Setup & Installation

### Prerequisites

- Python 3.10+
- PostgreSQL 16+ (running locally or via Docker)
- Git

---

### Step 1 — Clone the Repository

```bash
git clone https://github.com/TejasCS7/resource-intelligence-dashboard.git
cd resource-intelligence-dashboard
```

---

### Step 2 — Create Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate          # macOS / Linux
# venv\Scripts\activate           # Windows
```

---

### Step 3 — Install Dependencies

```bash
pip install -r requirements.txt
```

---

### Step 4 — Configure Environment Variables

```bash
cp .env.example .env
```

Edit `.env` with your PostgreSQL credentials:

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=resource_intelligence
DB_USER=your_postgres_user
DB_PASSWORD=your_postgres_password
FLASK_ENV=development
SECRET_KEY=your_secret_key_here
```

---

### Step 5 — Run the Application

```bash
python src/main.py
```

On first run, the application will automatically:
1. Create the PostgreSQL database schema
2. Ingest and clean the billing dataset
3. Start the Flask development server at `http://localhost:5000`

---

## 🖥 Application Features

### 📊 Main Dashboard (`/`)
Displays an overview of total spend by service, monthly trend chart, and top cost drivers — powered by SQL aggregations and rendered via Jinja2 + Chart.js.

### 🚨 Anomaly Explorer (`/anomalies`)
Lists all billing records flagged by the anomaly detection engine with their deviation scores, resource IDs, and expected vs actual cost comparison. Filterable by date range and service.

### 📈 Cost Forecast (`/forecast`)
Renders a 3-month projected cost chart alongside historical actuals — showing trend direction and confidence bands computed by the time-series forecasting module.

### 🔧 What-If Scenario Simulator (`/scenarios`)
An interactive form where users input hypothetical changes (idle resource removal, instance downgrades). On submission, a POST request hits `/api/simulate`, which returns a JSON projection the frontend renders as a before/after cost comparison.

#### API Endpoint — Scenario Simulation

```
POST /api/simulate
Content-Type: application/json

{
  "remove_idle_resources": true,
  "idle_threshold_days": 30,
  "downgrade_instance_tier": false
}
```

```json
{
  "baseline_monthly_cost": 4820.50,
  "projected_monthly_cost": 3186.37,
  "savings_amount": 1634.13,
  "savings_percentage": 33.9,
  "breakdown": {
    "idle_resource_elimination": 1200.00,
    "rightsizing": 434.13
  }
}
```

---

## 📊 Results & Impact

| Metric | Result |
|---|---|
| Cost Reduction Identified | **Up to 33.9%** via idle resource analysis |
| Forecast Horizon | **3 months** with trend-based projection |
| Anomaly Detection | Automated flagging via Z-score thresholding |
| What-If Scenarios | Dynamic, sub-second API response |
| Database | Structured PostgreSQL schema with SQL aggregations |
| Deployment | Single-command bootstrap (`python src/main.py`) |

---

## 🧪 Testing

### Manual Testing
Each module is independently invocable for isolated testing:

```bash
# Test ingestion layer independently
python src/code1_data_ingestion.py

# Test database setup and query functions
python src/code2_database_setup.py

# Test analytics engine (prints anomalies + forecast to console)
python src/code3_cost_analysis.py
```

### API Testing
The `/api/simulate` endpoint can be tested with curl:

```bash
curl -X POST http://localhost:5000/api/simulate \
  -H "Content-Type: application/json" \
  -d '{"remove_idle_resources": true, "idle_threshold_days": 30}'
```

### Database Validation
```sql
-- Verify record ingestion
SELECT COUNT(*) FROM billing_records;

-- Verify monthly aggregation view
SELECT * FROM monthly_cost_summary LIMIT 10;

-- Spot-check anomaly candidates
SELECT resource_id, billing_date, cost_usd
FROM billing_records
ORDER BY cost_usd DESC
LIMIT 20;
```

---

## 🛣 Future Roadmap

- **REST API Expansion** — Expose all dashboard views as versioned JSON API endpoints for integration with external tools
- **Authentication** — Add Flask-Login with session management and role-based access control
- **Multi-Cloud Support** — Extend ingestion layer to support AWS Cost Explorer and Azure Billing APIs alongside GCP
- **Docker Deployment** — Containerize the app and PostgreSQL with a `docker-compose.yml` for one-command reproducible setup
- **CI/CD Pipeline** — Add GitHub Actions workflow for automated linting, testing, and deployment
- **Advanced ML Forecasting** — Replace the statistical trend model with an ARIMA or Prophet-based time-series model via a pluggable forecasting interface

---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 📬 Contact

Built by **Tejas Gaikawad**

📧 tejasdgaikwad265@gmail.com
🔗 [LinkedIn](https://www.linkedin.com/in/tejas-gaikawad/)
🐙 [GitHub](https://github.com/TejasCS7)
