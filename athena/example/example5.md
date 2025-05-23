# Example 5

<!-- TOC -->
* [Example 5](#example-5)
  * [Select](#select)
  * [Function](#function)
  * [Subquery](#subquery)
  * [Join](#join)
  * [CTE](#cte)
<!-- TOC -->


In this example, we detour and learn a few SQL techniques. SQL is expressive and declarative.


## Select

```sql
-- you can always select 1 integer value regardless
select 1;
```

```sql
-- and perform an arithmetic operation
select 1 + 1;
```

```sql
-- and select some string
select 'some string';
```

```sql
-- this comes in handy when you want to create an inline label column
select 
    (select 'some string') as MY_LABEL_COLUMN,
    *
from lims
limit 10;
```

```sql
-- and transform some date-formatted string into a date, called "type casting"
select cast('2025-02-01' as date);
```

```sql
-- and combine two select results
select 1
union
select 2;
```

```sql
-- and compare (diff) two select results
select 1
except
select 1;
-- return 0 as no differences
```

To put into use, for example, the [requested](https://umccr.slack.com/archives/CP356DDCH/p1746170088163169) longitudinal re-analysis cases.

```sql
-- find me LIMS records for given SampleID and label them
select 
    (select 'Patient 1') as patient_label, 
    * 
from lims 
where 
    sample_id in ('MDX190012', 'MDX190003', 'MDX220074', 'MDX250208')

union all

select 
    (select 'Patient 2') as patient_label, 
    * 
from lims 
where 
    sample_id in ('MDX200051', 'MDX200052', 'MDX250209');
```

## Function

You can leverage [database function](https://www.google.com/search?q=database+function) where supported. There are plenty of [functions](https://docs.aws.amazon.com/athena/latest/ug/functions-env3.html) supported by AWS Athena as well as PostgresSQL.

```sql
-- select some string into all uppercase
select upper('some string');
```

```sql
-- select the year component of a date
select year(cast('2025-02-01' as date));
```

## Subquery

https://neon.tech/postgresql/postgresql-tutorial/postgresql-subquery

```sql
-- you can stack up a query within a query, hence, subquery
select 
    * 
from workflow 
where 
    library_id in (
        select distinct library_id from lims where internal_subject_id in ('SBJ00057', 'SBJ00306')
    ) 
    and workflow_status = 'SUCCEEDED';
```

## Join

https://neon.tech/postgresql/postgresql-tutorial/postgresql-joins

```sql
-- or join the two tables on the candidate column(s) that have them merged
select 
    * 
from workflow
    join lims on lims.library_id = workflow.library_id
where 
    workflow.workflow_status = 'SUCCEEDED'
    and lims.internal_subject_id in ('SBJ00057', 'SBJ00306');
```
_Recall Venn diagram and Set theory :). And the [cheat sheet](https://www.google.com/search?q=sql+join+cheat+sheet)._

## CTE

https://neon.tech/postgresql/postgresql-tutorial/postgresql-cte

```sql
-- how to remember something so that you can reference it in the next step
with cte1 as (
    select 100 as MY_VALUE
)
select * from cte1;
```

```sql
-- you can chain the CTE blocks
with cte1 as (
    select 100 as MY_VALUE
),
cte2 as (
    select MY_VALUE * 2 as MY_VALUE_2X from cte1
)
select * from cte2;
```

```sql
-- CTE can be syntactic sugar
with requested as (
    select library_id from lims where internal_subject_id in ('SBJ00057', 'SBJ00306')
),
succeeded as (
    select * from workflow where library_id in (
        select distinct library_id from requested
    ) and workflow_status = 'SUCCEEDED'
)
select * from succeeded;
```

```sql
-- CTE can be put into powerful use
with cte1 as (
    select 
        'ICAv1' as platform_version, 
        avg(workflow_duration) as tn_avg_runtime_seconds 
    from workflow 
    where 
        workflow_status = 'SUCCEEDED' 
        and workflow_name = 'wgs_tumor_normal'
),
cte2 as (
    select 
        'ICAv2' as platform_version, 
        avg(workflow_duration) as tn_avg_runtime_seconds 
    from workflow 
    where 
        workflow_status = 'SUCCEEDED' 
        and workflow_name = 'tumor-normal'
        and workflow_start > cast('2025-02-01' as date) -- trim outlier
)
select * from cte1
union
select * from cte2;
```
