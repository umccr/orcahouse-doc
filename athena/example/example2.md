# Example 2

<!-- TOC -->
* [Example 2](#example-2)
  * [Virtual Cohort](#virtual-cohort)
  * [BAM Inventory](#bam-inventory)
  * [Read Inventory](#read-inventory)
<!-- TOC -->


See the model guide [dbt](https://umccr.github.io/orcahouse-doc/dbt/orcavault/#!/model/model.orcavault.lims) and [erd](https://umccr.github.io/orcahouse-doc/erd/) for schema and column details.


## Virtual Cohort

```sql
-- list all unique cohort based on BAM storage
select distinct cohort_id from bam;
```

```sql
-- find me tumor/normal bam for wgs accreditation run
select 
    * 
from 
    lims
        join workflow on lims.library_id = workflow.library_id
        join bam on bam.library_id = workflow.library_id and bam.portal_run_id = workflow.portal_run_id
where 
    bam.cohort_id = 'wgs-accreditation-prod'
    and bam.format = 'bam'
    and workflow.workflow_name in ('tumor-normal', 'wgs_tumor_normal');
```

```sql
-- list BAM storage size and file count by Cohort
select 
    (sum(size) / 1099511627776) as TOTAL_SIZE_TB,
    count(1) as BAM_FILE_COUNT
from bam 
where 
    cohort_id = 'production';
```

## BAM Inventory

```sql
-- find all BAM by given LibraryID
select 
    * 
from bam
where
    library_id in ('L2500134', 'L2400667');
```

_NOTE: SQL [subquery](https://www.google.com/search?q=sql+subquery) demo instead of table [JOIN](https://www.google.com/search?q=sql+JOIN) query._
```sql
-- find all BAM by given LIMS InternalSubjectID
select 
    * 
from bam
where
    library_id in (
        select distinct library_id from lims where internal_subject_id = 'SBJ05033'
    )
order by portal_run_id;
```

```sql
-- total BAM storage size and file count produced by the production workflow
select
    workflow.workflow_name as WORKFLOW_NAME,
    sum(bam.size) / 1099511627776 as BAM_TOTAL_TB,
    count(bam.filename) as BAM_FILE_COUNT
from workflow
    join bam on bam.portal_run_id = workflow.portal_run_id and bam.library_id = workflow.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and bam.format = 'bam'
    and bam.cohort_id = 'production'
group by workflow.workflow_name
order by BAM_TOTAL_TB desc;
```

```sql
-- total BAM storage size and file count produced by cohort workflow
select
    bam.cohort_id as COHORT_ID,
    workflow.workflow_name as WORKFLOW_NAME,
    sum(bam.size) / 1099511627776 as BAM_TOTAL_TB,
    count(bam.filename) as BAM_FILE_COUNT
from workflow
    join bam on bam.portal_run_id = workflow.portal_run_id and bam.library_id = workflow.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and bam.format = 'bam'
group by bam.cohort_id, workflow.workflow_name
order by BAM_TOTAL_TB desc;
```

## Read Inventory

```sql
-- total storage size and file count for FASTQ
select 
    (sum(size) / 1099511627776) as TOTAL_TB, 
    count(1) as FILE_COUNT 
from fastq;
```

```sql
-- total storage size and file count for BAM
select 
    (sum(size) / 1099511627776) as TOTAL_TB, 
    count(1) as FILE_COUNT 
from bam;
```

```sql
-- find top 10 BAM by size
select
    cohort_id,
    bucket,
    key,
    library_id,
    format,
    size,
    (size / 1073741824) as size_in_gb,
    storage_class
from bam
where 
    format = 'bam' 
order by size desc 
limit 10;
```

```sql
-- find me all buckets for FASTQ data storage
select distinct bucket from fastq;
```

```sql
-- find me all buckets for BAM data storage
select distinct bucket from bam;
```

```sql
-- find total size and file count from the active pipeline cache bucket for FASTQ
select 
    (sum(size) / 1099511627776) as TOTAL_SIZE_TB, 
    count(1) as FASTQ_FILE_COUNT 
from fastq 
where 
    bucket = 'pipeline-prod-cache-503977275616-ap-southeast-2';
```

```sql
-- find total size and file count from the pipeline archive bucket for FASTQ
select 
    (sum(size) / 1099511627776) as TOTAL_SIZE_TB, 
    count(1) as FASTQ_FILE_COUNT 
from fastq 
where 
    bucket = 'archive-prod-fastq-503977275616-ap-southeast-2';
```

```sql
-- find me all FASTQ files older than the specified sequencing date in ORA format
select 
    * 
from fastq 
where
    format = 'ora'
    and sequencing_run_date < cast('2024-12-30' as date);
```

```sql
-- find me all FASTQ files older than the specified sequencing date in GZ format
select 
    * 
from fastq 
where
    format = 'gz'
    and sequencing_run_date < cast('2024-12-30' as date);
```
_NOTE: `cohort_id` for the reasoning about storage context_

```sql
-- what is the retention due date from now?
select current_date - interval '84' day as retention_due_date;
```

```sql
-- find me all FASTQ file in pipeline cache bucket and ORA format that is due for archival retention
select
    * 
from fastq 
where 
    sequencing_run_date < current_date - interval '84' day
    and cohort_id = 'production'
    and format = 'ora' 
    and bucket = 'pipeline-prod-cache-503977275616-ap-southeast-2';
```

```sql
-- find me all FASTQ file in pipeline cache bucket and GZ format that is due for retention
select 
    * 
from fastq 
where 
    sequencing_run_date < current_date - interval '84' day 
    and format = 'gz'
    and bucket = 'pipeline-prod-cache-503977275616-ap-southeast-2';
```

```sql
-- find me total size in TB and file count in pipeline cache production storage context for FASTQ GZ file
select
    coalesce((sum(size) / 1099511627776), 0) as TOTAL_GZ_SIZE_TB,
    count(1) as FASTQ_GZ_FILE_COUNT
from
    fastq
where
    format = 'gz'
    and bucket = 'pipeline-prod-cache-503977275616-ap-southeast-2'
    and cohort_id = 'production';
```

```sql
-- find me FASTQ total size in TB and file count by Project (ProjectName) as in LIMS
select
    distinct project_id as PROJECT_ID,
    sum(fastq.size) / 1099511627776 as FASTQ_TOTAL_TB,
    count(1) as FASTQ_COUNT
from lims 
    join fastq on lims.library_id = fastq.library_id and lims.sequencing_run_id = fastq.sequencing_run_id
group by project_id
order by FASTQ_TOTAL_TB desc;
```

```sql
-- find me FASTQ total size in TB and file count by Owner (ProjectOwner) as in LIMS
select
    distinct owner_id as OWNER_ID,
    sum(fastq.size) / 1099511627776 as FASTQ_TOTAL_TB,
    count(1) as FASTQ_COUNT
from lims 
    join fastq on lims.library_id = fastq.library_id and lims.sequencing_run_id = fastq.sequencing_run_id
group by owner_id
order by FASTQ_TOTAL_TB desc;
```
