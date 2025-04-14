# OrcaHouse Athena

**OrcaHouse Athena** follows similar setup from its predecessor [Portal Athena](https://github.com/umccr/data-portal-apis/tree/dev/docs/athena) setup. Please read high level [architecture note](../arch) to understand the overall data flow and data pipeline within warehouse.

Use Athena setting as follows.

```
Workgroup:      orcahouse
Data Source:    orcavault
Database:       mart
```

## Query

See model guide [dbt](https://umccr.github.io/orcahouse-doc/dbt/orcavault/#!/model/model.orcavault.lims) and [erd](https://umccr.github.io/orcahouse-doc/erd/) for schema and column details.

```sql
select * from orcavault.mart.lims;
```

```sql
select * from orcavault.mart.fastq;
```

```sql
select
    *
from orcavault.mart.lims as lims
    join orcavault.mart.fastq as fq on lims.sequencing_run_id = fq.sequencing_run_id and lims.library_id = fq.library_id
where
    lims.library_id = 'LPRJ230400' and format = 'ora';
```

### History query

At some point, you might want to query `fastq_history` table to trace FASTQ file movement i.e. historical locations of the file -- where they have been, when they get deleted and where they are at now. Basically their "effective" time window at the said location.

As best practise; please use predicate filter with `WHERE` or, `LIMIT` clause when querying historical records to prevent over-fetching and, maintain query performance.

Example:

```sql
select * from orcavault.mart.fastq_history limit 10;
```

In most cases, you would want to filter by `library_id`.

```sql
select * from orcavault.mart.fastq_history where library_id = 'L2400519';
```

### Federated query

It is best practise to use full data source pointer when you query.

```sql
select * from <data_source>.<database>.<table>;
```

Equivalently in native PostgreSQL, this [maps](https://docs.aws.amazon.com/athena/latest/ug/understanding-tables-databases-and-the-data-catalog.html) as follows.

```sql
select * from <database>.<schema>.<table>;
```

### Passthrough query

For some reason, if you'd like to run [passthrough](https://docs.aws.amazon.com/athena/latest/ug/connectors-postgresql.html#connectors-postgres-passthrough-queries) native PostgreSQL query statement, you can wrap to `table()` function like so. 

```sql
select * from table(
    system.query(
            query => 'select * from orcavault.mart.lims order by sequencing_run_date desc limit 10'
        ));
```

```sql
select * from table(
    system.query(
            query => 'select * from orcavault.mart.lims where sequencing_run_id = ''250328_A01052_0258_AHFGM7DSXF'' order by sequencing_run_date desc limit 10'
        ));
```
