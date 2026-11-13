# ADR 006: Peppol scope and prioritization

- **Status:** Accepted — interview-confirmed
- **Decision owner:** AgriCore project owner
- **Tracking:** [Issue #8](https://github.com/MartinJanssens1307/AgriCore/issues/8)

## Context

AgriCore contains an exploratory Ibanity-backed Peppol customer lookup. A complete Peppol workflow could be important to the product, but the APIs and their usage costs are not justified at the project's current stage.

## Decision

1. **Strategic importance:** Peppol remains an important potential product capability.
2. **Priority:** Peppol is the lowest-priority feature for now. API and usage costs are too high for the current project stage.
3. **Revisit trigger:** Reconsider Peppol investment when the product has reached a meaningful level of profitability or user adoption. No numeric threshold is defined yet.
4. **Current scope:** Existing customer-directory lookup code is exploratory. It is not a commitment to implement or support invoice transmission.
5. **Production readiness:** The current integration must not be relied on in production.
6. **Result semantics:** A confirmed customer-not-found result must be distinct from an unknown result caused by network, authentication, configuration, or provider failure.

## Alternatives considered

- **Prioritize complete Peppol invoicing now:** deferred because API and usage costs are not justified at the current stage.
- **Remove all Peppol code immediately:** not selected; the prototype can remain as exploratory work, provided it is not presented as production-ready.
- **Promise invoice sending based on the current lookup prototype:** rejected; discovery capability is not a complete invoice exchange workflow.

## Consequences

- Peppol work should not displace higher-priority foundational farm workflows at the current stage.
- Cost, operational readiness, and required compliance should be reassessed when adoption or profitability justifies revisiting the integration.
- The prototype's user-facing status must not conflate provider failure with a confirmed absence on the network.
- Any production rollout or invoice-sending capability requires a separate readiness and compliance decision.

## Implementation status

- `ERP/services_peppol.py` implements an Ibanity OAuth client-certificate token request and a Peppol customer-search request.
- `Customer` stores `peppol_status` and `last_peppol_check`; a customer-detail action triggers the lookup.
- No Peppol invoice-send or invoice-export workflow was found in the inspected code.
- The customer view currently maps every unsuccessful lookup response to the `IN` (“Not Found”) state and updates `last_peppol_check`. This does not distinguish a confirmed negative result from an API or configuration failure and should not be treated as reliable production status.
- The integration depends on Ibanity credentials and a client certificate supplied through environment variables. No production-readiness guarantee is made by this prototype.

## Open questions

- What measurable user-adoption or profitability threshold should trigger a formal Peppol cost and feasibility review?
- Which Ibanity product/API plan and usage model would be viable at that point?
- What compliance, operational monitoring, retry, and support requirements must be met before production use?
- Should recipient discovery remain a standalone capability, or be introduced only alongside end-to-end invoice exchange?
