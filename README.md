# Insurance POC Ontology

Fabric workspace: `sbfabiq`  
Lakehouse: `InsuranceDataLH`  
Ontology: `InsurnaceOntlogy` (Fabric item ID `df3ab284-7783-4196-a652-95276c448a7d`)  
Semantic model: `InsuranceSalesStarSchema`

This repository contains portable exports and documentation for the insurance policy-sales proof of concept created in the Fabric workspace. The data is synthetic sample data.

## Lakehouse Tables

| Table | Role | Expected rows |
| --- | --- | ---: |
| `dbo.dimagent` | Agent dimension | 5 |
| `dbo.dimcarrier` | Insurance carrier dimension | 5 |
| `dbo.dimcustomer` | Customer dimension | 5 |
| `dbo.dimdate` | 2026 date dimension | 12 |
| `dbo.dimproduct` | Insurance product dimension | 5 |
| `dbo.factpolicysales` | Policy sale fact | 50 |

The six corresponding seed CSVs are included at the repository root. In the Lakehouse, the originals are under `Files/InsuranceSalesSeed/`.

Local CSV extensions: the seeds now contain 10 rows in each non-date dimension, 100 policy sales, and 24 claims. The date dimension contains 24 unique 2026 dates (the 1st and 15th of each month), with matching sale, loss, and reported date keys in the facts. Existing identifiers are preserved. These local changes have not been uploaded to the Lakehouse or added to the live ontology; the table above records the original deployed seed counts. See [schema-diagram.md](schema-diagram.md) for the combined policy-sales and claims schema.

## Star Relationships

- `factpolicysales.AgentKey` -> `dimagent.AgentKey` (`soldBy`)
- `factpolicysales.CustomerKey` -> `dimcustomer.CustomerKey` (`purchasedBy`)
- `factpolicysales.ProductKey` -> `dimproduct.ProductKey` (`covers`)
- `factpolicysales.CarrierKey` -> `dimcarrier.CarrierKey` (`underwrittenBy`)
- `factpolicysales.SaleDateKey` -> `dimdate.DateKey` (`soldOn`)

The ontology models these same relationships with `PolicySale` as the origin entity and the dimensions as targets. `PolicySale` properties are bound to `factpolicysales`; each dimension entity is bound to its corresponding `dim*` table.

## Semantic Model

`InsuranceSalesStarSchema` is a separate illustrative semantic model. Its native TMDL export is in `semantic-model/`. It currently contains five sample fact rows with 2025 dates; it is not connected to the Lakehouse's 50-row 2026 seed tables. The exported model's dimensions and relationships document the intended star schema, but do not represent the current Lakehouse data.

The local TMDL model now also includes `FactClaims`, with six illustrative claims referencing its existing 2025 policies and dimensions. The policy-sales fact remains unchanged. Both facts share the five dimensions; claims use an active loss-date relationship and an inactive reported-date relationship.

## Ontology

The ontology follows the Lakeshore tutorial pattern: entity types with Lakehouse property bindings and explicit relationships. `ontology.ttl` is the Fabric-generated Turtle export. `ontology-model.json` is a companion manifest that records the fact-to-dimension key mappings; it is not a Fabric import package.

## Contents

- `DimAgent.csv`, `DimCarrier.csv`, `DimCustomer.csv`, `DimDate.csv`, `DimProduct.csv`, `FactPolicySales.csv`: synthetic seed data.
- `semantic-model.md`: semantic model and measure notes.
- `ontology.ttl`: native RDF/Turtle ontology export from Fabric.
- `ontology-model.json`: entity types, source bindings, and relationship key mappings.
- `fabric-artifacts.json`: Fabric workspace/item identifiers and export provenance.
