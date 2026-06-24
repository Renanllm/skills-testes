# Non-Confrontable Items

Do not create billing rules for operational statements unless they define expected invoice value or line inclusion/exclusion.

## Usually Non-Confrontable

- Activate, deactivate, or change IDs in PRM, CRM, billing setup, or backend systems.
- Internal workflow steps, approvals, migration checklists, deployment notes, or manual operations.
- Communication, eligibility copy, FAQ, sales guidance, or marketing explanation without expected charge behavior.
- Product objective, campaign narrative, or business rationale without billing condition.
- Pure catalog maintenance unless it changes what should be billed.
- Customer support instructions unless they define refund/credit or charge policy.

## Potentially Confrontable

Convert to rule only if the statement says or implies an invoice outcome:

- product must be free for a period;
- product must have fixed price;
- product must receive discount;
- product must not be charged in a bundle;
- old/new price applies after date or migration;
- charge applies only to eligible customers;
- one rule cancels or overrides another billing rule.

## Output

Return operational items under `nonConfrontableItems[]` with:

```json
{
  "title": "Deactivate Vivo Recado IDs in PRM",
  "reason": "Operational setup instruction; no expected invoice amount or billing predicate.",
  "evidence": [
    {
      "source": "dossier.pdf",
      "page": 3,
      "quote": "Short excerpt"
    }
  ]
}
```

If uncertain, mark as `needs_review` and explain what billing fact is missing.
