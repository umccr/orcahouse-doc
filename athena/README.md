# OrcaHouse Athena

> WIP

Please read high level [architecture note](../arch) to understand the overall data flow and data pipeline within warehouse.

**OrcaHouse Athena** follows similar setup from its predecessor [Portal Athena](https://github.com/umccr/data-portal-apis/tree/dev/docs/athena) setup.

> NOTE: 
> * During Portal system migration period, you can refer to contents from Portal Athena^^ documentation where applicable.
> * Eventually, we will migrate Portal Athena documentation to this repo.

Use Athena setting as follows.

```
Workgroup:      orcahouse
Data Source:    orcavault
Database:       ods, tsa, psa, raw
```

> NOTE: **Early Access Program**
> * At the mo, OrcaVault warehouse is WIP and release as early preview for "how it feels like..." and gather feedbacks.
> * Thera are sudden parts that are stable and, some are still moving targets.
> 
> For **Developer**;
> * For general downstream usage, you should consume from `PSA`, `RAW` or `DCL` layer.
> * Both `ODS` and `TSA` will get breaking schema changes, full refresh and reload at upstream. 
>   * We solve these issues at OrcaVault (hence, intermediate layers `PSA`, `RAW`, etc.). 
>   * You will need to handle these if used directly.
> 
> For **User**;
> * Please use the query only as guided by developer. _(Or, you can cross-check with Victor for the moment)_
> * There are still _planning in progress_ for DCL and Data Mart layer; which aims for more end User use cases.
> * You are, however, feel free to explore & browse any schemas.
> 
> All schemas are accessible as **Read Only**.


## Query

- Give me current status about `L2401697`.
```
select * from orcavault.tsa.spreadsheet_library_tracking_metadata where library_id = 'L2401697';
```

- Give me history about `L2401697` as-is Library tracking spreadsheet history. What has been changed? And when?
```
select * from orcavault.psa.spreadsheet_library_tracking_metadata where library_id = 'L2401697' order by load_datetime;
```

- How that has been currently recorded in "post" sequencing run LIMS tracking sheet?
```
select * from orcavault.tsa.spreadsheet_google_lims where library_id = 'L2401697';
```

- Anyone made changes to "post" sequencing run LIMS tracking sheet, since? Show me spreadsheet changes history?
```
select * from orcavault.psa.spreadsheet_google_lims where library_id = 'L2401697' order by load_datetime;
```

- These information flows into pipeline orchestration, the "OrcaBus" and some microservice applications and _legacy_ orchestration systems, etc. All these systems have slightly different perspective about the business key in question (e.g. `LibraryID`).
- Can we query about how it all linking up, and it gets changed history going through these systems?
  - Yes, this happens at `RAW_VAULT` layer which is actively developing at the mo. Have a browse to [ERD](../erd) model doc.
    - Please note though that this `RAW_VAULT` is not the `FINAL` expectation/view of what typically warehouse / reporting user get.
    - There are another `BUSINESS_VAULT` (or T.B.D models) that is planning to build out of these foundation data layers `PSA`, `RAW`.


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

* All Portal tables are mounted via FDW at ODS database schema.
* Once Portal is decommissioned, its database will be permanently archived under OrcaVault **PSA** schema.

```
select * from orcavault.ods.data_portal_workflow order by id limit 10;
```
