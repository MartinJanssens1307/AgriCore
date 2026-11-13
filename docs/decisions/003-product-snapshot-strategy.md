# ADR 003: Product snapshot strategy in transactions

- **Status:** Accepted — interview-confirmed
- **Decision owner:** AgriCore project owner
- **Tracking:** [Issue #8](https://github.com/MartinJanssens1307/AgriCore/issues/8)

## Context

AgriCore transactions must remain historically accurate after completion, even when product master data changes later. The current implementation already stores line-level financial values in `TransactionLineItem` and includes a product name snapshot field.

This ADR defines what is snapshotted, when it is frozen, and how product lifecycle should behave.

## Decision

1. **Snapshot freeze timing**
   - Product snapshot freeze happens on transaction completion only.

2. **Snapshot scope**
   - For now, snapshot only `product.name` into the line snapshot field.
   - Other commercial values needed for calculation are already persisted as dedicated line fields (`quantity`, `unit_price_net`, `vat_rate_percentage`, totals), so they are not duplicated as extra snapshot fields at this stage.

3. **Post-completion immutability**
   - Completed transactions must never be retroactively changed by product master-data updates.

4. **Before-completion flexibility**
   - While transaction status is not completed, line-level product/price/VAT data is fully editable according to existing draft workflow.

5. **Product lifecycle**
   - Products must never be hard-deleted, whether or not they are referenced by a transaction.
   - Products should instead be archived or inactivated.

## Current implementation and gaps

- `TransactionLineItem` includes `product_name_snap`, and `Transaction.validate_and_freeze()` sets it from the selected product when completing a transaction.
- `TransactionLineItem` also calculates and stores line net/VAT/gross totals on save.
- `TransactionLineItem.product` uses `on_delete=PROTECT`, which blocks deleting products referenced by any line item.
- The current delete view still permits deletion of unreferenced products, contrary to the intended no-hard-delete policy.
- The product delete view currently calls `product.delete()` directly and does not handle `ProtectedError`; failed deletions for referenced products are likely to surface as server errors rather than user-friendly feedback.
- No explicit product archive/inactive flag or archive workflow was found in the inspected product model/view path.

## Consequences

- Historical invoice readability is preserved by storing the product label at completion.
- Financial amounts remain stable because they are saved on line items and not dynamically recomputed from current product catalog values.
- Product cleanup needs an archive/inactivation workflow to avoid reliance on hard-delete paths.
- Product management should offer archiving instead of hard deletion and clearly communicate the product's status.

## Open questions

- When should AgriCore add additional snapshot fields (unit, type, internal reference) if business/legal needs expand?
- Should archived products remain selectable for draft edits in special cases, or always hidden from new drafts?
- Should product-list filtering distinguish active vs archived products once archive support is added?
