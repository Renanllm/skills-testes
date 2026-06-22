# Exemplo Vivo Recado

Use como padrao, nao como regra hardcoded universal.

## Alvo

```json
{
  "entityName": "Vivo Recado",
  "entityKind": "product",
  "affectedScope": "single_charge_line"
}
```

Mapping provisiorio da POC:

```json
{
  "chargecodeKeyIn": ["RMVIVORECADM", "RMVIVORECADVT"]
}
```

Por que e aceitavel para POC:

- os charge codes incluem tokens de Vivo Recado;
- candidate discovery/line identity devem sustentar o vinculo;
- foi escolhido como mapping manual inicial para teste.

Nao faca:

- nao pegue toda linha onde bundle/descricao menciona Vivo Recado;
- nao trate bundle caption ampla como prova de que todos os componentes sao Vivo Recado;
- nao inclua servicos vizinhos do bundle sem evidencia.

## Regra Exemplo

```json
{
  "ruleName": "Vivo Recado gratuito - sem cobranca",
  "ruleType": "fixed_price",
  "dossierCode": "20131",
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
    "kind": "fixed_amount",
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

Se o dossie nao sustentar a gratuidade/valor zero, nao use esse comportamento esperado.
