# Example 4

<!-- TOC -->
* [Example 4](#example-4)
  * [Workflow by LibraryID](#workflow-by-libraryid)
  * [Workflow by LIMS Dimension](#workflow-by-lims-dimension)
  * [Workflow and BAM Inventory](#workflow-and-bam-inventory)
  * [Workflow Name](#workflow-name)
    * [IAP](#iap)
    * [ICAv1](#icav1)
    * [ICAv2](#icav2)
<!-- TOC -->


See the model guide [dbt](https://umccr.github.io/orcahouse-doc/dbt/orcavault/#!/model/model.orcavault.lims) and [erd](https://umccr.github.io/orcahouse-doc/erd/) for schema and column details.


## Workflow by LibraryID

```sql
-- find me all workflows (analysis / run) done by LibraryID
select * from workflow where library_id = 'L2400035';
```

```sql
-- list all unique workflow status sorted alphabetically null last
select distinct workflow_status from workflow order by workflow_status nulls last;
```

```sql
-- find me all workflows successfully done by LibraryID
select 
    * 
from workflow 
where 
    workflow_status = 'SUCCEEDED'
    and library_id = 'L2400035';
```

## Workflow by LIMS Dimension

```sql
-- find me all workflows successfully done by Centre's InternalSubjectID
select 
    * 
from lims
    join workflow on workflow.library_id = lims.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and lims.internal_subject_id = 'SBJ02402';
```

```sql
-- find me tumor normal workflows (all generations) successfully done by Centre's InternalSubjectID
select 
    * 
from lims
    join workflow on workflow.library_id = lims.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and workflow.workflow_name in ('wgs_tumor_normal', 'tumor-normal')
    and lims.internal_subject_id = 'SBJ02402';
```

```sql
-- find me transcriptome workflows (all generations) successfully done by LIMS sample type assay combo
select 
    * 
from lims
    join workflow on workflow.library_id = lims.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and workflow.workflow_name in ('wts_tumor_only', 'wts')
    and lims.type = 'WTS'
    and lims.assay = 'ISTRL';
```

```sql
-- find me all transcriptome workflows (all generations) successfully done by LIMS ProjectName "CUP"
select 
    * 
from lims
    join workflow on workflow.library_id = lims.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and workflow.workflow_name in ('wts_tumor_only', 'wts')
    and lims.type = 'WTS'
    and lims.project_id = 'CUP'
order by workflow_version;
```

```sql
-- find me all ctTSO workflows (all generations) successfully done by LIMS ProjectName "Phaedra"
select
    * 
from lims
    join workflow on workflow.library_id = lims.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and workflow.workflow_name in ('tso_ctdna_tumor_only', 'cttsov2')
    and lims.assay in ('ctTSO', 'ctTSOv2')
    and lims.project_id = 'Phaedra'
order by sequencing_run_date desc;
```

## Workflow and BAM Inventory

_Caveat: BAM files are routinely deleted by data retention policy. Hence, this example may be outdated. In this case, the query may return no result as zero BAM inventory. Please adjust to recent sample information or use [full join](https://www.google.com/search?q=sql+full+join)._

```sql
-- find me TN workflow output BAM by LibraryID
select 
    * 
from workflow
    full join bam on bam.portal_run_id = workflow.portal_run_id and bam.library_id = workflow.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and workflow.workflow_name in ('wgs_tumor_normal', 'tumor-normal')
    and workflow.library_id = 'L2400667';
```

```sql
-- find me TN workflow output BAM by InternalSubjectID
select 
    * 
from workflow
    full join bam on bam.portal_run_id = workflow.portal_run_id and bam.library_id = workflow.library_id
    join lims on lims.library_id = workflow.library_id
where
    workflow.workflow_status = 'SUCCEEDED'
    and workflow.workflow_name in ('wgs_tumor_normal', 'tumor-normal')
    and bam.format = 'bam'
    and lims.internal_subject_id = 'SBJ05033';
```

## Workflow Name

_See the following sections for naming convention across different analysis platform generations._

```sql
-- find me all unique workflow names (or analysis type) being done by Centre
select distinct workflow_name from workflow;
```

```sql
-- find me all umccrise workflow versions sorted by latest
select distinct 
    workflow_name, 
    workflow_version
from workflow 
where 
    workflow_name in ('umccrise')
order by workflow_version desc;
```

### IAP
_DEPRECATED: All cap workflow names have used during early days, during the "IAP" era. All workflow records are marked deleted._
```
BCL_CONVERT
GERMLINE
DRAGEN_WGS_QC
DRAGEN_WTS
DRAGEN_TSO_CTDNA
TUMOR_NORMAL
```

### ICAv1
_See [https://github.com/umccr/data-portal-apis/tree/dev/docs/pipeline](https://github.com/umccr/data-portal-apis/tree/dev/docs/pipeline) for **legacy** pipeline naming convention_

```
bcl_convert
wts_alignment_qc
wgs_alignment_qc
wgs_tumor_normal
wts_tumor_only
tso_ctdna_tumor_only
umccrise
rnasum
-
star_alignment
oncoanalyser_wgs
oncoanalyser_wts
oncoanalyser_wgts_existing_both
sash
```

### ICAv2
_See [https://github.com/OrcaBus/wiki/tree/main/orcabus-platform](https://github.com/umccr/data-portal-apis/tree/dev/docs/pipeline) (T.B.D)_
```
BclConvert
bsshFastqCopy
bclconvert-interop-qc
ora-compression
wgts-qc
tumor-normal
wts
cttsov2
pieriandx
umccrise
rnasum
-
oncoanalyser-wgts-dna
oncoanalyser-wgts-rna
oncoanalyser-wgts-dna-rna
sash
```
