# 📊 Automated Stock Market Insights Dashboard

An end-to-end pipeline that fetches live **FAANG stock data** (Meta, Apple, Amazon, Netflix, Google), stores it in **AWS RDS PostgreSQL**, and visualizes it in an interactive **Power BI dashboard** — fully automated with **AWS Lambda + CloudWatch**.

> **Stack:** Python · Alpha Vantage API · PostgreSQL · AWS Lambda · AWS RDS · Power BI

---

## Table of Contents
1. [Architecture](#architecture)
2. [Getting Started](#getting-started)
3. [Alpha Vantage API Key](#alpha-vantage-api-key)
4. [AWS RDS Database Setup](#aws-rds-database-setup)
5. [Data Fetching](#data-fetching)
   - [Historical — Last 30 Days](#historical--last-30-days)
   - [Daily Automation via Lambda](#daily-automation-via-lambda)
6. [Power BI Dashboard](#power-bi-dashboard)
7. [Dashboard Features](#dashboard-features)

---

## Architecture

<p align="center">
  <img src="images/dfd_stock_market_dashboard.png">
</p>

| Component | Role |
|---|---|
| **Alpha Vantage API** | Source of daily OHLCV stock data |
| **Python scripts** | Fetch, parse, and insert data into PostgreSQL |
| **AWS RDS PostgreSQL** | Persistent storage for all stock data |
| **AWS Lambda + CloudWatch** | Scheduled daily data fetch (no server needed) |
| **Power BI (ODBC)** | Interactive dashboard with KPIs and trend charts |

---

## Getting Started

### Prerequisites
- Python 3.10+
- AWS account (free tier works)
- Power BI Desktop
- pgAdmin (for database management)

```bash
# Clone the repository
git clone https://github.com/NirajanKhadka/Automated-Stock-Market-Dashboard.git
cd Automated-Stock-Market-Dashboard

# Install dependencies
pip install -r requirements.txt
```

---

## Alpha Vantage API Key

1. Go to [alphavantage.co/support/#api-key](https://www.alphavantage.co/support/#api-key)
2. Click **Get your free API key** and sign up
3. Copy your key and set it in the script:

```python
api_key = 'YOUR_API_KEY_HERE'
```

---

## AWS RDS Database Setup

### 1. Create PostgreSQL Instance on RDS
- AWS Console → RDS → **Create Database** → PostgreSQL
- Note the **Endpoint**, username, and password after creation
- Add an inbound rule in the Security Group to allow port `5432` from your IP

### 2. Connect via pgAdmin
- Host: your RDS endpoint
- Port: `5432`
- Username/Password: as configured above

### 3. Create the Stock Data Table

```sql
CREATE TABLE stock_data (
    id SERIAL PRIMARY KEY,
    symbol VARCHAR(10) NOT NULL,
    timestamp DATE NOT NULL,
    open_price NUMERIC,
    high_price NUMERIC,
    low_price NUMERIC,
    close_price NUMERIC,
    volume BIGINT,
    CONSTRAINT unique_stock_timestamp UNIQUE(symbol, timestamp)
);
```

---

## Data Fetching

### Historical — Last 30 Days

Set your credentials in `fetch_last_30days.py`:

```python
api_key    = 'your_alpha_vantage_key'
db_name    = 'your_db_name'
db_user    = 'your_db_user'
db_password = 'your_db_password'
db_host    = 'your_rds_endpoint'
db_port    = '5432'
```

Run the script:

```bash
python fetch_last_30days.py
```

The script filters data using a 30-day rolling window:

```python
from datetime import datetime, timedelta

today = datetime.now()
recent_data = {
    k: v for k, v in time_series.items()
    if datetime.strptime(k, '%Y-%m-%d') >= today - timedelta(days=30)
}
```

---

### Daily Automation via Lambda

#### 1. Create Lambda Function
- AWS Console → Lambda → **Create function**
- Runtime: Python 3.10
- Upload `lambda_function.zip`

#### 2. Schedule with CloudWatch Events
- CloudWatch → Rules → **Create Rule**
- Event source: **Schedule**
- Rate expression: `rate(1 day)` or cron: `cron(0 12 * * ? *)`
- Target: Select your `fetch_stock_data` Lambda function

#### 3. Test the Function
- In Lambda console, click **Test** → Create test event → Run
- Check **CloudWatch Logs** for execution results and errors

---

## Power BI Dashboard

### Connect Power BI to PostgreSQL via ODBC

1. Download and install [PostgreSQL ODBC Driver](https://www.postgresql.org/ftp/odbc/versions/msi/)
2. Open **ODBC Data Source Administrator** → System DSN → Add → PostgreSQL
3. Configure:
   - **Server:** your RDS endpoint
   - **Port:** 5432
   - **Database / User / Password:** as set in RDS
4. In Power BI Desktop → **Get Data** → ODBC → select `StockDatabase` → Load

### Schedule Refresh
- Publish report to Power BI Service
- Datasets → **Schedule Refresh** → match your Lambda frequency (daily)

---

## Dashboard Features

<p align="center">
  <img src="images/dashboard_main.png">
</p>

### Filters
- **Date Selector** — toggle between Last 7 Days / Last 30 Days
- **Stock Selector** — switch between META, AAPL, AMZN, NFLX, GOOGL

### KPIs (per selected stock)
- Open Price · Close Price · Volume · Moving Average (Price) · Moving Average (Volume)

### Charts & Rankings (Last 30 Days)

| Visual | What It Shows |
|---|---|
| Price History | Daily open and close price over 30 days |
| Volume History | Shares traded per day |
| Top 3 by Price Change | Stocks with highest % price movement |
| Top 3 by Volume | Most actively traded stocks |
| Top 3 by Avg Return | Most profitable over the period |
| Top 3 by Volatility | Highest risk/fluctuation stocks |