# DSL de Regras

Represente somente regras monetarias ou confrontaveis contra fatura.

## Campo Monetario Oficial

Use `c.chargetotalamount`. Em respostas da API, o mesmo conceito pode aparecer como `chargeTotalAmount` ou `charge_total_amount`.

## Datas

Quando o dossie trouxer vigencia, copie para `effectiveFrom` e `effectiveTo`.

Quando o dossie nao for claro, use a data efetiva/competencia da linha como premissa de auditoria e registre a incerteza em `assumptions` ou `openQuestions`.

## Shape Canonico

```json
{
  "ruleName": "Nome humano da regra",
  "ruleType": "free_period | fixed_price | discount | exclusion | migration_price | other_monetary",
  "dossierCode": "20131",
  "target": {
    "entityName": "Vivo Recado",
    "entityKind": "product | variant | bundle | plan | offer | unknown",
    "entityIds": [],
    "affectedScope": "single_charge_line | product | bundle | plan | account | unknown"
  },
  "financialImpact": {
    "monetary": true,
    "impactKind": "customer_credit_risk | recovery_opportunity | validation_only",
    "amountField": "c.chargetotalamount"
  },
  "predicate": {
    "chargecodeKeyIn": ["RMVIVORECADM"],
    "productcatalogKeyIn": [],
    "bundleOfferCaptionIn": [],
    "descriptionContains": "somente recall"
  },
  "expected": {
    "kind": "free | fixed_amount | discount_percent | no_charge | custom_logic",
    "amount": 0,
    "amountField": "c.chargetotalamount",
    "tolerance": 0.01
  },
  "precedence": {
    "priority": 100,
    "stackingPolicy": "exclusive | stackable | cancels_lower_priority | requires_manual_review"
  },
  "evidence": [
    {
      "source": "20131.pdf",
      "page": 1,
      "quote": "trecho curto",
      "rationale": "por que esse trecho sustenta a regra"
    }
  ],
  "confidence": 0.8
}
```

## Status

- `confrontable`: regra monetaria com predicado executavel.
- `needs_mapping`: regra monetaria sem identificador de billing.
- `needs_agent_audit`: ha candidatos, mas eles sao ambiguos ou amplos.
- `needs_review`: evidencia, data, valor ou precedencia ainda incertos.
- `not_applicable`: item operacional ou informacional.
- `blocked`: tools indisponiveis ou falha impeditiva.

## Tipo de Impacto

- `customer_credit_risk`: esperado menor que faturado; possivel credito ao cliente.
- `recovery_opportunity`: esperado maior que faturado; possivel subfaturamento.
- `validation_only`: regra de validacao sem delta direto.

## Evidencia

Toda regra financeira precisa ter evidencia curta do dossie. A evidencia deve sustentar o comportamento de faturamento, nao apenas mencionar o produto.
