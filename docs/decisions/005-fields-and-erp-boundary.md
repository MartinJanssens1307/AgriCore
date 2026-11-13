# ADR 005: Fields and ERP domain boundary

- **Status:** Accepted — interview-confirmed
- **Decision owner:** AgriCore project owner
- **Tracking:** [Issue #8](https://github.com/MartinJanssens1307/AgriCore/issues/8)

## Context

AgriCore currently has two Django apps relevant to the farm domain: `ERP`, which contains company and commercial workflows, and `Fields`, which contains cadastral parcels. The codebase is still small, but the product direction includes additional CRM, sales, crop-planning, and field-work capabilities.

The app boundary should reflect business-domain ownership without splitting the project prematurely or creating unnecessary coupling.

## Decision

1. **ERP remains the core/shared platform domain for now.** It owns the farm/company tenant and membership/access concepts and remains the home of existing shared business capabilities while the project is small.
2. **Fields owns field-domain data and workflows.** This includes cadastral parcels, field geodata (boundaries, access points and hazards), crop planning, field work, and other field-specific activities.
3. **A parcel belongs to a farm tenant.** A customer relationship, when needed, is an optional association such as a customer or contract for work on that tenant's parcel; it does not transfer tenant ownership or independently grant access.
4. **No cross-domain workflow is required today.** In the future, a field activity may reference an ERP product or service and may be linked to a transaction when that relationship is useful. Fields should not own product, customer, or financial-document rules.
5. **Do not split CRM and Sales into separate Django apps solely for naming or tidiness.** Revisit extracting CRM and Sales from the current ERP app when their independent models and workflows justify the additional boundaries.

## Alternatives considered

- **Move CRM and Sales immediately into separate apps:** deferred because the current codebase is small and premature extraction would add migration and integration work without a demonstrated need.
- **Keep all field operations in ERP:** rejected because cadastral and operational field workflows form a distinct domain from shared company and commercial capabilities.
- **Allow a customer relationship to determine parcel tenancy:** rejected because customer/contract association is not the same as tenant ownership or user access.

## Consequences

- Fields can grow into crop planning and field operations while remaining scoped to a farm tenant.
- ERP remains the shared tenant/company foundation, even if its existing CRM and sales responsibilities are later extracted.
- Future links between Fields and CRM/Sales should be explicit and avoid placing commercial or financial rules inside Fields.
- The current `Fields` dependency on `ERP.Company` is consistent with the decision; future app extraction should preserve clear ownership and avoid circular imports.

## Implementation status

- `Fields.CadastralParcel` currently references `ERP.Company`; parcel views scope reads and writes to the active tenant.
- Current cadastral parcel and map workflows exist.
- Crop planning, field work/activity models, customer/contract associations to parcels, and cross-domain product/service or transaction workflows are not implemented in the inspected code.
- CRM and sales workflows currently remain in `ERP`; no extraction is part of this ADR.

## Open questions

- Which field activities should be modeled first, and what history/audit details should each preserve?
- Should customer/contract associations attach to a parcel, an individual field activity, or both?
- When CRM and Sales grow, should they become two separate apps or one commercial-domain app?
