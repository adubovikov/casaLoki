
CREATE KEYSPACE IF NOT EXISTS loki WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'}  AND durable_writes = true;

### Database Schema
```sql
CREATE TABLE time_series (
    name varchar,
    labels varchar,    
    fingerprint bigint,
    PRIMARY KEY (name, labels)
)
WITH CLUSTERING ORDER BY (labels ASC)
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
    PRIMARY KEY (shardid, uuid)
) WITH CLUSTERING ORDER BY (uuid ASC)
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys':'ALL', 'rows_per_partition':'NONE'}
    AND comment = 'samples'
    AND gc_grace_seconds = 345600
    AND compaction = {'compaction_window_size': '120','compaction_window_unit': 'MINUTES', 'class': 'org.apache.cassandra.db.compaction.TimeWindowCompactionStrategy' };


CREATE INDEX samples_fingerprint ON loki.samples (fingerprint);

cqlsh:loki> insert into time_series(name, labels, fingerprint) VALUES ('cpu','{"__name__":"up"}',7975981685167825999);
cqlsh:loki> select * from time_series ;

 name | labels            | fingerprint
------+-------------------+---------------------
  cpu | {"__name__":"up"} | 7975981685167825999

(1 rows)


```

```

insert into time_series(name, labels, fingerprint) VALUES ('cpu','{"__name__":"side","foo":"ar","TnMFeLL7eU1Xbx02":"JTwtk7BMr0WK27Bw"}',377426);
insert into time_series(name, labels, fingerprint) VALUES ('cpu','{"__name__":"side","foo":"ar","1vmeFtcafWhSFNLl":"Tqnj1KUjQX9PjPHs"}',677544);
insert into time_series(name, labels, fingerprint) VALUES ('cpu','{"__name__":"side","foo":"ar","Cq3uqHeqdtgfNVIB":"AOM1qzazkj659nqX"}',1657813);
insert into time_series(name, labels, fingerprint) VALUES ('cpu','{"__name__":"side","foo":"ar","vRfU3gTmxA13hOwo":"0JUkJLzqCP1dmtGZ"}',2829541);
insert into time_series(name, labels, fingerprint) VALUES ('cpu','{"__name__":"side","foo":"ar","99LjqgLYKIaKTQ0z":"DiCB0FdKbRH4J9JX"}',3154186);
insert into time_series(name, labels, fingerprint) VALUES ('cpu','{"__name__":"side","foo":"ar","UPJPsQliJgQX0wmM":"qlBskt3zZhkBIewq"}',3194976);



insert into samples(shardid, uuid, fingerprint, string, value) VALUES ('cpu_20190908',  0518a510-d23d-11e9-8664-b74cfc9ea5ad, 377426, 'fffffaaabbbbbb',0);
insert into samples(shardid, uuid, fingerprint, string, value) VALUES ('cpu_20190908', 051a04a0-d23d-11e9-8664-b74cfc9ea5ad, 677544, 'fffffaaabbbbbb',0);
insert into samples(shardid, uuid, fingerprint, string, value) VALUES ('cpu_20190908', 051b6430-d23d-11e9-8664-b74cfc9ea5ad, 1657813, 'fffffaaabbbbbb',0);
insert into samples(shardid, uuid, fingerprint, string, value) VALUES ('cpu_20190908', 051cc3c0-d23d-11e9-8664-b74cfc9ea5ad, 2829541, 'fffffaaabbbbbb',0);
insert into samples(shardid, uuid, fingerprint, string, value) VALUES ('cpu_20190908', 051dfc40-d23d-11e9-8664-b74cfc9ea5ad, 3194976, 'fffffaaabbbbbb',0);
```


Another way:

```
CREATE TABLE time_series (
    name varchar,
    labels map<text, text>,
    fingerprint bigint,
    PRIMARY KEY (name)
)
WITH bloom_filter_fp_chance = 0.01
        AND caching = {'keys':'ALL', 'rows_per_partition':'NONE'}
        AND comment = 'timeseries'
        AND gc_grace_seconds = 60
        AND compaction = {'compaction_window_size': '120','compaction_window_unit': 'MINUTES', 'class': 'org.apache.cassandra.db.compaction.TimeWindowCompactionStrategy' };

CREATE INDEX time_series_index ON time_series (ENTRIES(labels));

```
and inserts:

```
insert into time_series JSON '{"name":"cpu","labels":{"__name__":"side","foo":"ar","UPJPsQliJgQX0wmM":"qlBskt3zZhkBIewq"},"fingerprint":3194976}';
```


