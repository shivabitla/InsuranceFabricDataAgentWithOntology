# Insurance POC Ontology

- Fabric workspace: `sbfabricInsDataAgent`
- Lakehouse: `InsurancePolicySalesAndClaimsLH`
- Ontology: `InsurancePolicySalesAndClaimsOntlogy`
- Semantic model: `InsurancePolicySalesAndClaimsSchema` (Direct Lake)

This repository contains synthetic insurance policy-sales and claims data, a shared-dimension schema, and exports of the deployed Fabric model and ontology. The original `sbfabiq` workspace was used as a read-only reference and remains unchanged. Repository updates do not automatically deploy or refresh Fabric resources.

## Deployed Resources

- [Semantic model](https://app.powerbi.com/groups/d2e5a491-0114-4333-8f2a-e912ed564c3a/datasets/96bda898-d5ec-481f-bce7-dccd6c93b99c/details): seven Direct Lake tables, seven measures, and eleven relationships.
- [Ontology](https://app.fabric.microsoft.com/groups/d2e5a491-0114-4333-8f2a-e912ed564c3a/ontologies/a0af8c51-62a3-47b3-a71e-92b6fe17a5c5): seven entities and eleven relationships, including claims.
- [Lakehouse](https://app.fabric.microsoft.com/groups/d2e5a491-0114-4333-8f2a-e912ed564c3a/lakehouses/d982dd11-3041-4a8e-a924-fba1ed3ab29b): seven Delta tables loaded from the root CSVs, with seed files under `Files/`.

All model and ontology table bindings point to the destination lakehouse. [fabric-artifacts.json](fabric-artifacts.json) records current identifiers, original reference identifiers, and verification results. On October 19, 2026, DAX queries confirmed 100 sales, 24 claims, USD 151,000 premium, USD 79,700 paid claims, USD 358,200 outstanding reserve, and USD 437,900 incurred claims. Ontology creation and saved bindings were verified; interactive graph browsing and graph query results were not verified. No data agent has been configured.

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

The original `sbfabiq` Lakehouse seed files were stored under `Files/InsuranceSalesSeed/`. Those original resources remain unchanged. The expanded CSVs and claims fact were loaded into the new destination lakehouse and bound to the new ontology.

## Star Relationships

Both facts share `DimAgent`, `DimCustomer`, `DimProduct`, and `DimCarrier` through their corresponding `*Key` columns. Each dimension has a one-to-many relationship to each fact.

- `FactPolicySales.SaleDateKey` -> `DimDate.DateKey`.
- `FactClaims.LossDateKey` -> `DimDate.DateKey` (active in TMDL).
- `FactClaims.ReportedDateKey` -> `DimDate.DateKey` (inactive in TMDL).

`FactPolicySales` has one row per issuance or renewal transaction. `FactClaims` has one row per claim, storing current status, cumulative payments, and the remaining reserve; it is not a payment history. `AgentKey` identifies the policy's selling agent, not a claims adjuster.

`PolicyNumber` provides business traceability between the facts, not a direct fact-to-fact relationship. Multiple claims may reference one policy. Amounts are USD; incurred claims equal `PaidAmount + OutstandingReserve`, which need not equal the amount requested.

## Semantic Model

The native TMDL in [semantic-model/](semantic-model/) now exports the deployed `InsurancePolicySalesAndClaimsSchema`, replacing the earlier illustrative calculated-table model. Its seven Direct Lake entity partitions reference the matching lowercase `dbo` lakehouse tables through the SQL analytics endpoint.

The model's existing item ID, seven measures, and eleven relationships were preserved during the source change. All seven partitions were verified ready. Editing a local CSV does not update Fabric: load changed data into the lakehouse and refresh the model. See [semantic-model.md](semantic-model.md) for its inventory, measures, and date-filter behavior.

## Ontology

The deployed ontology copies the original six entities and five relationships and adds a `Claim` entity containing all 14 claims columns and six relationships. [ontology-definition.json](ontology-definition.json) is the native generation 2 Fabric definition, containing 21 InlineBase64 TMDL parts with destination lakehouse bindings. It is a definition payload, not a complete create-item request or credential package. [ontology-model.json](ontology-model.json) is its readable source-column and join manifest, not an import package. [ontology.ttl](ontology.ttl) remains a descriptive RDF extension using the original ontology namespace, not the destination's native definition.

| Claim relationship | Source column | Target |
| --- | --- | --- |
| `policyAgent` | `AgentKey` | `Agent.AgentKey` |
| `claimedBy` | `CustomerKey` | `Customer.CustomerKey` |
| `claimedProduct` | `ProductKey` | `Product.ProductKey` |
| `claimCarrier` | `CarrierKey` | `Carrier.CarrierKey` |
| `lossOn` | `LossDateKey` | `Date.DateKey` |
| `reportedOn` | `ReportedDateKey` | `Date.DateKey` |

Shared property names such as `PolicyNumber` and dimension keys are qualified for `Claim` in Turtle, preserving the existing `PolicySale` property domains. The two claim date relationships are ontology links; active/inactive date filtering is a TMDL behavior, not an RDF relationship setting. Claim amount properties use `xsd:decimal` for USD values.

The `dbo.factclaims` table and claim bindings are deployed in `sbfabricInsDataAgent`. Reusing these exports in another workspace requires remapping item IDs and source connections. The repository does not contain a configured or deployed data agent.

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
- [semantic-model.md](semantic-model.md): deployed Direct Lake model inventory and measure notes.
- [semantic-model/tables/README.md](semantic-model/tables/README.md): per-table TMDL guide.
- [ontology.ttl](ontology.ttl): original RDF/Turtle ontology extended locally with claims.
- [ontology-model.json](ontology-model.json): deployed entity source columns and relationship join mappings.
- [ontology-definition.json](ontology-definition.json): native deployed ontology definition with TMDL parts.
- [fabric-artifacts.json](fabric-artifacts.json): deployed Fabric identifiers, read-only reference provenance, and verification results.
