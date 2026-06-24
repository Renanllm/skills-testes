# Example: Vivo Recado

Use this as a pattern, not as a hard-coded rule for every dossier.

## What Matters

For this POC, only rules that can affect invoice value matter. If the dossier includes operational instructions such as disabling IDs in PRM, migration steps, communications, or setup tasks, classify them as non-confrontable unless they directly define expected invoice amount.

## Target

Commercial target:

```json
{
  "entityName": "Vivo Recado",
  "entityKind": "product",
  "affectedScope": "single_charge_line"
}
```

Provisional billing mapping for the manual test:

```json
{
  "chargecodeKeyIn": ["RMVIVORECADM", "RMVIVORECADVT"]
}
```

Why this mapping is acceptable for the POC:

- the charge code tokens directly include the Vivo Recado name;
- candidate discovery returns them as direct identifier matches;
- the mapping was selected manually for this test after reviewing invoice fields.

What not to do:

- do not match every line where the bundle or description merely contains "Vivo Recado";
- do not treat a broad bundle caption like "Vivo Recado, NBA, ..." as proof that every component line is a Vivo Recado billing line;
- do not include sibling digital service charge codes unless the dossier rule explicitly covers them.

## Example Rule Draft

```json
{
  "ruleName": "Vivo Recado - expected no charge",
  "ruleType": "free_period",
  "dossierCode": "T3DMND0020131",
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
  "predicate": {
    "chargecodeKeyIn": ["RMVIVORECADM", "RMVIVORECADVT"]
  },
  "expected": {
    "kind": "free",
    "amount": 0,
    "amountField": "c.chargetotalamount",
    "tolerance": 0.01
  },
  "precedence": {
    "priority": 100,
    "stackingPolicy": "requires_manual_review"
  },
  "confidence": 0.82
}
```

Attach dossier evidence before returning the rule as confrontable. If the dossier does not support the free/no-charge expectation, do not use this example's expected behavior.

## Expected Tool Sequence

1. `GET /agent-tools/rule-dsl/contract`
2. `POST /agent-tools/catalog/search` with `query: "Vivo Recado"`
3. `POST /agent-tools/billing/candidate-discovery` with `targetName: "Vivo Recado"`
4. `POST /agent-tools/billing/candidate-clusters` for direct charge codes and broad candidates
5. `POST /agent-tools/invoices/sample-lines` for ambiguous candidates
6. `POST /agent-tools/billing/qualification-validate`
7. `POST /agent-tools/rules/existing`
8. `POST /agent-tools/rules/validate`
9. `POST /agent-tools/rules/conflicts`
