# Semantic Model Notes

`InsurancePolicySalesAndClaimsSchema` is exported as native TMDL under [semantic-model/](semantic-model/). Its Fabric item ID is `96bda898-d5ec-481f-bce7-dccd6c93b99c` in `sbfabricInsDataAgent`.

The model now uses seven Direct Lake entity partitions instead of embedded calculated DAX sample tables. The shared M expression in [semantic-model/expressions.tmdl](semantic-model/expressions.tmdl) connects to the SQL analytics endpoint of `InsurancePolicySalesAndClaimsLH`. Each partition references its matching lowercase `dbo` table. The model's item ID, seven measure formulas, and eleven relationships were preserved and verified after refresh. The original `sbfabiq` resources were not changed.

## Data Inventory

| Table | Verified model rows | Local CSV rows |
| --- | ---: | ---: |
| `DimAgent` | 10 | 10 |
| `DimCarrier` | 10 | 10 |
| `DimCustomer` | 10 | 10 |
| `DimProduct` | 10 | 10 |
| `DimDate` | 24 dates in 2026, on the 1st and 15th | 24 |
| `FactPolicySales` | 100 | 100 |
| `FactClaims` | 24 | 24 |

The exported customer table includes all seven CSV columns, including `Phone` and `Address`. The complete model has 52 source columns. TMDL stores source bindings and metadata, not a copy of the lakehouse data. Editing a CSV does not update Fabric; reload the affected lakehouse table and refresh the model.

On October 19, 2026, DAX verified USD 151,000 total premium, USD 79,700 paid claims, USD 358,200 outstanding reserve, and USD 437,900 incurred claims. All seven partitions were ready.

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

`FactClaims` has one row per claim, containing current status, cumulative paid amount, and outstanding reserve. Its 24 synthetic records are deployed in the destination lakehouse and read through Direct Lake. They reference the CSV seeds' 100 policies and 2026 dates (the 1st and 15th of each month).

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
