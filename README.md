# FastAPI City Temperature Management API

This project implements:

- **City CRUD API** (`/cities`)
- **Temperature API**
  - `POST /temperatures/update`: asynchronously fetches current temperatures for all cities and stores them
  - `GET /temperatures`: returns temperature history (optionally filtered by `city_id`)

## Tech stack

- FastAPI
- SQLAlchemy 2.0 (async)
- SQLite + aiosqlite driver
- httpx for async HTTP calls

## Project structure

```
app/
  main.py
  core/
    config.py
  db/
    session.py
    base.py
  models/
    city.py
    temperature.py
  schemas/
    city.py
    temperature.py
  crud/
    city.py
    temperature.py
  api/
    routers/
      cities.py
      temperatures.py
tests/
```

## How to run

### 1) Create venv and install deps

Using `pip`:

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
# source .venv/bin/activate

pip install -r requirements.txt
```

Or using `uv`:

```bash
uv venv
uv pip install -r requirements.txt
```

### 2) Configure environment

Create `.env` in the project root:

```env
DATABASE_URL=sqlite+aiosqlite:///./app.db
OPEN_METEO_TIMEOUT_SECONDS=10
```

### 3) Start API

```bash
uvicorn app.main:app --reload
```

Open docs: `http://127.0.0.1:8000/docs`

## Design choices

- **Async SQLAlchemy** (`AsyncSession`) is used everywhere.
- **Dependency injection**: `get_db()` yields an `AsyncSession` for endpoints.
- **Temperature provider**: `Open-Meteo` is used because it is free and does not require an API key.
  - We store `city.latitude` / `city.longitude` to fetch temperature reliably.
  - If you don't know coordinates, you can extend the app with a geocoding step (not included here).

## Assumptions / simplifications

- Each city record stores `latitude` and `longitude` inside `additional_info` (or better: dedicated fields).
  In this implementation we model them as explicit columns for correctness.
- Temperature is fetched as **current temperature** (`current.temperature_2m`) from Open-Meteo.

## Example workflow

1. `POST /cities` with name + lat/lon
2. `POST /temperatures/update` to fetch temperatures for all cities
3. `GET /temperatures` or `GET /temperatures?city_id=1` to see history

## Notes

SQLite is fine for the task; for production you'd typically migrate to Postgres and add Alembic migrations.