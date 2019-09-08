
CREATE KEYSPACE IF NOT EXISTS loki WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'}  AND durable_writes = true;

### Database Schema
```sql
CREATE TABLE time_series (
    shardid varchar,    
    name varchar,
    labels varchar,    
    fingerprint bigint,
    PRIMARY KEY (shardid, name, labels)
)
WITH CLUSTERING ORDER BY (name ASC, labels ASC)
        AND bloom_filter_fp_chance = 0.01
        AND caching = {'keys':'ALL', 'rows_per_partition':'NONE'}
        AND comment = 'timeseries'
        AND gc_grace_seconds = 60
        AND compaction = {'compaction_window_size': '120','compaction_window_unit': 'MINUTES', 'class': 'org.apache.cassandra.db.compaction.TimeWindowCompactionStrategy' };

CREATE TABLE samples (
    shardid varchar,    
    uuid timeuuid,
    fingerprint bigint,
    value float,
    string varchar,
    PRIMARY KEY (shardid, uuid, fingerprint)
) WITH CLUSTERING ORDER BY (uuid ASC, fingerprint ASC)
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys':'ALL', 'rows_per_partition':'NONE'}
    AND comment = 'samples'
    AND gc_grace_seconds = 345600
    AND compaction = {'compaction_window_size': '120','compaction_window_unit': 'MINUTES', 'class': 'org.apache.cassandra.db.compaction.TimeWindowCompactionStrategy' };


```

CREATE INDEX samples_fingerprint ON loki.samples (fingerprint);

