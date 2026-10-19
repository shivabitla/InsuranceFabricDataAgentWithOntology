# TMDL Table Definitions

These files define the seven tables in the local `InsuranceSalesStarSchema` model. All partitions contain calculated DAX sample data, not connections to the root CSVs or Fabric Lakehouse.

| Definition | Grain | Sample rows |
| --- | --- | ---: |
| [DimAgent.tmdl](DimAgent.tmdl) | Selling agent | 5 |
| [DimCarrier.tmdl](DimCarrier.tmdl) | Insurance carrier | 5 |
| [DimCustomer.tmdl](DimCustomer.tmdl) | Policyholder | 5 |
| [DimProduct.tmdl](DimProduct.tmdl) | Insurance product | 5 |
| [DimDate.tmdl](DimDate.tmdl) | Calendar day, January 1 through December 31, 2025 | 365 |
| [FactPolicySales.tmdl](FactPolicySales.tmdl) | Policy issuance or renewal transaction | 5 |
| [FactClaims.tmdl](FactClaims.tmdl) | Current claim record | 6 |

The fact tables share the five dimensions. Claims use loss date as the active date relationship and reported date as an inactive relationship. The relationships are defined in [../relationships.tmdl](../relationships.tmdl), and table references are registered in [../model.tmdl](../model.tmdl).

The current CSV data is larger and uses 2026 dates. Editing a CSV does not update these embedded DAX samples. Keep model columns, source expressions, and relationships consistent when replacing the partitions. See [../../semantic-model.md](../../semantic-model.md) for the CSV comparison, measures, and date-filter examples, or [../../README.md](../../README.md) for the repository inventory and Fabric export provenance.
