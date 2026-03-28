# Stock Price ELT Pipeline

An end-to-end data engineering pipeline that ingests real-time stock market data, stores it in PostgreSQL, and transforms it using dbt.

## Architecture
```
Polygon.io API → Airflow DAG → PostgreSQL (raw) → dbt → PostgreSQL (analytics)
```

## Tech Stack

- **Airflow** — orchestrates and schedules the daily pipeline
- **PostgreSQL** — stores raw and transformed data
- **dbt** — transforms raw data into clean analytical models
- **Docker** — containerises everything for reproducibility

## Models

- `stg_stock_prices` — cleans and type-casts raw API data
- `fct_daily_ohlcv` — adds daily change % and daily range
- `mrt_moving_averages` — 7-day and 30-day moving averages per ticker

## Setup

1. Clone the repo
2. Add your Polygon.io API key to `.env`
3. Run `docker compose up -d`
4. Open Airflow at http://localhost:8080 (admin/admin)
5. Trigger the `stock_price_pipeline` DAG
6. Run `cd dbt/stock_transform && dbt run && dbt test`

## Data Quality

7 automated dbt tests run on every pipeline execution covering null checks, accepted values, and price validity.