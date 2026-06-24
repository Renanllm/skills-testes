# Output Contract

Return one JSON object only. Do not include prose outside the JSON.

The output must preserve both auditable rules and mapped-but-unsupported statements. Unsupported statements are useful for backlog and coverage, but they must not generate financial impact.

```json
{
    "status": "draft | needs_mapping | needs_agent_qualification | needs_review | blocked | completed",
    "dossier": {
        "fileName": "T3DMND0020131.pdf",
        "dossierCode": "T3DMND0020131",
        "title": "Optional title inferred from dossier"
    },
    "summary": {
        "financialRuleCount": 1,
        "mappedUnsupportedCount": 2,
        "nonMonetaryCount": 1,
        "candidateSetCount": 4,
        "finalPredicateCount": 1,
        "openQuestionCount": 1
    },
    "financialRules": [
        {
            "ruleDraft": {
                "ruleName": "",
                "ruleType": "fixed_price | no_charge | discount | usage_tariff | prorata | bundle_composition | presence_rule | free_period | exclusion | migration_price | other_monetary | not_monetary",
                "target": {
                    "entityName": "",
                    "entityKind": "product | variant | bundle | plan | offer | unknown",
                    "entityIds": [],
                    "affectedScope": "single_charge_line | product | bundle | plan | account | unknown"
                },
                "financialImpact": {
                    "monetary": true,
                    "impactKind": "customer_credit_risk | recovery_opportunity | validation_only",
                    "amountField": "c.chargetotalamount"
                },
                "predicate": {},
                "calculation": {
                    "kind": "fixed_amount | no_charge | discount_amount | discount_percent | usage_event_tariff | usage_duration_tariff | usage_volume_tariff | daily_rate | prorata_by_period | bundle_component_sum | forbidden_charge | presence_required | custom_logic",
                    "amountField": "charge_total_amount",
                    "expected": {},
                    "requiredColumns": ["charge_total_amount"],
                    "tolerance": 0.01
                },
                "support": {
                    "confrontabilityStatus": "confrontable_deterministic | confrontable_after_mapping | needs_agent_qualification | needs_mapping | needs_usage_quantity | needs_reference_price | needs_crm | needs_subscription_event | needs_entitlement_inventory | needs_review | not_monetary | not_supported_yet | blocked",
                    "unsupportedReasons": [],
                    "requiredExternalData": []
                },
                "evidence": []
            },
            "confrontabilityStatus": "confrontable_deterministic | confrontable_after_mapping | needs_agent_qualification | needs_mapping | needs_usage_quantity | needs_reference_price | needs_crm | needs_subscription_event | needs_entitlement_inventory | needs_review | not_monetary | not_supported_yet | blocked",
            "candidateSets": [],
            "candidateQualification": {
                "status": "agent_qualified_needs_human_confirmation",
                "included": [],
                "excluded": [],
                "pending": [],
                "finalPredicate": {},
                "rationale": ""
            },
            "qualificationValidation": {},
            "ruleValidation": {},
            "conflicts": [],
            "evidence": [],
            "assumptions": [],
            "openQuestions": []
        }
    ],
    "mappedUnsupportedItems": [
        {
            "title": "",
            "supportStatus": "needs_crm | needs_subscription_event | needs_entitlement_inventory | needs_usage_quantity | needs_reference_price | not_supported_yet | needs_mapping | not_monetary",
            "unsupportedReasons": [],
            "requiredExternalData": [],
            "possibleFutureRuleType": "",
            "target": {},
            "calculationHint": {},
            "reason": "",
            "evidence": []
        }
    ],
    "nonConfrontableItems": [
        {
            "title": "",
            "reason": "",
            "evidence": []
        }
    ],
    "toolTrace": [
        {
            "tool": "POST /agent-tools/billing/candidate-discovery",
            "purpose": "Find broad candidates for Vivo Recado",
            "resultSummary": "2 direct identifier matches; broad description candidate excluded"
        }
    ],
    "globalOpenQuestions": []
}
```

## Required `candidateSets` Context

Every candidate returned inside a financial rule must preserve the billing context the agent used to qualify it. Do not return only id, decision and rationale.

```json
{
    "candidateSetId": "cand-chargecode-RMEXAMPLE001",
    "decision": "include | exclude | pending",
    "rationale": "Short rationale.",
    "billingContext": {
        "candidateSetKind": "chargecode_key | billing_line_identity | semantic_description_match | bundle_neighbor",
        "predicate": {
            "chargecodeKeyIn": ["RMEXAMPLE001"]
        },
        "chargecodeKeys": ["RMEXAMPLE001"],
        "productcatalogKeys": ["1234567890"],
        "productcatalogDescriptions": ["Example Product"],
        "bundleOfferCaptions": ["EXAMPLE BUNDLE"],
        "billingLineIdentityIds": ["bli-..."],
        "sampleInvoiceLineIds": ["invoice-line-id"],
        "lineCount": 10,
        "invoiceCount": 8,
        "customerCount": 8,
        "netAmount": 239,
        "positiveSignals": ["chargecode_token_match", "product_description_match"],
        "negativeSignals": [],
        "matchedOn": {
            "chargecodeKey": true,
            "productcatalogDescription": true,
            "bundleOfferCaption": false
        },
        "recommendedDecision": "include",
        "sourceTools": [
            "POST /agent-tools/billing/candidate-discovery",
            "POST /agent-tools/billing/candidate-clusters",
            "POST /agent-tools/invoices/sample-lines"
        ],
        "notes": "Billing context observed by the agent."
    }
}
```

Fields not available from tools may be empty arrays or `null`, but `billingContext` must exist for every candidate. Do not create candidates from expected amount or price windows.

## Required Final Checks

Before returning:

1. Every financial rule must have evidence.
2. Every financial rule must have `financialImpact.monetary: true`.
3. Every financial rule must have `calculation.kind`, `calculation.requiredColumns` and `support.confrontabilityStatus`.
4. Every `confrontable_deterministic` rule must have an executable predicate.
5. Every monetary final predicate must include `chargecodeKeyIn` or `billingLineIdentityIn[].chargecodeKey`.
6. Every broad candidate must be included, excluded, or pending.
7. Every unsupported rule-like statement must be returned under `mappedUnsupportedItems[]` or as a financial rule with non-deterministic `support.confrontabilityStatus`.
8. Every tool failure must be reflected in `status: "blocked"` or `globalOpenQuestions`.
9. Every returned candidate has structured billing context with available product descriptions, product keys, bundle captions and charge codes.
