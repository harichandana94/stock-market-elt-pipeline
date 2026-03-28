# Stock Price ELT Pipeline

An end-to-end data engineering pipeline that fetches real-time stock market data, stores it in PostgreSQL, and transforms it using dbt — fully containerised with Docker.

## Architecture
```
Polygon.io API → Airflow DAG → PostgreSQL (raw) → dbt → PostgreSQL (analytics)
```

## Tech Stack

| Tool | Purpose |
|------|---------|
| Apache Airflow | Orchestrates and schedules the daily pipeline |
| PostgreSQL | Stores raw and transformed data |
| dbt | Transforms raw data into analytical models |
| Docker | Containerises everything for reproducibility |
| Polygon.io | Source of real-time stock price data |

## Pipeline Overview

The DAG runs daily and executes 3 tasks in order:

1. `create_table` — creates the raw schema and table if not exists
2. `fetch_and_load` — fetches OHLCV data for AAPL, MSFT, TSLA, GOOGL, AMZN
3. `validate` — runs data quality checks before marking the run complete

## dbt Models
```
raw.stock_prices
      ↓
analytics.stg_stock_prices   (clean types, filter bad rows)
      ↓
analytics.fct_daily_ohlcv    (daily change %, daily range)
      ↓
analytics.mrt_moving_averages (7-day and 30-day moving averages)
```

## Data Quality

7 automated dbt tests run on every execution:
- No null tickers, dates, or prices
- Only valid ticker symbols accepted
- No negative or zero prices

## Setup

1. Clone the repo
2. Create a `.env` file with your Polygon.io API key:
```
   POLYGON_API_KEY=your_key_here
   POSTGRES_USER=airflow
   POSTGRES_PASSWORD=airflow
   POSTGRES_DB=stocks
```
3. Start all services:
```bash
   docker compose up -d
```
4. Open Airflow at http://localhost:8080 (admin/admin)
5. Trigger the `stock_price_pipeline` DAG
6. Run dbt models:
```bash
   cd dbt/stock_transform
   dbt run
   dbt test
```

## Sample Output
```sql
SELECT ticker, trade_date, close_price, daily_change_pct
FROM analytics.fct_daily_ohlcv
ORDER BY daily_change_pct DESC;

 ticker | trade_date | close_price | daily_change_pct
--------+------------+-------------+-----------------
 GOOGL  | 2026-03-27 |      274.34 |            -1.06
 MSFT   | 2026-03-27 |      356.77 |            -1.42
 AAPL   | 2026-03-27 |      248.80 |            -2.01
 TSLA   | 2026-03-27 |      361.83 |            -2.13
```