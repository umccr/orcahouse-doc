# Example 1

<!-- TOC -->
* [Example 1](#example-1)
  * [Weekly Runs](#weekly-runs)
  * [Tumor Normal](#tumor-normal)
  * [By Sequencer](#by-sequencer)
  * [Unique Project and Owner](#unique-project-and-owner)
  * [Unique Type and Assay](#unique-type-and-assay)
<!-- TOC -->


See the model guide [dbt](https://umccr.github.io/orcahouse-doc/dbt/orcavault/#!/model/model.orcavault.lims) and [erd](https://umccr.github.io/orcahouse-doc/erd/) for schema and column details.


## Weekly Runs

```sql
-- find me what cases should be preformed for WGS analysis from the weekly sequencing?
select 
    * 
from lims 
where 
    sequencing_run_id in ('250509_A01052_0262_BHFGJWDSXF') 
    and type = 'WGS' 
    and workflow in ('clinical', 'research')
    and phenotype in ('tumor', 'normal')
order by internal_subject_id;
```

```sql
-- find me what cases should be preformed for WTS analysis from the weekly sequencing?
select 
    * 
from lims 
where 
    sequencing_run_id in ('250509_A01052_0262_BHFGJWDSXF')
    and type = 'WTS' 
    and workflow in ('clinical', 'research')
    and phenotype in ('tumor')
order by internal_subject_id;
```

```sql
-- find me what cases should be preformed for ctTSO analysis from the weekly sequencing?
select 
    * 
from lims 
where
    sequencing_run_id in ('250417_A01052_0261_AHFHG7DSXF')
    and assay in ('ctTSO', 'ctTSOv2')
    and workflow in ('clinical', 'research')
    and phenotype in ('tumor')
order by internal_subject_id;
```

```sql
-- list all ctTSO assay samples sorted by recent sequencing
select 
    * 
from lims 
where 
    assay in ('ctTSO', 'ctTSOv2')
order by sequencing_run_id desc;
```

```sql
-- find all libraries from the project owned by PI on the specified date range
select
    *
from lims
where
    sequencing_run_date between cast('2025-03-27' as date) and cast('2025-03-31' as date)
    and project_id = 'CUP'
    and owner_id = 'Tothill';
```

```sql
-- find all LIMS entries and FASTQ locations for given SequencingRunID and Owner
select
    lims.*,
    fastq.*
from lims
    join fastq on fastq.sequencing_run_id = lims.sequencing_run_id and fastq.library_id = lims.library_id
where 
    lims.sequencing_run_id in ('250328_A01052_0257_BHFFVFDSXF', '250328_A01052_0258_AHFGM7DSXF')
    and lims.owner_id = 'Tothill';
```

```sql
-- list all PI/Owner sorted alphabetically and null entries to the last
select distinct owner_id from lims order by owner_id nulls last;
```

## Tumor Normal

NOTE:
* This goes across searching for entries that belong to the InternalSubjectID from multiple sequencing runs. 
* It assumes InternalSubjectID can group the related samples that have been sequenced by the Centre.

```sql
-- find me WGS tumor normal pair based on the Centre InternalSubjectID
select
    *
from lims
where
    type = 'WGS'
    and workflow in ('clinical', 'research')
    and phenotype in ('tumor', 'normal')
    and internal_subject_id = 'SBJ06470';
```

```sql
-- find me WGS tumor normal pair along with FASTQ in ORA format based on the Centre InternalSubjectID
select
    *
from lims
    join fastq on fastq.sequencing_run_id = lims.sequencing_run_id and fastq.library_id = lims.library_id
where
    fastq.format = 'ora'
    and lims.type = 'WGS'
    and lims.workflow in ('clinical', 'research')
    and lims.phenotype in ('tumor', 'normal')
    and lims.internal_subject_id = 'SBJ06470';
```

```sql
-- find me WGS tumor normal pair along with FASTQ in ORA format based on the Centre LibraryID that I have chosen
select
    *
from lims
    join fastq on fastq.sequencing_run_id = lims.sequencing_run_id and fastq.library_id = lims.library_id
where
    fastq.format = 'ora'
    and lims.type = 'WGS'
    and lims.workflow in ('clinical', 'research')
    and lims.phenotype in ('tumor', 'normal')
    -- and lims.internal_subject_id = 'SBJ06464'
    and lims.library_id in ('L2500458', 'L2500267');
```

## By Sequencer

```sql
-- find me the latest sequencing done by "PO"
select distinct 
    sequencing_run_id, 
    sequencing_run_date 
from lims 
where 
    sequencing_run_id like '%A01052%' 
order by sequencing_run_date desc;
```

```sql
-- find me the latest sequencing done by "Baymax"
select distinct 
    sequencing_run_id, 
    sequencing_run_date 
from lims 
where 
    sequencing_run_id like '%A00130%' 
order by sequencing_run_date desc;
```

```sql
-- list all successful sequencing done by "PO" in 2025 sorted the latest first
select distinct 
    sequencing_run_id 
from lims 
where 
    sequencing_run_id like '%A01052%' 
    and year(sequencing_run_date) = 2025
order by sequencing_run_id desc;
```

```sql
-- give me the successful sequencing run count for each year by "PO"
select
    year(sequencing_run_date) as YEAR,
    count(distinct sequencing_run_id) as RUN_COUNT
from lims 
where 
    sequencing_run_id like '%A01052%'
group by year(sequencing_run_date)
order by year(sequencing_run_date) desc;
```

```sql
-- give me the successful sequencing run count for each year by "Baymax"
select
    year(sequencing_run_date) as YEAR,
    count(distinct sequencing_run_id) as RUN_COUNT
from lims 
where 
    sequencing_run_id like '%A00130%'
group by year(sequencing_run_date)
order by year(sequencing_run_date) desc;
```

## Unique Project and Owner

```sql
-- list OWNER, PROJECT unique combination count sorted by highest row count
select distinct
    owner_id,
    project_id,
    count(1) as row_count
from lims
group by owner_id, project_id
order by count(1) desc;
```

```sql
-- find OWNER, PROJECT unique combination counts all matches to specific owner or project
select distinct
    owner_id,
    project_id,
    count(1) as row_count
from lims
where
    owner_id = 'Tothill' or lower(project_id) like '%tothill%'
group by owner_id, project_id
order by count(1) desc;
```

## Unique Type and Assay

```sql
-- find WTS library count group by sample TYPE, ASSAY sorted by assay
select 
    type, 
    assay, 
    count(1) as library_count 
from lims 
where 
    type = 'WTS' 
group by type, assay
order by assay;
```

```sql
-- find WGS sample unique TYPE, ASSAY combination
select distinct 
    type, 
    assay, 
    count(1) as row_count 
from lims
where 
    type = 'WGS'
group by type, assay
order by type, assay nulls last;
```

```sql
-- find all unique TYPE, ASSAY combination
select distinct 
    type, 
    assay, 
    count(1) as row_count 
from lims
group by type, assay
order by type, assay nulls last;
```
