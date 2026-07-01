# Endpoint Contracts

Base URL:

```txt
https://server-production-4285.up.railway.app
```

Auth:

```http
Authorization: Bearer {{AGENT_TOOLS_TOKEN}}
Content-Type: application/json
```

`GET /agent-tools/health` is public. Every other `/agent-tools/*` endpoint requires the bearer token. Never include the token in outputs.

## Common Predicate

Use this shape for invoice-line predicates:

```json
{
    "invoiceKeys": ["3332859950_358710428"],
    "chargecodeKeyIn": ["RMVIVORECADM"],
    "billingLineIdentityIn": [
        {
            "bundleOfferCaption": "Servico Digital 9 TBF",
            "productcatalogDescription": "Vivo Recado",
            "chargecodeKey": "RMVIVORECADM"
        }
    ],
    "productcatalogKeyIn": ["1125259208"],
    "bundleOfferCaptionIn": ["Servico Digital 9 TBF"],
    "descriptionContains": "Vivo Recado"
}
```

For final monetary rules, prefer `chargecodeKeyIn` or `billingLineIdentityIn` with a `chargecodeKey`. Use `descriptionContains`, `productcatalogKeyIn` and `bundleOfferCaptionIn` only for recall, never as the only final predicate unless the rule is explicitly marked `needs_mapping`, `confrontable_after_mapping` or `needs_agent_qualification`.

Do not use `minAmount`, `maxAmount`, expected price or billed amount to discover candidates.

## Health

```http
GET /agent-tools/health
```

Returns service status and version.

## Rule DSL Contract

```http
GET /agent-tools/rule-dsl/contract
```

Use before creating a rule. Returns DSL purpose, official amount field, canonical shape, and workflow guidance.

## Catalog Search

```http
POST /agent-tools/catalog/search
```

Body:

```json
{
    "query": "Vivo Recado",
    "entityKinds": ["product", "variant", "bundle", "plan", "offer"],
    "needsReview": false,
    "limit": 20,
    "offset": 0
}
```

Returns `candidates[]` with:

- `entityId`
- `entityKind`
- `canonicalName`
- `displayName`
- `category`
- `entitySubtype`
- `confidence`
- `needsReview`
- `aliases[]`
- `billingIdentifiers[]`
- `invoiceStats.lineCount`
- `invoiceStats.invoiceCount`
- `invoiceStats.netAmount`

Use this to understand the commercial catalog, not to approve billing mappings by itself.

## Catalog Entity Detail

```http
GET /agent-tools/catalog/entities/:entityId
```

Returns one entity with aliases, variants, relationships, source data, and invoice stats.

## CRM Mock Contract Search

```http
POST /agent-tools/crm/contracts/search
```

Body:

```json
{
    "customerKeys": ["100001272"],
    "subscriberKeys": ["58233393"],
    "financialAccountKeys": ["1127295986"],
    "contractIds": ["mock-spotify-100001272"],
    "crmProductIds": ["0055013624"],
    "crmOfferIds": ["0055013625"],
    "productName": "Spotify",
    "productVariant": "Individual",
    "activeOn": "2026-04-21",
    "statuses": ["active"],
    "includeInactive": false,
    "limit": 20,
    "offset": 0
}
```

Returns:

- `contracts[]`
- `total`

Each contract includes:

- `customerKey`
- `subscriberKey`
- `financialAccountKey`
- `crmSystem`
- `contractId`
- `crmProductId`
- `crmOfferId`
- `productName`
- `productVariant`
- `offerName`
- `offerInstanceId`
- `activationDate`
- `validFrom`
- `validTo`
- `status`
- `region`
- `bundleName`
- `bundleCrmId`
- `bundleOfferCaption`
- `bundleStatus`
- `metadata`

Use esta tool quando a regra depender de elegibilidade por cliente, produto/oferta CRM, bundle CRM, vigencia de contrato, ativacao, praca/regiao, janela relativa a contratacao, ou quando candidatos de billing precisarem ser desambiguados por contexto de cliente. `crmProductIds`, `crmOfferIds` e `bundleCrmIds` sao filtros opcionais, nao campos obrigatorios de toda regra. O mock de CRM e incremental: se a tool nao retornar dados suficientes, nao invente contratos ou IDs; registre a lacuna em `required_crm_checks` e explique que a auditoria depende de CRM.

## Billing Identifier Search

```http
POST /agent-tools/billing/identifier-search
```

Body:

```json
{
    "query": "Vivo Recado",
    "identifierKinds": ["chargecode_key", "productcatalog_key", "bundle_offer_caption"],
    "onlyChargecodeProduct1to1": true,
    "limit": 20,
    "offset": 0
}
```

Returns `identifiers[]` with:

- `identifierKind`
- `identifierValue`
- `normalizedDescription`
- `chargecodeDescription`
- `billMessageText`
- `descriptionMatchTokens[]`
- `confidenceHint`
- `isChargecodeProduct1to1`
- `productcatalogKeys[]`
- `bundleCaptions[]`
- `invoiceLineCount`
- `invoiceCount`
- `customerCount`
- `amountBreakdown`

Use this to inspect whether a `chargecode_key` is stable. A 1:1 chargecode-product relationship is a strong signal, not automatic business approval.

## Billing Line Identity Search

```http
POST /agent-tools/billing/line-identity-search
```

Body:

```json
{
    "query": "Vivo Recado",
    "productcatalogKeys": ["1125259208"],
    "bundleOfferCaptions": ["Servico Digital 9 TBF"],
    "chargecodeKeys": ["RMVIVORECADM"],
    "limit": 20,
    "offset": 0
}
```

Returns `identities[]` with composed invoice-line identity:

- `billingLineIdentityId`
- `productcatalogKey`
- `productcatalogDescription`
- `bundleOfferCaption`
- `chargecodeKey`
- counts and amounts

Use this when the candidate is only understandable through bundle + product description + charge code context. Do not return `billingLineIdentityId` as the final predicate unless the server contract later adds direct support for it; translate to supported predicate fields.

## Candidate Discovery

```http
POST /agent-tools/billing/candidate-discovery
```

Body:

```json
{
    "targetName": "Vivo Recado",
    "targetAliases": ["Vivo Recado Premium", "Recado Vivo"],
    "entityKinds": ["product"],
    "identifiers": ["RMVIVORECADM", "RMVIVORECADVT"],
    "strategy": "high_recall",
    "limit": 20
}
```

Returns:

- `candidateSets[]`
- `candidateBuckets.directIdentifierMatches[]`
- `candidateBuckets.bundleNeighborCandidates[]`
- `candidateBuckets.lineIdentityCandidates[]`
- `candidateBuckets.semanticDescriptionCandidates[]`
- `targetAliases[]` and `derivedTargetAliases[]`
- catalog and billing candidates
- `recommendedNextToolCalls[]`
- warnings and guidance

Use this after drafting the target. It intentionally favors recall and can include false positives. Candidate discovery prioritizes `chargecode_description` and `bill_message_text`, then uses product description, bundle context and inferred chargecode keys as qualification context; it must not use expected price, billed amount, `netAmount`, or amount windows as discovery criteria.

When building the final candidate context, copy all available candidate fields such as `candidateSetKind`, `predicate`, `chargecodeDescription`, `billMessageText`, `lineCount`, `invoiceCount`, `netAmount`, `matchedOn`, `matchedAliases`, `matchedAliasSources`, `candidateApplicationFilters`, `candidateSourcePriority`, `lineRoleSuggestion`, `positiveSignals`, `negativeSignals`, `recommendedDecision`, `ambiguityReason` and `risk`.

## Product Family Candidates

```http
POST /agent-tools/billing/product-family-candidates
```

Body:

```json
{
    "targetNames": ["Disney+", "Disney+ Padrão", "Disney+ Standard"],
    "targetAliases": ["Disney Plus", "Disney Padrão"],
    "excludedVariants": ["Anúncios"],
    "includeBundleContext": true,
    "limit": 30
}
```

Use esta tool como guardrail de recall sempre que a regra afetar uma familia/plano de produto, uma reprecificacao, gratuidade ampla, ou quando o dossie disser "todos os canais", "todos os IDs", "todos os fluxos" ou equivalente.

Returns `candidates[]` with:

- `candidateSetId`
- `candidateSetKind`
- `chargecodeKey`
- `chargecodeDescription`
- `billMessageText`
- `predicate`
- `productcatalogDescriptions[]`
- `bundleOfferCaptions[]`
- `productcatalogKeys[]`
- `sampleInvoiceLineIds[]`
- `lineCount`
- `invoiceCount`
- `customerCount`
- `positiveAmount`
- `negativeAmount`
- `netAmount`
- `candidateSourcePriority`: `chargecode_description | bill_message_text | productcatalog_description | bundle_caption | catalog_alias | inferred_chargecode`
- `lineRoleSuggestion`: `direct_product_charge | discount | different_variant | plan_with_benefit | context_only | sva_ambiguous | chargecode_description_match | unknown`
- `suggestedDecision`: `include | pending | exclude`
- `matchedAliases[]`
- `matchedAliasSources`
- `candidateApplicationFilters`
- `positiveSignals[]`
- `negativeSignals[]`
- `rationale`
- `billingContext`

Regra de decisao:

- Inclua candidatos `direct_product_charge` quando a descricao da linha representa diretamente o produto/plano afetado, mesmo que exista `bundleOfferCaption`.
- Exclua `discount`, `different_variant` e `plan_with_benefit` do predicado final, mantendo-os como contexto ou evidencia de exclusao.
- Mantenha `context_only` como `pending`, salvo evidencia explicita de que a regra atinge aquela linha.
- Se `candidateApplicationFilters.bundleEligibilityStatus = "pending_bundle_eligibility"`, preserve o candidato como considerado/pendente e deixe a elegibilidade para o resolver CRM/bundle.
- Nao use `positiveAmount`, `negativeAmount`, `netAmount` ou preco esperado para descobrir candidatos. Valores monetarios servem para contexto e auditoria posterior.

## Candidate Clusters

```http
POST /agent-tools/billing/candidate-clusters
```

Body:

```json
{
    "predicate": {
        "chargecodeKeyIn": ["RMVIVORECADM", "RMVIVORECADVT"]
    },
    "groupBy": ["chargecode_key", "productcatalog_key", "bundle_offer_caption"],
    "limit": 30
}
```

Returns grouped counts and amounts. Use this to see whether a candidate is a focused product line, a bundle neighbor, or an ambiguous mapping.

Use clusters to fill candidate `billingContext` fields: `productcatalogKeys`, `productcatalogDescriptions`, `bundleOfferCaptions`, `chargecodeKeys`, `lineCount`, `invoiceCount`, `customerCount` and `netAmount`.

## Qualification Validate

```http
POST /agent-tools/billing/qualification-validate
```

Body:

```json
{
    "finalPredicate": {
        "chargecodeKeyIn": ["RMVIVORECADM", "RMVIVORECADVT"]
    },
    "candidateQualification": {
        "includeCandidateSetIds": ["cand-chargecode-RMVIVORECADM"],
        "excludeCandidateSetIds": ["cand-description-vivo-recado"],
        "rationale": "Direct charge codes match target; broad description is over-inclusive."
    }
}
```

Returns:

- `isExecutable`
- `warnings[]`
- `coverage.lineCount`
- `coverage.invoiceCount`
- `coverage.customerCount`
- `coverage.positiveAmount`
- `coverage.netAmount`
- echoed `finalPredicate` and qualification

Use this before returning a final predicate.

## Existing Rules

```http
POST /agent-tools/rules/existing
```

Body:

```json
{
    "dossierCode": "T3DMND0020131",
    "targetEntityIds": ["catalog-entity-id"],
    "status": ["active", "approved", "draft"],
    "limit": 50,
    "offset": 0
}
```

Use this before closing `ruleSet` and `ruleRelationship`. Query by dossier, target entity ids and status when available; if the final predicate uses chargecodes that are not yet mapped to entity ids, still call this endpoint with the known target context and then call `/agent-tools/rules/conflicts` with the predicate.

Existing rules do not automatically block a new rule. They are context for priority, sibling/default/supersedes relationships, CRM caveats and manual review.

## Rule Context

```http
POST /agent-tools/rules/context
```

Body:

```json
{
    "targetName": "Spotify Nordeste",
    "targetAliases": ["Spotify", "Spotify Premium"],
    "chargecodeKeys": ["RMSPOTIFYVM"],
    "crmProductIds": ["CRM-SPOTIFY-NORDESTE"],
    "crmOfferIds": [],
    "bundleNames": [],
    "effectiveFrom": "2025-08-18",
    "effectiveTo": null,
    "proposedRelationship": {
        "ruleId": "br-new-rule-if-known",
        "parentRuleId": "br-existing-default-rule",
        "relationshipType": "sibling_of",
        "priorityRank": 80,
        "conditionJson": {
            "crmProductIds": ["CRM-SPOTIFY-NORDESTE"]
        }
    }
}
```

Returns:

- `ruleSetKey`
- `rules[]` with current rules in the same rule set, expected values, final predicates, chargecodes, CRM/bundle hints and `relationship`
- `relationships[]` with parent/child, `relationshipType`, `priorityRank`, condition and rationale
- `conflicts[]` for chargecode overlaps, whether coexistence is allowed and which disambiguators are missing
- `suggestedPlacement` with recommended `relationshipType`, `priorityRankSuggestion`, options and rationale
- `validation.isValid` and `validation.errors[]`, including `cycle_detected`
- `gaps[]`

Use this after candidate qualification and before finalizing `ruleSet`, `ruleRelationship` and `stacking`. Prefer this endpoint over `/rules/existing` when the agent needs precedence context because it expands from matched rules to the whole rule set and validates obvious cycles. Same-chargecode rules can coexist when CRM, bundle, region, effective window or another explicit condition disambiguates them.

## Rule Applicability Preview

```http
POST /agent-tools/rules/applicability-preview
```

Body:

```json
{
    "billingRuleId": "br-spotify-individual",
    "ruleSetKey": "product:spotify",
    "finalPredicateId": "rfp-spotify-individual",
    "includeCrmContext": true,
    "limit": 20
}
```

Use this after `/rules/context` when a rule set has CRM, bundle or competing-rule ambiguity. The preview does not calculate final money. It selects invoice candidate lines from the rule predicate, respects the rule `auditUnit` (`single_charge_line` or `monthly_charge_group`), consults the CRM mock when requested, and returns the rule that currently applies to each billing unit.

Returns:

- `auditUnit`
- `rules[]` ordered by priority
- `applicabilityUnits[]` with invoice line ids, customer/subscriber context, `finalResolvedRuleId`, `crmMatchStatus`, `bundleEligibilityStatus`, `eligibilityStatus`, `applicationBasis`, `rulePath`, and optional `crmContracts`
- `summary.evaluatedUnitCount`
- `summary.eligibleUnitCount`
- `summary.unknownEligibilityUnitCount`
- `summary.crmMatchedUnitCount`
- `summary.bundleMatchedUnitCount`

Interpretation:

- `applicationBasis: "crm_confirmed"` or `"bundle_confirmed"` means the mock CRM/bundle data confirms which rule wins.
- `applicationBasis: "needs_crm"` means the billing line exists, but the rule requires CRM evidence that is missing.
- `applicationBasis: "needs_bundle_eligibility"` means the line is a candidate, but active bundle eligibility was not proven.
- `applicationBasis: "billing_only"` means no CRM/bundle condition was required for that rule.

## Rule Validate

```http
POST /agent-tools/rules/validate
```

Body:

```json
{
    "ruleDraft": {
        "ruleName": "Vivo Recado - no charge",
        "ruleSituation": "executable",
        "dependencyCodes": [],
        "situationRationale": "Regra monetaria com predicado final por chargecode.",
        "ruleType": "free_period",
        "target": {
            "entityName": "Vivo Recado",
            "entityKind": "product",
            "affectedScope": "single_charge_line"
        },
        "financialImpact": {
            "monetary": true,
            "impactKind": "customer_credit_risk",
            "amountField": "c.chargetotalamount"
        },
        "effectiveFrom": "2025-07-01",
        "effectiveTo": null,
        "predicate": {
            "chargecodeKeyIn": ["RMVIVORECADM"]
        },
        "calculation": {
            "kind": "no_charge",
            "amountField": "charge_total_amount",
            "expected": {
                "amount": 0
            },
            "requiredColumns": ["charge_total_amount"],
            "tolerance": 0.01
        },
        "support": {
            "confrontabilityStatus": "confrontable_deterministic",
            "unsupportedReasons": [],
            "requiredExternalData": []
        },
        "externalConditions": {
            "crm": {
                "policy": "not_required"
            },
            "bundleEligibility": {
                "policy": "not_required"
            }
        },
        "ruleSet": {
            "key": "product:vivo-recado",
            "targetProductName": "Vivo Recado",
            "targetChargecodes": ["RMVIVORECADM"]
        },
        "ruleRelationship": {
            "relationshipType": "independent",
            "priorityRank": 100,
            "rationale": "Sem regra concorrente identificada nas tools."
        },
        "evidence": []
    }
}
```

Returns `valid`, `errors[]`, `warnings[]`, `confrontabilityStatus`, `canAuditDeterministically`, `requiredColumns`, and `normalizedRule`.

## Rule Conflicts

```http
POST /agent-tools/rules/conflicts
```

Body:

```json
{
    "targetEntityIds": ["catalog-entity-id"],
    "predicate": {
        "chargecodeKeyIn": ["RMVIVORECADM"]
    },
    "effectiveFrom": "2026-01-01",
    "effectiveTo": null
}
```

Use this after rule validation. Conflicts do not always block the rule, but they require precedence reasoning or human review. If conflict priority cannot be resolved from dossier, CRM mock or existing-rule metadata, set `ruleRelationship.relationshipType: "requires_manual_review"` and explain the unresolved condition.

## Audit Preview

```http
POST /agent-tools/audit/preview
```

Body:

```json
{
    "ruleDraft": {},
    "predicate": {
        "chargecodeKeyIn": ["RMVIVORECADM"]
    },
    "limit": 20
}
```

Returns a deterministic preview with:

- `isExecutable`
- `blocked`
- `blockReason`
- `warnings[]`
- `expected`
- `summary.auditedLines`
- `summary.actualAmount`
- `summary.expectedAmount`
- `summary.amountDelta`
- `summary.divergentLines`
- `summary.recoverableRevenue`
- `summary.customerCreditRisk`
- `summary.financialImpactAmount`
- `previewLines[]`

For monetary rules, final audit requires `chargecodeKeyIn` or `billingLineIdentityIn[]` with `chargecodeKey`. Semantic predicates such as `descriptionContains` are blocked here and should stay in candidate discovery/review.

Use preview to inspect deterministic behavior before running the internal audit job. Do not ask the agent to audit all invoice lines manually.

## Invoice Sample Lines

```http
POST /agent-tools/invoices/sample-lines
```

Body:

```json
{
    "predicate": {
        "chargecodeKeyIn": ["RMVIVORECADM"]
    },
    "limit": 20,
    "offset": 0
}
```

Returns `lines[]` with invoice key, customer key, product catalog key, product description, bundle caption, chargecode key, amount, and effective date.
