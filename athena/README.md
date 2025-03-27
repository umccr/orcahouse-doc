# OrcaHouse Athena

**OrcaHouse Athena** follows similar setup from its predecessor [Portal Athena](https://github.com/umccr/data-portal-apis/tree/dev/docs/athena) setup. Please read high level [architecture note](../arch) to understand the overall data flow and data pipeline within warehouse.

Use Athena setting as follows.

```
Workgroup:      orcahouse
Data Source:    orcavault
Database:       ods, tsa, psa, dcl, mart
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

Equivalently in native PostgreSQL, this [maps as follows](https://docs.aws.amazon.com/athena/latest/ug/understanding-tables-databases-and-the-data-catalog.html).

```sql
select * from <database>.<schema>.<table>;
```

### Passthrough query

For some reason, if you'd like to run [native passthrough PostgreSQL query](https://docs.aws.amazon.com/athena/latest/ug/connectors-postgresql.html#connectors-postgres-passthrough-queries) statement, you can wrap to `table()` function like so. 

```sql
select * from table(
    system.query(
            query => 'select * from orcavault.raw.hub_library order by load_datetime limit 10'
        ));
```
