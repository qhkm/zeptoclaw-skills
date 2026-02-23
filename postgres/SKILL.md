---
name: postgres
version: 1.0.0
description: PostgreSQL queries, schema management, backups, and performance tuning.
author: ZeptoClaw
license: MIT
tags:
  - database
  - postgresql
  - sql
env_needed:
  - name: DATABASE_URL
    description: PostgreSQL connection string (postgresql://user:pass@host:5432/db)
    required: false
metadata: {"zeptoclaw":{"emoji":"🐘","requires":{"anyBins":["psql","pg_dump"]}}}
---

# PostgreSQL Skill

Interact with PostgreSQL databases — queries, schema changes, backups, and diagnostics.

## Connect

```bash
# Via DATABASE_URL
psql "$DATABASE_URL"

# Explicit params
psql -h localhost -U myuser -d mydb -p 5432
```

## Common Queries

```sql
-- List all tables
\dt

-- Describe a table
\d users

-- Row counts for all tables
SELECT schemaname, tablename, n_live_tup AS rows
FROM pg_stat_user_tables ORDER BY n_live_tup DESC;

-- Find slow queries (requires pg_stat_statements)
SELECT query, calls, mean_exec_time::int AS avg_ms, total_exec_time::int AS total_ms
FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 10;
```

## Schema Management

```bash
# Run a migration file
psql "$DATABASE_URL" -f migration.sql

# Create a table
psql "$DATABASE_URL" -c "
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL REFERENCES customers(id),
  total NUMERIC(10,2) NOT NULL,
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT NOW()
);"

# Add a column
psql "$DATABASE_URL" -c "ALTER TABLE orders ADD COLUMN notes TEXT;"

# Create an index
psql "$DATABASE_URL" -c "CREATE INDEX CONCURRENTLY idx_orders_customer ON orders(customer_id);"
```

## Backup & Restore

```bash
# Full backup (compressed)
pg_dump "$DATABASE_URL" -Fc -f backup_$(date +%Y%m%d).dump

# Restore
pg_restore -d "$DATABASE_URL" --no-owner -1 backup_20260101.dump

# Schema only
pg_dump "$DATABASE_URL" --schema-only -f schema.sql

# Single table
pg_dump "$DATABASE_URL" -t orders -f orders.sql
```

## Performance Diagnostics

```sql
-- Active connections
SELECT pid, usename, application_name, state, query_start, query
FROM pg_stat_activity WHERE state != 'idle';

-- Table bloat estimate
SELECT tablename, pg_size_pretty(pg_total_relation_size(tablename::text)) AS size
FROM pg_tables WHERE schemaname = 'public' ORDER BY pg_total_relation_size(tablename::text) DESC;

-- Missing indexes (sequential scans on large tables)
SELECT schemaname, tablename, seq_scan, idx_scan,
  seq_scan - idx_scan AS seq_minus_idx
FROM pg_stat_user_tables WHERE seq_scan > 0
ORDER BY seq_scan DESC LIMIT 20;

-- Kill a long-running query
SELECT pg_terminate_backend(pid) FROM pg_stat_activity
WHERE pid != pg_backend_pid() AND query_start < NOW() - INTERVAL '5 minutes';
```

## Tips

- Use `CONCURRENTLY` for `CREATE INDEX` in production — avoids table lock
- Always test `ALTER TABLE` on a staging db first
- `pg_dump -Fc` (custom format) is smaller and restores faster than plain SQL
- Set `statement_timeout = '30s'` in `postgresql.conf` to prevent runaway queries
