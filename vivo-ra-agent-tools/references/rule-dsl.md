# Rule DSL v0.3

Represent dossier statements that are monetary, invoice-confrontable, or intentionally mapped as unsupported.

The agent can interpret the dossier and qualify context, but the final financial audit is deterministic.

## Official Billing Field

Use `c.chargetotalamount` as the official POC amount field. In API responses this appears as `chargeTotalAmount` or `charge_total_amount`.

Other invoice fields such as usage quantity, duration, volume, free events, daily charged days and `charge_base_rate` may be used only as inputs to deterministic calculations.

## Date Handling

When the dossier provides a validity period, copy it to `effectiveFrom` and `effectiveTo`.

When the dossier is unclear, prefer invoice-line `c_effectivedate` for audit matching and mark the assumption in `openQuestions` or `assumptions`. Use `period_start_date` and `period_end_date` when the rule explicitly talks about service period, prorata, or billed days. Do not invent a date window.

## Rule Shape

```json
{
    "ruleName": "Human-readable rule name",
    "ruleType": "fixed_price | no_charge | discount | usage_tariff | prorata | bundle_composition | presence_rule | free_period | exclusion | migration_price | other_monetary | not_monetary",
    "dossierCode": "T3DMND0020131",
    "target": {
        "entityName": "Product, variant, bundle, plan, or catalog entity",
        "entityKind": "product | variant | bundle | plan | offer | unknown",
        "entityIds": ["catalog entity ids when known"],
        "affectedScope": "single_charge_line | product | bundle | plan | account | unknown"
    },
    "financialImpact": {
        "monetary": true,
        "impactKind": "customer_credit_risk | recovery_opportunity | validation_only",
        "amountField": "c.chargetotalamount"
    },
    "candidatePolicy": {
        "discoveryHints": ["product description", "bundle", "inferred chargecode"],
        "mustResolveTo": "chargecode_key_or_line_identity",
        "allowHighRecall": true
    },
    "predicate": {
        "chargecodeKeyIn": ["preferred final identifiers"],
        "billingLineIdentityIn": [
            {
                "bundleOfferCaption": "Optional bundle caption",
                "productcatalogDescription": "Optional exact product description",
                "chargecodeKey": "Required for deterministic monetary audit"
            }
        ],
        "productcatalogKeyIn": ["candidate only"],
        "bundleOfferCaptionIn": ["candidate only"],
        "descriptionContains": "recall only, not final by itself"
    },
    "calculation": {
        "kind": "fixed_amount | no_charge | discount_amount | discount_percent | usage_event_tariff | usage_duration_tariff | usage_volume_tariff | daily_rate | prorata_by_period | bundle_component_sum | forbidden_charge | presence_required | custom_logic",
        "amountField": "charge_total_amount",
        "expected": {
            "amount": 0,
            "referenceAmount": 29.9,
            "unitRate": 0.5,
            "discountPercent": 50
        },
        "requiredColumns": ["charge_total_amount"],
        "tolerance": 0.01
    },
    "support": {
        "confrontabilityStatus": "confrontable_deterministic | confrontable_after_mapping | needs_agent_qualification | needs_mapping | needs_usage_quantity | needs_reference_price | needs_crm | needs_subscription_event | needs_entitlement_inventory | needs_review | not_monetary | not_supported_yet | blocked",
        "unsupportedReasons": [],
        "requiredExternalData": []
    },
    "precedence": {
        "priority": 100,
        "stackingPolicy": "exclusive | stackable | cancels_lower_priority | requires_manual_review"
    },
    "evidence": [
        {
            "source": "dossier file name",
            "page": 1,
            "quote": "Short supporting excerpt",
            "rationale": "Why this supports the rule"
        }
    ],
    "confidence": 0.8
}
```

## Confrontability Statuses

- `confrontable_deterministic`: rule has evidence, calculation and final predicate with `chargecodeKeyIn` or `billingLineIdentityIn[].chargecodeKey`.
- `confrontable_after_mapping`: rule looks financially auditable, but current predicate is still a candidate such as product, bundle, or description.
- `needs_agent_qualification`: candidate sets are broad or ambiguous and need include/exclude/pending decisions.
- `needs_mapping`: rule is monetary but lacks billing identifiers.
- `needs_usage_quantity`: rule depends on quantity, duration, volume, events or daily charged days not available in the rule/base.
- `needs_reference_price`: rule depends on a base/reference price not extracted or not available through `charge_base_rate`.
- `needs_crm`: rule depends on eligibility, segment, customer status or another CRM attribute.
- `needs_subscription_event`: rule depends on activation, cancellation, migration, first charge, or another subscription event.
- `needs_entitlement_inventory`: rule depends on whether the customer should have the service.
- `needs_review`: evidence, date, precedence or expected value remains uncertain.
- `not_monetary`: operational, informational or technical item without direct invoice impact.
- `not_supported_yet`: theoretically auditable, but the current deterministic engine does not support the calculation.
- `blocked`: required tools failed.

Only `confrontable_deterministic` can generate financial impact.

## Calculation Kinds

- `fixed_amount`: billed amount must equal a fixed amount.
- `no_charge`: existing candidate line should be free or zero.
- `discount_amount`: billed amount should reflect an absolute discount from a reference amount.
- `discount_percent`: billed amount should reflect a percent discount from a reference amount.
- `usage_event_tariff`: billed amount derives from event count, free events and unit rate.
- `usage_duration_tariff`: billed amount derives from event duration, free duration and unit rate.
- `usage_volume_tariff`: billed amount derives from volume and unit rate.
- `daily_rate`: billed amount derives from charged days and daily rate.
- `prorata_by_period`: billed amount derives from period dates and reference price.
- `bundle_component_sum`: billed amount derives from grouped component lines.
- `forbidden_charge`: candidate line should not have positive charge.
- `presence_required`: validates a required presence/absence, usually needing external data.
- `custom_logic`: keep as not supported unless the deterministic engine explicitly supports it.

## Required Columns By Calculation

Use these names in `calculation.requiredColumns`:

- fixed/no-charge/forbidden: `charge_total_amount`
- event usage: `charge_total_amount`, `number_of_events`, optional `number_of_free_events`, `charge_base_rate`
- duration usage: `charge_total_amount`, `event_duration`, optional `free_duration`, `charge_base_rate`
- volume usage: `charge_total_amount`, `event_volume`, optional `volume_uom`, `charge_base_rate`
- daily rate: `charge_total_amount`, `daily_charge_days`, `charge_base_rate`
- prorata: `charge_total_amount`, `period_start_date`, `period_end_date`, and a reference price source
- discount: `charge_total_amount`, plus either `charge_base_rate` or `calculation.expected.referenceAmount`

If required columns or external data are missing, do not force a deterministic rule. Set the correct `support.confrontabilityStatus`.

## Candidate Predicate Rules

The agent can use product names, descriptions, bundles and productcatalog keys to find candidates.

For final monetary audit, the predicate must resolve to:

- `chargecodeKeyIn`; or
- `billingLineIdentityIn` entries that include `chargecodeKey`.

Do not return `descriptionContains`, `productcatalogKeyIn`, or `bundleOfferCaptionIn` as the only final monetary predicate. Keep them as candidates and mark the rule `confrontable_after_mapping`, `needs_mapping`, or `needs_agent_qualification`.

Never use expected price, billed amount, `netAmount`, `minAmount`, or `maxAmount` to select candidates.

## Financial Impact Kinds

- `customer_credit_risk`: expected amount is lower than billed, so Vivo may owe credit/refund.
- `recovery_opportunity`: expected amount is higher than billed, so Vivo may have underbilled.
- `validation_only`: expected equals billed, rule only validates, or the result is non-actionable without more data.

## Confidence

Use confidence as a communication aid, not as approval.

- `0.90-1.00`: direct dossier evidence plus strong billing predicate such as 1:1 chargecode.
- `0.75-0.89`: strong evidence, candidate predicate needs human confirmation.
- `0.50-0.74`: plausible but inferred from broad text, bundles, or samples.
- `<0.50`: keep as open question or non-confrontable unless tools improve evidence.

## Required Evidence

For every financial rule, include at least one short dossier evidence item. Keep excerpts short. Evidence should explain the billing expectation, not just mention the product.
