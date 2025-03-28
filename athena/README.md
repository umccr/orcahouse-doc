# OrcaHouse Athena

**OrcaHouse Athena** follows similar setup from its predecessor [Portal Athena](https://github.com/umccr/data-portal-apis/tree/dev/docs/athena) setup. Please read high level [architecture note](../arch) to understand the overall data flow and data pipeline within warehouse.

Use Athena setting as follows.

```
Workgroup:      orcahouse
Data Source:    orcavault
Database:       mart
```

## Query

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

See model guide [dbt](https://umccr.github.io/orcahouse-doc/dbt/orcavault/#!/model/model.orcavault.lims) and [erd](https://umccr.github.io/orcahouse-doc/erd/) for schema details and more...

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
