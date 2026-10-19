# TMDL Table Definitions

These files export the seven tables in the deployed `InsurancePolicySalesAndClaimsSchema` model. All partitions use Direct Lake entity sources pointing to lowercase `dbo` tables in `InsurancePolicySalesAndClaimsLH` in `sbfabricInsDataAgent`.

| Definition | Grain | Verified rows |
| --- | --- | ---: |
| [DimAgent.tmdl](DimAgent.tmdl) | Selling agent | 10 |
| [DimCarrier.tmdl](DimCarrier.tmdl) | Insurance carrier | 10 |
| [DimCustomer.tmdl](DimCustomer.tmdl) | Policyholder | 10 |
| [DimProduct.tmdl](DimProduct.tmdl) | Insurance product | 10 |
| [DimDate.tmdl](DimDate.tmdl) | 1st and 15th of each month in 2026 | 24 |
| [FactPolicySales.tmdl](FactPolicySales.tmdl) | Policy issuance or renewal transaction | 100 |
| [FactClaims.tmdl](FactClaims.tmdl) | Current claim record | 24 |

The fact tables share the five dimensions. Claims use loss date as the active date relationship and reported date as an inactive relationship. The relationships are defined in [../relationships.tmdl](../relationships.tmdl), and table references are registered in [../model.tmdl](../model.tmdl).

The shared source expression is defined in [../expressions.tmdl](../expressions.tmdl). Editing a CSV does not update the deployed tables: reload the affected lakehouse table and refresh the model. See [../../semantic-model.md](../../semantic-model.md) for measures and date-filter examples, or [../../README.md](../../README.md) for the repository inventory and Fabric deployment provenance.
