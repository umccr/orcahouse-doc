# OrcaHouse Athena

> WIP

Please read high level [architecture note](../arch) to understand the overall data flow and data pipeline within warehouse.

**OrcaHouse Athena** follows similar setup from its predecessor [Portal Athena](https://github.com/umccr/data-portal-apis/tree/dev/docs/athena) setup.

Use Athena setting as follows.

```
Workgroup:      orcahouse
Data Source:    orcavault
Database:       ods, tsa, psa, dcl, mart
```

> NOTE: **Early Access Program**
> * At the mo, OrcaVault warehouse is WIP and release as early preview for "how it feels like..." and gather feedbacks.
> * Thera are sudden parts that are stable and, some are still moving targets.
> 
> For **Developer**;
> * For general downstream usage, you should consume from `PSA`, `DCL` or data mart layer.
> * Both `ODS` and `TSA` will get breaking schema changes, full refresh and reload at upstream. 
>   * We resolve and consolidate these issues at OrcaVault data warehouse. From this warehouse, we provide you with "managed" self-service BI and reporting purpose.
>   * You will need to handle these implications if used them directly.
> 
> For **User**;
> * Please use the query only as guided by developer. _(Or, you can cross-check with Victor for the moment)_
> * There are still _planning in progress_ for DCL and Data Mart layer; which aims for more end User use cases.
> * You are, however, feel free to explore & browse any schemas.
> 
> All schemas are accessible as **Read Only**.


## Query


### Federated query

It is best practise to use full data source pointer when you query.

```
select * from <data_source>.<database>.<table>;
```

Equivalently in native PostgreSQL, this maps as follows.

```
select * from <database>.<schema>.<table>;
```

See [https://docs.aws.amazon.com/athena/latest/ug/understanding-tables-databases-and-the-data-catalog.html](https://umccr.github.io/orcahouse-doc/dbt/)

### Passthrough query

For some reason, if you'd like to run native PostgreSQL query statement, you can wrap to `table()` function like so. 

```
select * from table(
    system.query(
            query => 'select * from orcavault.raw.hub_library order by load_datetime limit 10'
        ));
```

See [https://docs.aws.amazon.com/athena/latest/ug/connectors-postgresql.html#connectors-postgres-passthrough-queries](https://umccr.github.io/orcahouse-doc/dbt/)

_*Side note: This resembles [GA4GH Data Connect](https://www.google.com/search?q=ga4gh+data+connect) concept. FYI._

## Portal Migration

> NOTE: 
> * During Portal system migration period, you can refer to contents from [Portal Athena](https://github.com/umccr/data-portal-apis/tree/dev/docs/athena) documentation where applicable.
> * Eventually, we will migrate Portal Athena documentation to this repo, if any.

* All Portal tables are mounted via FDW at ODS database schema.
* Once Portal is decommissioned, its database will be permanently archived under OrcaVault **PSA** schema.

```
select * from orcavault.ods.data_portal_workflow order by id limit 10;
```
