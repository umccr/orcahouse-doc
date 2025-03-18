# OrcaHouse Documentation

- **Website**: [https://umccr.github.io/orcahouse-doc](https://umccr.github.io/orcahouse-doc)
- **GitHub**: [https://github.com/umccr/orcahouse-doc](https://github.com/umccr/orcahouse-doc)
- **Description**: Documentation for [OrcaHouse](https://github.com/umccr/orcahouse)

## Navigation

* [arch](arch)
* [athena](athena)
* [erd](erd)
* [dbt](dbt)
  * [orcavault](dbt/orcavault)

## FAQ

### What is OrcaHouse?

OrcaHouse is the project. The project comprises Enterprise Data Warehouses (EDW). The term "OrcaHouse" kept as high level project concept.

### What is the high level architecture?

OrcaHouse mainly follows implementation of "[Enterprise Data Warehouse](https://www.google.com/search?q=Enterprise+Data+Warehouse)" architecture. Alternate would be "[Data LakeHouse](https://www.google.com/search?q=LakeHouse+architecture)" architecture. There are subtle differences in how you approach to data modelling based on the architecture choice being made.

### What is OrcaVault?

OrcaVault is the first EDW of OrcaHouse. Its main objective is to build the data warehouse need of our upstream [OrcaBus](https://github.com/umccr/orcabus) system.

### What problem OrcaVault solve?

As mentioned, our upstream [OrcaBus](https://github.com/umccr/orcabus) system is based on "Event-driven" "Microservice" architecture. The system comprises many smaller-sized transactional databases. These microservices may come and go (being replaced) as it pivots to new business requirement. OrcaVault EDW aims to capture, consolidate, observe, archive and history track these changes. Provide business intelligence and audit trail.

### What is data flow looking like?

Please read high level [architecture note](arch) to understand the overall data flow and data layers within the warehouse.

### What is the federated query interface looking like?

Federated query is possible via [OrcaHouse Athena](athena) setup.
