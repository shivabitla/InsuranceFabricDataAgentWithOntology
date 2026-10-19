# Insurance POC Ontology

Fabric workspace: `sbfabiq`  
Lakehouse: `InsuranceDataLH`  
Ontology: `InsurnaceOntlogy` (Fabric item ID `df3ab284-7783-4196-a652-95276c448a7d`)  
Semantic model: `InsuranceSalesStarSchema`

This repository contains synthetic insurance policy-sales and claims data, a shared-dimension schema, and portable exports from the original Fabric proof of concept. Repository updates do not deploy or refresh Fabric resources.

## Current Local Data

| CSV | Role | Rows |
| --- | --- | ---: |
| [DimAgent.csv](DimAgent.csv) | Selling agents | 10 |
| [DimCarrier.csv](DimCarrier.csv) | Insurance carriers | 10 |
| [DimCustomer.csv](DimCustomer.csv) | Policyholders | 10 |
| [DimProduct.csv](DimProduct.csv) | Insurance products | 10 |
| [DimDate.csv](DimDate.csv) | 1st and 15th of every month in 2026 | 24 |
| [FactPolicySales.csv](FactPolicySales.csv) | Policy issuance or renewal transactions | 100 |
| [FactClaims.csv](FactClaims.csv) | Current claim records | 24 |

All CSV sale, loss, and reported dates are in 2026 and reference the date dimension. Existing identifiers are preserved, including policy and claim numbers with a `27` prefix; the prefix does not determine a record's date. All data is synthetic, including future-dated sample transactions.

See [the schema diagram](schema-diagram.md) for table columns, keys, and relationships.

## Original Fabric Export

The following counts describe the original deployed seeds, not the current CSV files or a fresh check of the live workspace.

| Table | Role | Original seed rows |
| --- | --- | ---: |
| `dbo.dimagent` | Agent dimension | 5 |
| `dbo.dimcarrier` | Insurance carrier dimension | 5 |
| `dbo.dimcustomer` | Customer dimension | 5 |
| `dbo.dimdate` | 2026 date dimension | 12 |
| `dbo.dimproduct` | Insurance product dimension | 5 |
| `dbo.factpolicysales` | Policy sale fact | 50 |

The original Lakehouse seed files were stored under `Files/InsuranceSalesSeed/`. The expanded local CSVs and claims fact have not been uploaded to the Lakehouse or added to the live ontology. [fabric-artifacts.json](fabric-artifacts.json) records the original item identifiers and the local ontology extension's provenance, not the current local data inventory.

## Star Relationships

Both facts share `DimAgent`, `DimCustomer`, `DimProduct`, and `DimCarrier` through their corresponding `*Key` columns. Each dimension has a one-to-many relationship to each fact.

- `FactPolicySales.SaleDateKey` -> `DimDate.DateKey`.
- `FactClaims.LossDateKey` -> `DimDate.DateKey` (active in TMDL).
- `FactClaims.ReportedDateKey` -> `DimDate.DateKey` (inactive in TMDL).

`FactPolicySales` has one row per issuance or renewal transaction. `FactClaims` has one row per claim, storing current status, cumulative payments, and the remaining reserve; it is not a payment history. `AgentKey` identifies the policy's selling agent, not a claims adjuster.

`PolicyNumber` provides business traceability between the facts, not a direct fact-to-fact relationship. Multiple claims may reference one policy. Amounts are USD; incurred claims equal `PaidAmount + OutstandingReserve`, which need not equal the amount requested.

## Semantic Model

`InsuranceSalesStarSchema` is a separate illustrative model defined in [semantic-model/model.tmdl](semantic-model/model.tmdl). Its calculated partitions contain 5 members per non-date dimension, a 365-day calendar for 2025, 5 policy sales, and 6 claims. The CSV expansions are not reflected in those partitions.

The model is not connected to the local CSVs or the Lakehouse. Replace its partitions with the intended data source and refresh before treating it as synchronized. See [semantic-model.md](semantic-model.md) for its inventory, measures, and date-filter behavior.

## Ontology

The repository ontology extends the original Fabric export with a `Claim` entity containing all 14 columns from the claims CSV. It now describes seven entity types and eleven relationships. [ontology.ttl](ontology.ttl) is the locally extended RDF/Turtle file, not an untouched native export or a Fabric deployment package. [ontology-model.json](ontology-model.json) records the source-column join mappings as a descriptive manifest, not an import package.

| Claim relationship | Source column | Target |
| --- | --- | --- |
| `policyAgent` | `AgentKey` | `Agent.AgentKey` |
| `claimedBy` | `CustomerKey` | `Customer.CustomerKey` |
| `claimedProduct` | `ProductKey` | `Product.ProductKey` |
| `claimCarrier` | `CarrierKey` | `Carrier.CarrierKey` |
| `lossOn` | `LossDateKey` | `Date.DateKey` |
| `reportedOn` | `ReportedDateKey` | `Date.DateKey` |

Shared property names such as `PolicyNumber` and dimension keys are qualified for `Claim` in Turtle, preserving the existing `PolicySale` property domains. The two claim date relationships are ontology links; active/inactive date filtering is a TMDL behavior, not an RDF relationship setting. Claim amount properties use `xsd:decimal` for USD values.

The proposed `dbo.factclaims` source and its joins are local definitions only. They have not been created or bound in the live Fabric ontology. Provision the claims table and configure its entity property bindings and relationship joins in Fabric before querying live claims through the ontology. The repository does not contain a configured or deployed data agent.

## Working With the Data

Import the seven root CSVs as separate tables, preserve the column names, and configure the relationships shown in [schema-diagram.md](schema-diagram.md). Use integer types for surrogate and date keys, text for identifiers, and fixed-decimal types for monetary amounts.

Before loading changed data, verify unique dimension and fact keys, valid dimension/date references, and that each claim's policy exists with matching dimension keys. Check `SaleDateKey <= LossDateKey <= ReportedDateKey`. Closed or denied sample claims have zero outstanding reserve.

To inspect CSV row counts locally with PowerShell:

```powershell
Get-ChildItem -Filter *.csv | ForEach-Object {
	[pscustomobject]@{ Table = $_.BaseName; Rows = @(Import-Csv $_.FullName).Count }
}
```

## Contents

- Seven CSVs listed above: current synthetic seed data.
- [schema-diagram.md](schema-diagram.md): combined policy-sales and claims schema.
- [semantic-model.md](semantic-model.md): illustrative model inventory and measure notes.
- [semantic-model/tables/README.md](semantic-model/tables/README.md): per-table TMDL guide.
- [ontology.ttl](ontology.ttl): original RDF/Turtle ontology extended locally with claims.
- [ontology-model.json](ontology-model.json): entity source columns and relationship join mappings, including the proposed claims source.
- [fabric-artifacts.json](fabric-artifacts.json): original Fabric identifiers and local extension provenance.
