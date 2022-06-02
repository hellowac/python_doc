# clickhouse

* 官网: <https://clickhouse.com/>
* 文档: <https://clickhouse.com/docs/en/intro>

## 常用查询

### 查询数据库表大小

```sql
SELECT
    concat(database, '.', table) AS table,
    formatReadableSize(sum(bytes)) AS size,
    sum(bytes) AS bytes_size,
    sum(rows) AS rows,
    max(modification_time) AS latest_modification,
    any(engine) AS engine
FROM system.parts
WHERE active
GROUP BY
    database,
    table
ORDER BY bytes_size DESC
```

结果:

table                               |size      |bytes_size  |rows      |latest_modification    |engine             |
------------------------------------|----------|------------|----------|-----------------------|-------------------|
system.asynchronous_metric_log      |9.02 GiB  |  9684296449|7632396946|2022-06-01 15:23:21.000|MergeTree          |
system.metric_log                   |1.66 GiB  |  1783387541|  24864079|2022-06-01 15:23:25.000|MergeTree          |
system.query_thread_log             |1.40 GiB  |  1502640094|  54269589|2022-06-01 15:22:14.000|MergeTree          |
system.query_log                    |364.15 MiB|   381840631|   2324632|2022-06-01 15:23:19.000|MergeTree          |
system.trace_log                    |43.55 MiB |    45665106|   1529712|2022-06-01 15:18:53.000|MergeTree          |

### 集群查询

参考: <https://clickhouse.com/docs/en/sql-reference/table-functions/cluster/>

```sql
SELECT 
    table,
    sum(rows) AS `rows`,
    formatReadableSize(sum(data_uncompressed_bytes)) AS `uncps_bytes`,
    formatReadableSize(sum(data_compressed_bytes)) AS `bytes`,
    round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100, 0) AS `cps_rate`
FROM cluster('rz_cluster', system.parts)
GROUP BY table
```

结果:

table                          |rows       |uncps_bytes|bytes     |cps_rate|
-------------------------------|-----------|-----------|----------|--------|
trace_log                      |    2162927|452.87 MiB |60.91 MiB |    13.0|
query_log                      |    3167006|3.95 GiB   |466.63 MiB|    12.0|
asynchronous_metric_log        |30720829855|686.76 GiB |32.01 GiB |     5.0|
dimension_process_info         |      21618|506.67 KiB |101.94 KiB|    20.0|
metric_log                     |  103575826|220.51 GiB |6.44 GiB  |     3.0|
query_thread_log               |   57482489|41.82 GiB  |1.60 GiB  |     4.0|
dimension_send_info            |         12|354.00 B   |260.00 B  |    73.0|
