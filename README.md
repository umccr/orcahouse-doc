# OrcaHouse Documentation

- **Website**: [https://umccr.github.io/orcahouse-doc](https://umccr.github.io/orcahouse-doc)
- **GitHub**: [https://github.com/umccr/orcahouse-doc](https://github.com/umccr/orcahouse-doc)
- **Description**: Documentation for [OrcaHouse](https://github.com/umccr/orcahouse)

## Navigation

* [arch](arch)
* [athena](athena)
* [dbt](dbt)
  * [orcavault](dbt/orcavault)

## FAQ

### What is OrcaHouse?

OrcaHouse is the project. The project comprises Enterprise Data Warehouses (EDW). The term "OrcaHouse" kept as high level project concept.

### What is OrcaVault?

OrcaVault is the first EDW of OrcaHouse. Its main objective is to build the data warehouse need of our upstream [OrcaBus](https://github.com/umccr/orcabus) system.

### What is data flow looking like?

Please read high level [architecture note](arch) to understand the overall data flow and data pipeline within the warehouse.

### What is the federated query interface looking like?

Federated query is possible via [OrcaHouse Athena](athena) setup.
