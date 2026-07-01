# Rule DSL v0.5

Represent dossier statements that are monetary, invoice-confrontable, or intentionally mapped as unsupported.

The agent can interpret the dossier and qualify context, but the final financial audit is deterministic.

## Official Billing Field

Use `c.chargetotalamount` as the official POC amount field. In API responses this appears as `chargeTotalAmount` or `charge_total_amount`.

Other invoice fields such as usage quantity, duration, volume, free events, daily charged days and `charge_base_rate` may be used only as inputs to deterministic calculations.

## Date Handling

When the dossier provides a validity period, copy it to `effectiveFrom` and `effectiveTo` inside `rule_draft_json`.

For every monetary rule that can become `confrontable_deterministic`, include `effectiveFrom` when the dossier gives a start date. Use `effectiveTo: null` when the dossier has no explicit end date. Do not keep the date only in tool calls or free-text rationale; it must be persisted in the rule draft.

When the dossier is unclear, prefer invoice-line `c_effectivedate` for audit matching and mark the assumption in `openQuestions` or `assumptions`. Use `period_start_date` and `period_end_date` when the rule explicitly talks about service period, prorata, or billed days. Do not invent a date window.

## Rule Shape

```json
{
    "ruleName": "Human-readable rule name",
    "ruleSituation": "executable | needs_review | not_applicable",
    "dependencyCodes": ["needs_crm", "needs_bundle_eligibility"],
    "situationRationale": "Resumo curto da situacao principal e das dependencias.",
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
    "effectiveFrom": "YYYY-MM-DD start date from dossier",
    "effectiveTo": "YYYY-MM-DD end date from dossier, or null when open-ended",
    "candidatePolicy": {
        "discoveryHints": ["product name", "product family", "variant names"],
        "candidateSourcePriority": ["chargecode_description", "bill_message_text", "productcatalog_description", "bundle_caption"],
        "mustResolveTo": "chargecode_key_or_line_identity",
        "allowHighRecall": true
    },
    "chargecodeCandidates": [
        {
            "chargecodeKey": "RMEXAMPLE001",
            "chargecodeDescription": "Descricao do chargecode",
            "billMessageText": "Texto de fatura quando existir",
            "decision": "include | exclude | pending",
            "lineRole": "direct_product_charge | plan_with_benefit | discount | different_variant | context_only | sva_ambiguous | chargecode_description_match | unknown",
            "sourcePriority": "chargecode_description | bill_message_text | productcatalog_description | bundle_caption | catalog_alias | inferred_chargecode",
            "rationale": "Racional curto em portugues brasileiro"
        }
    ],
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
    "externalConditions": {
        "crm": {
            "policy": "not_required | required_to_apply | required_to_disambiguate | optional_context",
            "crmProductIds": ["0055013624"],
            "crmOfferIds": ["0055013625"],
            "bundleCrmIds": ["BUNDLE-CRM-001"],
            "regions": ["SP"],
            "activationDatePolicy": "active_on_effective_date | active_on_period_start | relative_to_activation | not_required",
            "requiredChecks": ["crm_product_id", "activation_date"],
            "rationale": "Dados CRM qualificam elegibilidade/aplicacao, nao criam preco."
        },
        "bundleEligibility": {
            "policy": "not_required | required_to_apply | optional_context",
            "bundleNames": ["Vivo Total"],
            "requiredComponents": ["internet", "tv", "mobile"],
            "lostEligibilityWhenMissing": ["internet"],
            "requiredChecks": ["active_bundle", "bundle_components"],
            "rationale": "Bundle qualifica elegibilidade ou escopo da regra."
        }
    },
    "ruleSet": {
        "key": "product:spotify",
        "targetProductName": "Spotify",
        "targetAliases": ["Spotify", "Spotify Premium"],
        "targetChargecodes": ["RMSPOTIFYVM"]
    },
    "ruleRelationship": {
        "relationshipType": "default_for | supersedes | fallback_of | sibling_of | independent | requires_manual_review",
        "parentRuleId": null,
        "supersedesRuleIds": [],
        "priorityRank": 100,
        "conditionJson": {},
        "rationale": "Como esta regra convive com outras do mesmo produto/familia."
    },
    "disambiguation": {
        "missingDisambiguators": ["crm_product_id", "service_id", "activation_date", "region"],
        "requiredCrmChecks": ["crm_product_id", "service_id"],
        "rationale": "Explique quais dados externos faltam para confirmar a regra neste chargecode."
    },
    "requiredCrmChecks": ["crm_product_id", "service_id", "activation_date", "region"],
    "stacking": {
        "stackingPolicy": "exclusive | stackable | cancels_lower_priority | requires_manual_review",
        "competingRulePolicy": "highest_expected_amount_for_underbilling | explicit_priority | not_applicable",
        "rationale": "Explique como tratar concorrencia de regras no mesmo chargecode."
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

## Rule Situation and Dependencies

Use `ruleSituation` as the primary business-facing state:

- `executable`: the rule has monetary behavior, candidate predicate and enough structure for the deterministic engine, even if it carries caveats such as CRM or bundle checks.
- `needs_review`: the rule has monetary relevance, but missing mapping, ambiguous scope, unsupported calculation, missing usage/reference data, or priority uncertainty prevents deterministic execution without review.
- `not_applicable`: the dossier item is operational, informational or otherwise unrelated to billable invoice value.

Do not invent other statuses. Put technical reasons in `dependencyCodes`, `support.unsupportedReasons`, `disambiguation`, `externalConditions` and `requiredCrmChecks`.

`support.confrontabilityStatus` remains for compatibility with the deterministic engine and older UI fields. It must not be used as a free-form business status. A rule can be `ruleSituation: "executable"` and still carry `dependencyCodes` such as `needs_crm` or `needs_bundle_eligibility` when those checks explain eligibility, ambiguity or caveats rather than the existence of the price/formula.

CRM, CRM offer/product IDs, region, activation date and bundle eligibility do not create price or formula. They only decide applicability, disambiguation and audit caveats. The value, discount, free rule or usage formula must come from the dossier.

Every monetary rule must include:

- `ruleSet.key`, grouping rules by product, product family, bundle or plan.
- `ruleRelationship.relationshipType`, explaining whether this rule is independent, a default, a sibling, a superseder, a fallback, or requires review.
- Existing-rule context from `/agent-tools/rules/context` and conflict context from `/agent-tools/rules/conflicts` before finalizing `ruleRelationship`.

`highest_expected_amount_for_underbilling` is only a fallback for underbilling/recovery when two or more rules can govern the same billing context and CRM/taxonomy cannot disambiguate. Do not use it for customer-credit scenarios or as a universal priority rule.

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

The agent must start candidate discovery with product/family names against `chargecode_description` and `bill_message_text`. Then use `productcatalog_description`, bundle captions, catalog aliases and productcatalog keys to qualify the role of the line.

Preserve every considered candidate in `chargecodeCandidates` and in `candidate_sets_resumo`. Include the source priority, line role, matched fields, positive/negative signals, ambiguity reason and rationale.

For final monetary audit, the predicate must resolve to:

- `chargecodeKeyIn`; or
- `billingLineIdentityIn` entries that include `chargecodeKey`.

Do not return `descriptionContains`, `productcatalogKeyIn`, or `bundleOfferCaptionIn` as the only final monetary predicate. Keep them as candidates and mark the rule `confrontable_after_mapping`, `needs_mapping`, or `needs_agent_qualification`.

Never use expected price, billed amount, `netAmount`, `minAmount`, or `maxAmount` to select candidates.

If the same chargecode can have more than one monetary rule, do not discard the rule. Fill `disambiguation`, `requiredCrmChecks` and `stacking` so the deterministic audit can later explain whether it used explicit priority, highest expected amount for underbilling, or manual review.

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
