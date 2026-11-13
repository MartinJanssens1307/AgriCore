# ADR 004: VAT null handling and money precision

- **Status:** Accepted — interview-confirmed; compliance review outstanding
- **Decision owner:** AgriCore project owner
- **Tracking:** [Issue #8](https://github.com/MartinJanssens1307/AgriCore/issues/8)

## Context

A product's VAT rate may be unspecified, which is not the same as an explicit 0% rate. Farm transactions also need consistent calculations and rounding. [ADR 002](./002-transaction-lifecycle-and-status-rules.md) requires an explicit VAT treatment before a transaction is completed.

This ADR records the intended behavior, not a claim that the current application implements it or that the policy satisfies Belgian VAT or Peppol requirements.

## Decision

1. A product may have an unspecified VAT rate. An unspecified rate must remain distinguishable from an explicitly chosen 0% rate on a transaction line.
2. A draft line may be saved with VAT unspecified. For draft calculations only, use 0% as the provisional rate without replacing the unspecified value with 0%. Visibly mark the affected VAT and totals as provisional.
3. Before completion, each line must have an explicit VAT treatment. A provisional calculation with an unspecified rate must not satisfy this requirement. Details of VAT categories, exemptions, and supporting information remain open.
4. For now, quantities and unit prices use two decimal places. Perform monetary calculations with decimal arithmetic.
5. For each line, calculate quantity times net unit price and round the net amount to cents using half-up rounding. Calculate VAT using that **rounded line net** and round the VAT amount to cents using half-up rounding. The line gross is the sum of its rounded net and VAT amounts.
6. Transaction net, VAT, and gross totals are the sums of the corresponding rounded line amounts.

### Examples

- A draft line with net €100.00 and unspecified VAT calculates provisional VAT of €0.00 and provisional gross of €100.00. The rate remains unspecified and completion is blocked until an explicit treatment is supplied.
- Quantity 0.07 at €0.35 net per unit gives an unrounded net of €0.0245. Rounded half-up to cents, net is €0.02. At 21%, VAT on €0.02 rounds to €0.00, giving €0.02 gross.
- An unrounded amount of €0.005 rounds half-up to €0.01.

## Current implementation and gaps

- `Product.vat_rate` allows null, but `TransactionLineItem.vat_rate_percentage` is currently required. A missing line rate cannot currently be stored as an incomplete draft under the existing schema/form.
- The product lookup in `ERP/Views/transaction_views.py` returns `0.00` for an unspecified product rate. The transaction line template also defaults missing VAT to zero and populates the field, so the current workflow can lose the distinction between unspecified and explicit 0%.
- Line amounts are stored in decimal fields, but `TransactionLineItem.save()` does not explicitly apply the agreed per-step, half-up rounding policy. The line template uses JavaScript floating-point calculations for display.
- The validation method checks for line items and certain issuer/customer details but does not check for an explicitly specified VAT treatment or verify the proposed totals policy.
- Draft provisional indicators and the agreed completion gate are not present in the inspected code paths.

## Consequences

- Draft storage, product lookup, forms, and UI must preserve an unspecified rate while computing and displaying provisional values.
- Completion validation must reject an unspecified VAT treatment rather than interpreting it as an explicit 0% rate.
- Server-side line and transaction calculations, as well as UI previews, need consistent rounding and aggregation; the server must remain authoritative.
- Test coverage should include blank versus explicit 0%, provisional drafts, completion validation, half-cent rounding, multiple lines, and the rounded-net VAT base.

## Open questions

- Which VAT categories, exemptions, reverse-charge cases, and supporting details should AgriCore support?
- Does this rounding and aggregation policy meet the applicable Belgian accounting and Peppol requirements for the transactions AgriCore will support? Confirm with an accountant and the relevant specifications before relying on it for compliance.
- Is two-decimal precision sufficient for all farm quantities and unit prices, such as small quantities and price-per-unit calculations?
