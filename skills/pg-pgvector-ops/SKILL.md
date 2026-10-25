---
name: pg-pgvector-ops
description: Postgres 16 with the pgvector extension — migrations, extension checks, embedding columns, HNSW vs IVFFlat index choice, and similarity query patterns. Use when adding or changing schema, writing a migration, storing or querying embeddings, or diagnosing slow vector search.
---

# Postgres + pgvector Operations

Targets `pgvector/pgvector:pg16` driven through TypeORM migrations.

## Schema changes go through migrations. Always.

There is no path where editing an entity and restarting is the deploy. `synchronize`
stays off.

```bash
npm run migration:generate -- src/database/migrations/DescriptiveName
# READ the generated SQL — the generator gets destructive diffs wrong
npm run migration:run
npm run migration:revert    # undoes the last applied migration only
```

Before applying anything against data you care about, dump first:

```bash
docker compose exec postgres pg_dump -U <user> <db> > backup-$(date +%F).sql
```

Read the generated file specifically for: `DROP COLUMN`, `DROP TABLE`, and type changes
that imply a cast. The generator will happily produce a column drop when you meant a
rename, and a rename it cannot detect is always emitted as drop + add — which silently
discards the data.

## Extensions

`vector` must exist before any migration referencing a `vector` column runs. Create it
in the earliest migration:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Verify connectivity and extensions before debugging anything else:

```bash
npm run db:check
```

Checking by hand:

```sql
SELECT extname, extversion FROM pg_extension;
```

## Embedding columns

Declare the dimension and never change it in place — a different model means a new
column and a backfill, not an `ALTER TYPE`.

```sql
ALTER TABLE documents ADD COLUMN embedding vector(1536);
```

Record which model produced the vectors, alongside them:

```sql
ALTER TABLE documents ADD COLUMN embedding_model text NOT NULL DEFAULT 'unknown';
```

Without that column, a model swap leaves you with a table of vectors from two different
spaces and no way to tell them apart. Cosine distances between them are meaningless but
still return results — the failure is silent and looks like "search got worse".

## Distance operators

Match the operator to how the embeddings were trained, and use the same one in the index
and the query — a mismatch means the index is ignored.

| Operator | Distance | Use when |
|---|---|---|
| `<->` | L2 / Euclidean | embeddings are not normalized |
| `<=>` | Cosine | most text embedding models |
| `<#>` | Negative inner product | normalized vectors, speed matters |

## Index choice

**Build the index after bulk loading, not before** — inserting into an existing vector
index is dramatically slower.

**HNSW** — better recall and query speed, slower build, more memory. The default choice
for a read-heavy corpus.

```sql
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);
```

**IVFFlat** — faster to build, less memory, needs `lists` tuned to row count and needs
rebuilding as the table grows.

```sql
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

Rule of thumb for `lists`: `rows / 1000` up to 1M rows, then `sqrt(rows)`. An IVFFlat
index built on an empty table is worthless — it must be built on populated data.

## Query pattern

Order by distance, limit, and filter with a WHERE that the planner can use:

```sql
SELECT id, title, embedding <=> $1 AS distance
FROM documents
WHERE tenant_id = $2
ORDER BY embedding <=> $1
LIMIT 10;
```

Recall is tunable at query time:

```sql
SET hnsw.ef_search = 100;      -- HNSW, default 40; higher = better recall, slower
SET ivfflat.probes = 10;       -- IVFFlat, default 1
```

## When vector search is slow

1. `EXPLAIN ANALYZE` the query. If you see a sequential scan, the index is not being
   used — nearly always an operator mismatch between index and `ORDER BY`, or a `WHERE`
   clause that forced a different plan.
2. Confirm the index exists and finished building: `\di+` in psql.
3. A `LIMIT` is required for the index to help. `ORDER BY distance` with no limit reads
   everything.
4. Check the filter selectivity — a highly selective `WHERE` on a non-indexed column
   makes Postgres prefer a scan. Add a btree index on the filter column.
