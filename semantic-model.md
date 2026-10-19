# Semantic Model Notes

`InsuranceSalesStarSchema` was exported as native TMDL under `semantic-model/`.

This is a separate illustrative model, not a live connection to `InsuranceDataLH` or the root CSVs. Its partitions are calculated DAX sample tables. Its five policy sales use `POL-10001` through `POL-10005` and 2025 dates. The original Lakehouse seeds contained 50 policy sales dated in 2026; the current local CSVs contain 100 sales and 24 claims dated in 2026. Replace the model partitions with the intended source and refresh before treating these datasets as synchronized.

## Data Inventory

| Table | TMDL rows | Local CSV rows |
| --- | ---: | ---: |
| `DimAgent` | 5 | 10 |
| `DimCarrier` | 5 | 10 |
| `DimCustomer` | 5 | 10 |
| `DimProduct` | 5 | 10 |
| `DimDate` | 365 daily dates in 2025 | 24 dates in 2026, on the 1st and 15th |
| `FactPolicySales` | 5 | 100 |
| `FactClaims` | 6 | 24 |

The exported TMDL customer table has `CustomerKey`, `CustomerID`, `CustomerName`, `State`, and `Region`. The CSV also includes `Phone` and `Address`; those two fields are not present in the current TMDL table.

## Relationships

- `FactPolicySales.AgentKey` -> `DimAgent.AgentKey`
- `FactPolicySales.CustomerKey` -> `DimCustomer.CustomerKey`
- `FactPolicySales.ProductKey` -> `DimProduct.ProductKey`
- `FactPolicySales.CarrierKey` -> `DimCarrier.CarrierKey`
- `FactPolicySales.SaleDateKey` -> `DimDate.DateKey`
- `FactClaims.AgentKey` -> `DimAgent.AgentKey`
- `FactClaims.CustomerKey` -> `DimCustomer.CustomerKey`
- `FactClaims.ProductKey` -> `DimProduct.ProductKey`
- `FactClaims.CarrierKey` -> `DimCarrier.CarrierKey`
- `FactClaims.LossDateKey` -> `DimDate.DateKey` (active)
- `FactClaims.ReportedDateKey` -> `DimDate.DateKey` (inactive)

All eleven relationships use many-to-one fact-to-dimension cardinality and single-direction dimension-to-fact filtering. Ten are active. There is no direct relationship between the two facts.

## Date Filtering

A shared date filter applies to sale dates for policy sales and loss dates for claims. For a report-date view of claims, use an explicit measure such as:

```dax
Claims by Reported Date =
CALCULATE(
	COUNTROWS(FactClaims),
	USERELATIONSHIP(FactClaims[ReportedDateKey], DimDate[DateKey])
)
```

This is an example, not a measure currently defined in the exported model. Comparing premium and claims under the same date filter compares sale-date premium against loss-date claims, not an earned-premium loss ratio.

## Claims Fact

`FactClaims` is a local extension with one row per claim, containing current status, cumulative paid amount, and outstanding reserve. Its six synthetic records reuse the existing model's 2025 policy numbers and dimension keys. The separate `FactClaims.csv` contains 24 claims referencing the local CSV seeds' 100 policies and 2026 dates (the 1st and 15th of each month). Neither claims dataset is deployed to Fabric.

`PolicyNumber` provides business traceability, not a direct relationship between facts. `AgentKey` is the policy's selling agent. Amounts are USD; incurred claims equal `PaidAmount + OutstandingReserve`, not necessarily `ClaimedAmount`. No claim-payment history is retained by this grain. See [schema-diagram.md](schema-diagram.md) for the complete schema and date-filter semantics.

## Measures

The seven existing measures are defined on `FactPolicySales`. There are no stored claims measures yet.

- `Total Premium`
- `Total Commission`
- `Policy Transactions`
- `Average Premium`
- `Total Coverage`
- `Commission Rate`
- `New Policies`
