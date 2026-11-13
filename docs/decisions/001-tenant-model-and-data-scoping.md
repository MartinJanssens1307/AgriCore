# ADR 001: Tenant model and data scoping

- **Status:** Accepted — interview-confirmed
- **Decision owner:** AgriCore project owner
- **Tracking:** [Issue #8](https://github.com/MartinJanssens1307/AgriCore/issues/8)

## Context

AgriCore needs a clear boundary for farm data and a predictable way to grant users access to that data. The current code uses `Company` as the company/farm record and associates user accounts with companies through `CompanyAccess`. ERP business-partner records are separate from user access.

This ADR captures the intended product rules from the tenant-model interview. It does not claim that every rule is implemented.

## Decision

1. **A company/farm is a tenant.** Each tenant's operational data is isolated from other tenants. A user may have access to more than one tenant.
2. **Membership requires an AgriCore account and explicit access.** The creator of a farm company becomes its owner. Another person may join only after being invited and accepting the invitation.
3. **Business partners are not tenant members by default.** A farm may store a customer, supplier, contact, or other business partner who has no AgriCore account and may not know about AgriCore. A business-partner record alone grants no login or tenant access.
4. **The owner is the only member intended to write company data.** Other members are intended to have read-only access to that company's data. This is a target policy; role-specific permissions are not implemented yet.
5. **Users may work with multiple companies.** The application should remember each user's last-used company across logins and devices. A company should become active immediately after that user creates it. A user may change the active company through configuration.
6. **Access must be checked against current membership.** Revoking a membership must immediately remove access to that tenant, including when it was previously active. The application must not silently switch to a different tenant after revocation; no tenant should remain active until the user explicitly selects an accessible company.
7. **No eligible company means no active tenant.** The application should show an appropriate company-selection or onboarding state, rather than displaying another company's data.

## Current implementation and gaps

- `Company` is the tenant-like entity. `CompanyAccess` associates users with companies and has Owner, Employee, and Accountant role values.
- The company-creation view creates an Owner `CompanyAccess` link for the current user.
- The tenant middleware protects `/app/` routes, requires authentication, and sets `request.tenant` from the active company.
- `get_current_company` stores `current_company_id` in the browser session and checks that the user still has access before using it. If that membership is gone, it clears the session value but automatically falls back to the first company membership it finds.
- That fallback does not implement the agreed no-silent-switch behavior. The helper also has no explicit ordering for its fallback membership.
- There is no per-user, cross-session/device last-used company preference in the inspected code. A company created by a user is not explicitly made active by the creation view.
- `set_current_company` validates membership and stores the selection in the session, but no application route or view was found calling it.
- Company-scoped query patterns are present for customers, products, transactions, addresses, and cadastral parcels. The tenant boundary still depends on each relevant query/view applying the correct scope.
- No invitation/acceptance flow or role-specific write/read enforcement was found in the inspected application paths.

## Consequences

- Tenant membership and business-partner relationships must remain distinct in both the domain model and documentation.
- Company selection needs a persistent per-user preference, an explicit post-creation selection, and a user-initiated switch.
- Membership revocation must invalidate the active tenant without quietly moving the user into another one.
- Authorization checks must enforce the owner-write/member-read-only policy; storing role labels alone is insufficient.
- Invitation creation, acceptance, and revocation need explicit behavior before those workflows can be documented as implemented.
- Views and data access paths need consistent tenant scoping, including related objects.

## Open questions

- What exact capabilities, if any, should distinguish Employee from Accountant while both are read-only?
- What should invitation expiration, cancellation, re-invitation, and acceptance look like?
- What user-facing screen should appear after access is revoked when the user still belongs to another company?
- Which data areas, if any, should be readable by only certain non-owner roles?
