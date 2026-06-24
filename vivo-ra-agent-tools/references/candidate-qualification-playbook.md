# Candidate Qualification Playbook

Qualification converts broad candidate sets into a final executable predicate or an explicit gap.

## Required Inputs

Before qualifying, inspect:

- `candidateBuckets` from candidate discovery;
- clusters by chargecode, productcatalog, and bundle caption;
- sample invoice lines for each ambiguous bucket;
- catalog entity aliases and relationships when relevant;
- existing rules and conflicts.

Candidate discovery must be based on product description, bundle context, `productcatalog_description`, line identities and inferred chargecode keys. Do not use expected price, billed amount, `netAmount`, or amount windows to select candidates. Monetary values are used later to define and audit the rule, after the candidate lines have been found.

## Include, Exclude, Pending

For every candidate set, choose one:

- `include`: evidence and billing context support that this invoice line is affected by the rule.
- `exclude`: candidate is a neighbor, broad description hit, unrelated product, operational-only item, or unsupported scope.
- `pending`: potentially related but lacks enough evidence or mapping.

Each decision needs a short rationale.

## Billing Context Per Candidate

For every candidate returned in the rule package, preserve the billing context used by the agent in `billingContext`.

When available, include:

- `candidateSetKind`;
- `predicate`;
- `chargecodeKeys`;
- `productcatalogKeys`;
- `productcatalogDescriptions`;
- `bundleOfferCaptions`;
- `billingLineIdentityIds`;
- `sampleInvoiceLineIds`;
- `lineCount`, `invoiceCount`, `customerCount`, `netAmount`;
- `positiveSignals`, `negativeSignals`, `matchedOn`, `recommendedDecision`;
- `sourceTools`.

Use candidate discovery as the starting source, clusters to fill chargecode/productcatalog/bundle/description combinations, sample lines for concrete line examples, and line identity search when the line only makes sense through `bundle + productcatalog_description + chargecode_key`.

## Final Predicate Rules

For monetary line-level rules:

1. Prefer `chargecodeKeyIn`.
2. Use multiple charge codes when the same commercial product is represented by multiple billing variants.
3. Use `productcatalogKeyIn` or `bundleOfferCaptionIn` only when chargecode cannot express the target and explain the risk.
4. Do not use `descriptionContains` alone as final predicate.
5. Validate with `/billing/qualification-validate`.
6. If `isExecutable` is false, return `needs_mapping`.

## Bundle Neighbor Rule

If candidate discovery returns bundle neighbors, sibling components, or broad description hits, exclude them unless the dossier says the bundle itself is the billing target or the evidence shows the same commercial product is represented by that billing identity.

## Qualification Output Fragment

```json
{
  "candidateQualification": {
    "status": "agent_qualified_needs_human_confirmation",
    "included": [
      {
        "candidateSetId": "cand-chargecode-RMEXAMPLE001",
        "reason": "Direct chargecode token match and invoice samples show the target service."
      }
    ],
    "excluded": [
      {
        "candidateSetId": "cand-description-broad-match",
        "reason": "Description search is broad and can include bundle captions or adjacent products."
      }
    ],
    "pending": [],
    "finalPredicate": {
      "chargecodeKeyIn": ["RMEXAMPLE001", "RMEXAMPLE002"]
    },
    "rationale": "The final predicate uses direct charge codes rather than broad description or bundle context."
  }
}
```

## Red Flags

Mark `needs_agent_audit` or `needs_mapping` when:

- a chargecode maps to many unrelated productcatalog keys;
- the only signal is a broad bundle caption;
- expected value depends on rule precedence not available in existing rules;
- the target is a bundle but the dossier does not say whether all component lines are affected;
- the same invoice line could be governed by another active rule with higher priority.
