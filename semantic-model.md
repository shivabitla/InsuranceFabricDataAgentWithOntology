# Semantic Model Notes

`InsuranceSalesStarSchema` was exported as native TMDL under `semantic-model/`.

This is a separate illustrative model, not a live connection to `InsuranceDataLH`. Its exported `FactPolicySales` partition contains five illustrative records (`POL-10001` through `POL-10005`) and its `DimDate` partition covers 2025. The Lakehouse seed tables instead contain 50 policy transactions dated in 2026. Refresh or replace the model partitions before treating it as synchronized with the Lakehouse.

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

## Claims Fact

`FactClaims` is a local extension with one row per claim, containing current status, cumulative paid amount, and outstanding reserve. Its six synthetic records reuse the existing model's 2025 policy numbers and dimension keys. The separate `FactClaims.csv` contains 24 claims referencing the local CSV seeds' 100 policies and 2026 dates (the 1st and 15th of each month). Neither claims dataset is deployed to Fabric.

`PolicyNumber` provides business traceability, not a direct relationship between facts. `AgentKey` is the policy's selling agent. Amounts are USD; incurred claims equal `PaidAmount + OutstandingReserve`, not necessarily `ClaimedAmount`. No claim-payment history is retained by this grain. See [schema-diagram.md](schema-diagram.md) for the complete schema and date-filter semantics.

## Measures

- `Total Premium`
- `Total Commission`
- `Policy Transactions`
- `Average Premium`
- `Total Coverage`
- `Commission Rate`
- `New Policies`
