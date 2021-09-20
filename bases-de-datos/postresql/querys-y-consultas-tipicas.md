---
description: Querys típicas SQL para postgresql
---

# Querys y consultas típicas

## ÍNDICE

* Espacio ocupado por objetos.
* Replicación.



## Espacio ocupado por tablas, índices, etc... <a id="espacio-ocupado-por-tablas-indices-etc1"></a>

Espacio ocupado por una base de datos.

```text
-- Espacio ocupado por una base de datos
SELECT pg_size_pretty(pg_database_size('mydb'));

-- Espacio ocupado por cada una de las bbdd en el servidor
SELECT
    pg_database.datname,
    pg_size_pretty(pg_database_size(pg_database.datname)) AS size
    FROM pg_database;
```

Espacio ocupado por un tablespace.

```text
SELECT pg_size_pretty(pg_tablespace_size('mytblspace'));
```

Espacio ocupado por un índice.

```text
SELECT  pg_size_pretty (pg_indexes_size('actor'));
```

Lista de las 5 tablas que ocupan más espacio en la base de datos.

```text
SELECT nspname || '.' || relname AS "relation",
    pg_size_pretty(pg_total_relation_size(C.oid)) AS "total_size"
  FROM pg_class C
  LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
  WHERE nspname NOT IN ('pg_catalog', 'information_schema')
    AND C.relkind <> 'i'
    AND nspname !~ '^pg_toast'
  ORDER BY pg_total_relation_size(C.oid) DESC
  LIMIT 5;
```

Lista de sesiones con estado idle

```text
SELECT datname,
CASE
    WHEN state_change < current_timestamp - INTERVAL '1' DAY THEN ' 1 DAY'
    WHEN state_change < current_timestamp - INTERVAL '12' HOUR THEN '12 HOURS'
    WHEN state_change < current_timestamp - INTERVAL '3' HOUR THEN '3 HOURS'
    WHEN state_change < current_timestamp - INTERVAL '75' MINUTE THEN '1.5 HOURS'
    WHEN state_change < current_timestamp - INTERVAL '1' HOUR THEN '1 HOUR'
    WHEN state_change < current_timestamp - INTERVAL '30' MINUTE THEN '30 MINUTES'
    WHEN state_change < current_timestamp - INTERVAL '20' MINUTE THEN '20 MINUTES'
    WHEN state_change < current_timestamp - INTERVAL '10' MINUTE THEN '10 MINUTES'
    ELSE 'OTHERS'
END AS idle_time, count(*)
FROM pg_stat_activity WHERE state = 'idle' 
AND (datname = 'ApiManagerCarbonDB' OR datname = 'regdb') 
GROUP BY idle_time,datname ;


SELECT datname,
CASE
    WHEN state_change < current_timestamp - INTERVAL '8' DAY THEN '8 DAYS'
    WHEN state_change < current_timestamp - INTERVAL '7' DAY THEN '7 DAYS'
    WHEN state_change < current_timestamp - INTERVAL '5' DAY THEN '5 DAYS'
    WHEN state_change < current_timestamp - INTERVAL '4' DAY THEN '4 DAYS'
    WHEN state_change < current_timestamp - INTERVAL '3' DAY THEN '3 DAYS'
    WHEN state_change < current_timestamp - INTERVAL '2' DAY THEN '2 DAYS'
    WHEN state_change < current_timestamp - INTERVAL '1' DAY THEN '1 DAY'
    WHEN state_change < current_timestamp - INTERVAL '12' HOUR THEN '12 HOURS'
    WHEN state_change < current_timestamp - INTERVAL '3' HOUR THEN '3 HOURS'
    WHEN state_change < current_timestamp - INTERVAL '75' MINUTE THEN '1.5 HOURS'
    WHEN state_change < current_timestamp - INTERVAL '1' HOUR THEN '1 HOUR'
    ELSE 'OTHERS'
END AS idle_time, count(*)
FROM pg_stat_activity WHERE state = 'idle' 
AND (datname = 'ApiManagerCarbonDB' OR datname = 'regdb') 
GROUP BY idle_time,datname ;
```

Ultima query que se ha ejecutado en una sesión idle.

```text
SELECT datname, query, count(*) 
FROM pg_stat_activity WHERE state = 'idle' 
AND (state_change < current_timestamp - INTERVAL '1' DAY) 
AND (datname = 'ApiManagerCarbonDB' OR datname = 'regdb') 
GROUP BY datname,query  ;
```

Detectar degradación de tablas e indices:

```text
SELECT
  current_database(), schemaname, tablename, /*reltuples::bigint, relpages::bigint, otta,*/
  ROUND((CASE WHEN otta=0 THEN 0.0 ELSE sml.relpages::FLOAT/otta END)::NUMERIC,1) AS tbloat,
  CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::BIGINT END AS wastedbytes,
  iname, /*ituples::bigint, ipages::bigint, iotta,*/
  ROUND((CASE WHEN iotta=0 OR ipages=0 THEN 0.0 ELSE ipages::FLOAT/iotta END)::NUMERIC,1) AS ibloat,
  CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes
FROM (
  SELECT
    schemaname, tablename, cc.reltuples, cc.relpages, bs,
    CEIL((cc.reltuples*((datahdr+ma-
      (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::FLOAT)) AS otta,
    COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,
    COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::FLOAT)),0) AS iotta -- very rough approximation, assumes all cols
  FROM (
    SELECT
      ma,bs,schemaname,tablename,
      (datawidth+(hdr+ma-(CASE WHEN hdr%ma=0 THEN ma ELSE hdr%ma END)))::NUMERIC AS datahdr,
      (maxfracsum*(nullhdr+ma-(CASE WHEN nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2
    FROM (
      SELECT
        schemaname, tablename, hdr, ma, bs,
        SUM((1-null_frac)*avg_width) AS datawidth,
        MAX(null_frac) AS maxfracsum,
        hdr+(
          SELECT 1+COUNT(*)/8
          FROM pg_stats s2
          WHERE null_frac<>0 AND s2.schemaname = s.schemaname AND s2.tablename = s.tablename
        ) AS nullhdr
      FROM pg_stats s, (
        SELECT
          (SELECT current_setting('block_size')::NUMERIC) AS bs,
          CASE WHEN SUBSTRING(v,12,3) IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr,
          CASE WHEN v ~ 'mingw32' THEN 8 ELSE 4 END AS ma
        FROM (SELECT version() AS v) AS foo
      ) AS constants
      GROUP BY 1,2,3,4,5
    ) AS foo
  ) AS rs
  JOIN pg_class cc ON cc.relname = rs.tablename
  JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname = rs.schemaname AND nn.nspname <> 'information_schema'
  LEFT JOIN pg_index i ON indrelid = cc.oid
  LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid
) AS sml
ORDER BY wastedbytes DESC;
```

## Listado de usuarios





