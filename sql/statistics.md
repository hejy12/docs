---
title: Introduction to Statistics
category: user guide
---

# Introduction to Statistics

Based on the statistics, the TiDB optimizer chooses the most efficient query execution plan. The statistics collect table-level and column-level information.

- The statistics of a table include the total number of rows and the number of updated rows.
- The statistics of a column include the number of different values, the number of `NULL`, the histogram, and the Count-Min Sketch of the column.

## Collect statistics

### Manual collection

You can run the `ANALYZE` statement to collect statistics.

Syntax:

```sql
ANALYZE TABLE TableNameList
> The statement collects statistics of all the tables in `TableNameList`. 

ANALYZE TABLE TableName INDEX [IndexNameList]
> The statement collects statistics of the index columns on all `IndexNameList` in `TableName`.
> The statement collects statistics of all index columns when `IndexNameList` is empty.
```

### Automatic update

For the `INSERT`, `DELETE`, or `UPDATE` statements, TiDB automatically updates the number of rows and updated rows. TiDB persists this information regularly and the update cycle is 5 * `stats-lease`. The default value of `stats-lease` is `3s`. If you specify the value as `0`, it does not update automatically.

When the query is executed, TiDB collects feedback with the probability of `feedback-probability` and uses it to update the histogram and Count-Min Sketch.

### Control `ANALYZE` concurrency

When you run the `ANALYZE` statement, you can adjust the concurrency using the following parameters, to control its effect on the system.

#### `tidb_build_stats_concurrency`

Currently, when you run the `ANALYZE` statement, the task is divided into multiple small tasks. Each task only works on one column or index. You can use the `tidb_build_stats_concurrency` parameter to control the number of simultaneous tasks. The default value is `4`.

#### `tidb_distsql_scan_concurrency`

When you analyze regular columns, you can use the `tidb_distsql_scan_concurrency` parameter to control the number of Region to be read at one time. The default value is `10`.

#### `tidb_index_serial_scan_concurrency`

When you analyze index columns, you can use the `tidb_index_serial_scan_concurrency` parameter to control the number of Region to be read at one time. The default value is `1`.

## View statistics

You can view the statistics status using the following statements.

### Metadata of tables

You can use the `SHOW STATS_META` statement to view the total number of rows and the number of updated rows.

Syntax:

```sql
SHOW STATS_META [ShowLikeOrWhere]
> The statement returns the total number of rows and the number of updated rows. You can use `ShowLikeOrWhere` to filter the information you need.
```

Currently, the `SHOW STATS_META` statement returns the following 5 columns:

| Syntax Element | Description  |
| :-------- | :------------- |
| `db_name`  |  database name    |
| `table_name` | table name |
| `update_time` | the time of the update |
| `modify_count` | the number of modified rows |
| `row_count` | the total number of rows |

### Metadata of columns

You can use the `SHOW STATS_HISTOGRAMS` statement to view the number of different values and the number of `NULL` in all the columns.

Syntax:

```sql
SHOW STATS_HISTOGRAMS [ShowLikeOrWhere]
> The statement returns the number of different values and the number of `NULL` in all the columns. You can use `ShowLikeOrWhere` to filter the information you need.
```

Currently, the `SHOW STATS_HISTOGRAMS` statement returns the following 7 columns:

| Syntax Element | Description    |
| :-------- | :------------- |
| `db_name`  |  database name    |
| `table_name` | table name |
| `column_name` | column name |
| `is_index` | whether it is an index column or not |
| `update_time` | the time of the update |
| `distinct_count` | the number of different values |
| `null_count` | the number of `NULL` |
| `avg_col_size` | the average length of columns |

### Buckets of histogram

You can use the `SHOW STATS_BUCKETS` statement to view each bucket of the histogram.

Syntax:

```sql
SHOW STATS_BUCKETS [ShowLikeOrWhere]
> The statement returns information about all the buckets. You can use `ShowLikeOrWhere` to filter the information you need.
```

Currently, the `SHOW STATS_BUCKETS` statement returns the following 9 columns:

| Syntax Element | Description   |
| :-------- | :------------- |
| `db_name`  |  database name    |
| `table_name` | table name |
| `column_name` | column name |
| `is_index` | whether it is an index column or not |
| `bucket_id` | the ID of a bucket |
| `count` | the number of all the values that falls on the bucket and the previous buckets |
| `repeats` | the occurrence number of the maximum value |
| `lower_bound` | the minimum value |
| `upper_bound` | the maximum value |

## Delete statistics

You can run the `DROP STATS` statement to delete statistics.

Syntax:

```sql
DROP STATS TableName
> The statement deletes statistics of all the tables in `TableName`.
```

## Import and export statistics

### Export statistics

The interface to export statistics:

```
http://${tidb-server-ip}:${tidb-server-status-port}/stats/dump/${db_name}/${table_name}
> Use this interface to obtain the JSON format statistics of the `${table_name}` table in the `${db_name}` database.
```

### Import statistics

Generally, the imported statistics refer to the JSON file obtained using the export interface.

Syntax:

```
LOAD STATS 'file_name'
> `file_name` is the file name of the statistics to be imported.
```