# ADR 002: Transaction lifecycle and status rules

- **Status:** Accepted — interview-confirmed
- **Decision owner:** AgriCore project owner
- **Tracking:** [Issue #8](https://github.com/MartinJanssens1307/AgriCore/issues/8)

## Context

AgriCore transactions represent financially meaningful documents for farm operations. The project needs explicit lifecycle rules so behavior is predictable, financially defensible, and still practical for a small farm workflow.

The current model provides four statuses (`new`, `progress`, `cancelled`, `completed`) and a freeze action (`validate_and_freeze`) that snapshots core data and assigns a public identifier. This ADR defines the intended lifecycle and correction policy and distinguishes it from current implementation details.

## Decision

1. **Status meanings**
   - `new`: a newly created transaction draft object.
   - `progress`: a working state while details are being completed.
   - `completed`: a validated and fixed financial transaction.
   - `cancelled`: a draft/work transaction that was abandoned before final financial completion.

2. **Allowed transitions (strict matrix)**
   - `new -> progress`
   - `new -> cancelled`
   - `progress -> completed`
   - `progress -> cancelled`
   - No transitions out of `completed`.
   - No transitions out of `cancelled`.

3. **Immutability and correction**
   - `completed` transactions are immutable.
   - Corrections for completed transactions must use compensating documents (credit/correction flow), not direct edits and not status rollback.

4. **Completion authority**
   - Completion/validation is allowed for Owner and Accountant roles.

5. **Completion prerequisites (baseline)**
   - At least one line item exists.
   - Issuer (tenant/company) exists with required invoicing identity fields (including VAT where required).
   - Customer exists.
   - Billing address exists.
   - VAT treatment is explicit on lines, including zero-rate cases where applicable.
   - Header totals are consistent with line-level calculations.

6. **Public identifier policy**
   - Public ID is assigned only when the transaction reaches `completed`.
   - Cancelled transactions that never reached completion do not receive a public ID.

## Current implementation and gaps

- The `Transaction` model already defines statuses: `new`, `progress`, `cancelled`, `completed`.
- `validate_and_freeze` sets status to `completed`, snapshots issuer/customer data, and generates `public_id`/`local_sequence` only when needed. Current sequence logic is per tenant and based on previously completed transactions.
- `transaction_delete` prevents deletion of `completed` transactions but allows deleting non-completed ones.
- `transaction_validate` currently checks business validation errors and then calls `validate_and_freeze`.
- Owner/Accountant completion authority is not currently enforced in the inspected transaction view path; no role gate was found around validation.
- The strict transition matrix is not fully enforced in the inspected view path (for example, there is no explicit status transition endpoint controlling all transitions).
- A dedicated compensating-document workflow for correcting completed transactions was not found in the inspected code.
- `cancelled` behavior is represented in statuses, but no explicit cancel action flow was found in the inspected transaction views.

## Consequences

- Financial history remains stable after completion, improving legal/accounting confidence.
- Corrections require explicit compensating flows, which adds implementation work but preserves traceability.
- Role-based authorization must be implemented on completion actions to align behavior with policy.
- Status transitions should be implemented through explicit, validated actions rather than ad hoc assignments.
- The UI should expose cancel and completion actions consistent with the strict transition matrix.

## Open questions

- Should Employee role remain strictly read-only for all transaction operations, or should limited draft creation be allowed later?
- What document model should be used for compensating transactions (separate type, linked transaction reference, numbering conventions)?
- Should cancelled transactions be soft-deactivated for audit visibility, or remain hard-deletable before completion?
