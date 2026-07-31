---
name: pycegerb-yoda
description: Write and review PySpark against Iceberg tables — schema discipline, native functions over UDFs, correct write mode, and table maintenance.
when_to_use: writing or reviewing a PySpark job, a Spark SQL query on Iceberg, an OOM or skew problem, a slow read, table maintenance, partitioning or schema evolution and prunning, or a file-size problem
---
# PySpark on Iceberg

Every claim here is traceable to the Apache Spark or Apache Iceberg docs — links
per section. Version-sensitive defaults are marked; verify against the version
you run before relying on one.

## Schema and types

- **Declare the schema; don't infer it.** `inferSchema` costs an extra pass over
  the data and pins correctness to whatever the sample happened to contain. Pass
  an explicit `StructType`.
- **Never `select("*")` into a write.** Column order and presence become part of
  your contract the moment something downstream reads it.
- **`toPandas()` and `createDataFrame(pandas_df)` move data through the driver.**
  The docs are explicit that this should be "done on a small subset of the data",
  and that not all Spark/Arrow types are supported — an unsupported column type
  raises. ([Arrow in PySpark](https://spark.apache.org/docs/latest/api/python/tutorial/sql/arrow_pandas.html))

## Transformations that don't bloat the plan

- **Don't chain `withColumn()` in a loop.** The PySpark docstring is explicit:
  it "introduces a projection internally", so calling it repeatedly "can generate
  big plans which can cause performance issues and even `StackOverflowException`".
  Use one `select()` with all the columns, or `withColumns({...})` (Spark 3.3.0+)
  to add many at once.
- **`try_cast` instead of `cast` on untrusted input.** It is "identical to `CAST`,
  except that it returns `NULL` result instead of throwing an exception on runtime
  error" — Spark's own `CAST_INVALID_INPUT` and `CAST_OVERFLOW` errors tell you to
  reach for it. In ANSI mode a bad row otherwise kills the job.
  ([ANSI compliance](https://spark.apache.org/docs/latest/sql-ref-ansi-compliance.html))
- **`eqNullSafe` when joining on nullable keys.** `=` returns NULL when either
  side is NULL, so those rows silently drop; `eqNullSafe` is an "equality test
  that is safe for null values" (since 2.3.0).
- **Qualify columns after a join.** Once two frames share a name, an unqualified
  reference is ambiguous — keep the frame alias on both sides.

## Native functions over Python UDFs

Reach for `pyspark.sql.functions` first. A Python UDF is a black box to the
Catalyst optimizer and pays per-row serialization between the JVM and the Python
worker.

When you genuinely need Python, prefer a **pandas UDF** (`@pandas_udf`) over a
plain `udf` — it uses Arrow to transfer data in columnar batches instead of
row-by-row.

Documented caveats worth knowing before you ship one:

- **Arrow is on by default** via `spark.sql.execution.arrow.pyspark.enabled`, and
  silently falls back to the non-Arrow path on error unless you disable
  `spark.sql.execution.arrow.pyspark.fallback.enabled`. A job that "works" may be
  on the slow path.
- **Batch size is capped by `spark.sql.execution.arrow.maxRecordsPerBatch`**
  (default 10,000 rows). The docs note that with many columns the value "should
  be adjusted accordingly", since batches raise JVM memory use.
- **`applyInPandas` / `cogroup` load an entire group into memory** — the docs
  warn this "can lead to out of memory exceptions, especially if the group sizes
  are skewed", and that `maxRecordsPerBatch` does *not* apply. Group size is your
  responsibility.
- `MapType` and `ArrayType` of nested `StructType` have restricted support.

([Arrow / pandas UDFs](https://spark.apache.org/docs/latest/api/python/tutorial/sql/arrow_pandas.html))

## Let AQE do the tuning

Adaptive Query Execution is enabled by default since Spark 3.2.0
(`spark.sql.adaptive.enabled`). Before hand-tuning `spark.sql.shuffle.partitions`,
check that you aren't fighting it:

- **Coalescing small partitions** — `coalescePartitions.enabled` targets
  `advisoryPartitionSizeInBytes` (default 64MB). On a busy cluster the docs
  recommend setting `coalescePartitions.parallelismFirst=false`, since the
  default `true` ignores the target size and maximizes parallelism instead.
- **Skewed joins** — `skewJoin.enabled` splits skewed sort-merge-join tasks. A
  partition counts as skewed when it exceeds *both* `skewedPartitionFactor` ×
  median *and* `skewedPartitionThresholdInBytes`. The docs advise setting that
  threshold larger than `advisoryPartitionSizeInBytes`.

([Performance tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html))

## Writing to Iceberg

Pick the write op deliberately — this is where most data bugs get introduced.

| Intent | Use | Note |
|---|---|---|
| Append rows | `INSERT INTO` / `df.writeTo(t).append()` | |
| Row-level upsert | `MERGE INTO` | **Recommended over `INSERT OVERWRITE`** |
| Replace touched partitions | `df.writeTo(t).overwritePartitions()` | dynamic overwrite |
| Replace by explicit filter | `df.writeTo(t).overwrite(<filter>)` | |
| CTAS / RTAS | `.create()` / `.replace()` / `.createOrReplace()` | |

**Use `MERGE INTO` rather than `INSERT OVERWRITE`.** The docs give two reasons:
Iceberg can replace only the affected data files, and "the data overwritten by a
dynamic overwrite may change if the table's partitioning changes."

**If you do overwrite, know which mode you're in.** Spark's default is *static*,
but **dynamic is what the Iceberg docs recommend** — set
`spark.sql.sources.partitionOverwriteMode=dynamic`. This matters more than it
sounds: in static mode an `INSERT OVERWRITE` with no `PARTITION` clause **drops
every existing row in the table**. Static mode also can't target hidden
partitions, because the `PARTITION` clause only references table columns.

Other documented behavior:

- `MERGE INTO`, `UPDATE`, and row-level `DELETE FROM` require the **Iceberg Spark
  SQL extensions**. Plain `INSERT INTO` / `INSERT OVERWRITE` need
  `spark.sql.storeAssignmentPolicy=ANSI` (the default since Spark 3.0).
- In a `MERGE INTO`, only one source row may match a given target row, "or else
  an error will be thrown." Deduplicate the source first.
- `DELETE FROM` matching whole partitions is a **metadata-only** delete; matching
  individual rows rewrites just the affected files.
- Prefer the **v2 writer** (`df.writeTo(...)`). The v1 `format("iceberg")` path
  carries an explicit danger note: it "loads an isolated table reference that
  will not automatically refresh tables used by queries." Use `saveAsTable` /
  `insertInto` if you must stay on v1.
- Schema merge on write is opt-in on **both** sides — table property
  `write.spark.accept-any-schema=true` *and* writer `.option("mergeSchema","true")`.

([Iceberg Spark writes](https://iceberg.apache.org/docs/latest/spark-writes/))

## Partitioning and evolution

- **Don't add a derived partition column by hand.** Iceberg uses *hidden
  partitioning*: it produces partition values from a source column via a
  transform (`days(ts)`, `bucket(8, id)`, …) and tracks the relationship, so
  queries filter on the real column and still prune files. The docs call out the
  Hive-style alternative as producing "silently incorrect results" when a
  consumer formats the partition column wrong.
- **Schema evolution is a metadata change** — adding, dropping, renaming or
  reordering a column rewrites no data files, and the docs guarantee the changes
  are "independent and free of side-effects".
- **Partition evolution is also metadata-only** and "does not eagerly rewrite
  files." Old data keeps its old spec; Iceberg plans each layout separately
  (split planning). So a partitioning mistake is fixable in place — no migration.

([Evolution](https://iceberg.apache.org/docs/latest/evolution/) ·
[Partitioning](https://iceberg.apache.org/docs/latest/partitioning/))

## Write distribution and file sizes

Since Iceberg 1.2.0 the default `write.distribution-mode` is `hash`, which asks
Spark to shuffle rows by partition value so each task writes few files. The three
modes: `none` (no shuffle — you must sort manually), `hash` (default), `range`
(two-stage, more expensive, globally sorted; the default when the table has a
sort order). Note Spark ignored distribution mode in CTAS/RTAS before 3.5.0.

On file sizes: Spark cannot write a file larger than a task, and a file can't
span a partition boundary — so `write.target-file-size-bytes` only takes effect
if tasks are big enough. AQE's `advisoryPartitionSizeInBytes` is measured on
row-based shuffle data, while the output is columnar and better compressed, so it
needs to be set **larger** than the target file size.

([Iceberg Spark writes](https://iceberg.apache.org/docs/latest/spark-writes/))

## Maintenance

Every write creates a snapshot; snapshots and small files accumulate until
something reclaims them. Run these as Spark SQL procedures:

```sql
-- reclaim expired snapshots and their exclusive files
CALL catalog.system.expire_snapshots(table => 'db.t', older_than => TIMESTAMP '2024-01-01 00:00:00.000', retain_last => 100);

-- compact small files (strategy: binpack default, or sort / zorder)
CALL catalog.system.rewrite_data_files(table => 'db.t', strategy => 'sort', sort_order => 'id DESC NULLS LAST');

-- regroup manifests when write order stops matching read filters
CALL catalog.system.rewrite_manifests('db.t');

-- ALWAYS dry_run first
CALL catalog.system.remove_orphan_files(table => 'db.t', dry_run => true);
```

Documented safety rules — the destructive ones:

- **`remove_orphan_files` can cause data loss.** Its retention interval must not
  be shorter than the longest in-flight write, or in-progress files get treated
  as orphans; the default is 3 days. Iceberg compares **string paths**, so a
  changed filesystem authority makes nothing match — the docs state plainly that
  this "will lead to data loss when RemoveOrphanFiles is run."
- `expire_snapshots` defaults to 5 days ago and retains the last 1 snapshot; it
  never removes files a live snapshot still needs. Snapshots held by branches or
  tags survive, and `main` never expires. Expiring destroys time travel to those
  snapshots.
- Set `stream_results => true` on both procedures for large tables — recommended
  in the docs to avoid driver OOM.
- Old **metadata** files are a separate problem: set
  `write.metadata.delete-after-commit.enabled=true` (default `false`) to prune
  tracked ones as they're superseded. It won't touch already-untracked files —
  only orphan-file removal reaches those.

([Maintenance](https://iceberg.apache.org/docs/latest/maintenance/) ·
[Spark procedures](https://iceberg.apache.org/docs/latest/spark-procedures/))

## Source

Written from the Apache Spark and Apache Iceberg documentation (both Apache-2.0),
which every claim above is traceable to.

Prompted by
[skill-pyspark-expert](https://github.com/leandroasaservice/skill-pyspark-expert)
by leandroasaservice. That skill is GPL-3.0 and this repo is MIT, so no text was
copied from it. Its subject matter was surveyed to decide what belonged here —
it focuses on code style and testing rather than performance — and the few
overlapping topics (`withColumn` chains, `try_cast`, `eqNullSafe`) were then
written independently from the Spark docs and source docstrings.

Verifying rather than trusting caught one error worth noting: `eqNullSafe` is
sometimes described as a Spark 3.5+ feature, but its docstring reads
`.. versionadded:: 2.3.0`. Check version claims against the docs for the release
you actually run.

## Pairs with

- `sql-review` — the general query review pass; this skill is the engine-specific layer.
