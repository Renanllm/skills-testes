# Candidate Discovery Playbook

Candidate discovery must favor recall first, then qualification. It is acceptable to return too many candidates if the qualification step explicitly excludes false positives.

## Search Strategy

1. Start from the dossier target name, such as `Vivo Recado`.
2. Search catalog with likely kinds: `product`, `variant`, `bundle`, `plan`, `offer`.
3. Call candidate discovery with `strategy: "high_recall"` and `limit` between 10 and 20.
4. If the dossier names a known identifier, pass it in `identifiers`.
5. Inspect buckets in this order:
   1. `directIdentifierMatches`
   2. `lineIdentityCandidates`
   3. `bundleNeighborCandidates`
   4. `semanticDescriptionCandidates`

## Bucket Semantics

### directIdentifierMatches

Best signal. These are charge codes whose value directly matches target tokens or have strong deterministic behavior.

Example for Vivo Recado:

- `RMVIVORECADM`
- `RMVIVORECADVT`

These are strong candidates, but still validate coverage and samples.

### lineIdentityCandidates

Composed billing context: productcatalog key, product description, bundle caption, and chargecode key. Use this when the product only becomes clear through the combination.

Translate final predicates to supported fields, preferably `chargecodeKeyIn`. Do not return unsupported composed objects as final predicates.

### bundleNeighborCandidates

These are charge codes discovered through broader billing/catalog context. They can be sibling components inside a bundle, such as unrelated VAS products around a digital service bundle.

Do not include them unless the dossier rule explicitly applies to that neighbor or to the entire bundle charge structure.

### semanticDescriptionCandidates

Weakest signal. Description search is intentionally broad and may match bundle captions, product descriptions, or adjacent services.

Never use a semantic description candidate as the only final predicate for a monetary rule. Use it to discover identifiers, clusters, and samples.

## When Product/Bundle Mappings Are Not in the Dossier

Most dossiers will not name `chargecode_key`, `productcatalog_key`, or exact invoice descriptions. Infer a broad candidate set by product, bundle, and description, then qualify it with:

- candidate clusters;
- sample invoice lines;
- 1:1 chargecode-product stats;
- catalog aliases and relationships;
- whether the line amount looks like the expected target charge;
- whether the line belongs to the same commercial bundle but is a different product.

If the candidate set is still uncertain, keep the rule and mark `needs_mapping` or `needs_agent_audit`. Do not collapse uncertainty into a fake mapping.
