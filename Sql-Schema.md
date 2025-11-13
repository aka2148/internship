# Basic Sql Schema

```
CREATE TABLE production_summary (
    date DATE PRIMARY KEY,
    total_production INT
);

```
```
CREATE TABLE bench_summary (
    bench_id INTEGER PRIMARY KEY AUTOINCREMENT,
    date DATE NOT NULL,
    bench_name TEXT NOT NULL,
    asset_name TEXT NOT NULL,
    bench_production INT,
    FOREIGN KEY (date) REFERENCES production_summary(date)
);
```
```

CREATE TABLE trip_summary (
    bench_id INTEGER NOT NULL,
    source TEXT,
    destination TEXT,
    specification TEXT,
    asset_name TEXT,
    shift1_trips INT DEFAULT 0,
    shift2_trips INT DEFAULT 0,
    shift3_trips INT DEFAULT 0,
    total_trips INT GENERATED ALWAYS AS (shift1_trips + shift2_trips + shift3_trips) STORED,  -- PostgreSQL / MySQL 5.7+ only
    FOREIGN KEY (bench_id) REFERENCES bench_summary(bench_id)
);
```
