# Contratos dos Endpoints

Base URL:

```txt
https://server-production-4285.up.railway.app
```

Headers para endpoints protegidos:

```http
Authorization: Bearer {{AGENT_TOOLS_TOKEN}}
Content-Type: application/json
```

`GET /agent-tools/health` e publico. Os demais endpoints exigem bearer token. Nunca inclua o token no output.

## Predicado de Linha de Fatura

Formato comum:

```json
{
  "invoiceKeys": ["3332859950_358710428"],
  "chargecodeKeyIn": ["RMEXEMPLO001"],
  "productcatalogKeyIn": ["1234567890"],
  "bundleOfferCaptionIn": ["Bundle Exemplo"],
  "descriptionContains": "Produto Exemplo",
  "minAmount": 0,
  "maxAmount": 10
}
```

Para regra monetaria final, prefira `chargecodeKeyIn`. Use `descriptionContains` apenas para busca ampla, nunca como unico predicado final.

## Endpoints

### `GET /agent-tools/rule-dsl/contract`

Use antes de criar regra. Retorna campo monetario oficial, shape canonico e orientacoes.

### `POST /agent-tools/catalog/search`

Payload tipico:

```json
{
  "query": "Produto Exemplo",
  "entityKinds": ["product", "variant", "bundle", "plan", "offer"],
  "limit": 20,
  "offset": 0
}
```

Use para entender entidades comerciais, aliases e stats. Nao aprove mapping so com catalogo.

### `GET /agent-tools/catalog/entities/:entityId`

Retorna uma entidade com aliases, variantes, relacionamentos e stats de fatura.

### `POST /agent-tools/billing/identifier-search`

Payload:

```json
{
  "query": "Produto Exemplo",
  "identifierKinds": ["chargecode_key", "productcatalog_key", "bundle_offer_caption"],
  "onlyChargecodeProduct1to1": true,
  "limit": 20
}
```

Use para inspecionar estabilidade de identificadores. `isChargecodeProduct1to1` e sinal forte, mas nao e aprovacao automatica.

### `POST /agent-tools/billing/line-identity-search`

Payload:

```json
{
  "query": "Produto Exemplo",
  "productcatalogKeys": ["1234567890"],
  "bundleOfferCaptions": ["Bundle Exemplo"],
  "chargecodeKeys": ["RMEXEMPLO001"],
  "limit": 20
}
```

Use quando a linha so fizer sentido pela composicao `bundle + productcatalog_description + chargecode_key`.

### `POST /agent-tools/billing/candidate-discovery`

Payload:

```json
{
  "targetName": "Produto Exemplo",
  "entityKinds": ["product"],
  "strategy": "high_recall",
  "limit": 20
}
```

Retorna `candidateSets` e buckets:

- `directIdentifierMatches`
- `lineIdentityCandidates`
- `bundleNeighborCandidates`
- `semanticDescriptionCandidates`

Essa busca erra para mais. A precisao vem na qualificacao.

### `POST /agent-tools/billing/candidate-clusters`

Payload:

```json
{
  "predicate": {
    "chargecodeKeyIn": ["RMEXEMPLO001", "RMEXEMPLO002"]
  },
  "groupBy": ["chargecode_key", "productcatalog_key", "bundle_offer_caption"],
  "limit": 30
}
```

Use para ver contagens, clientes, faturas e valores por agrupamento.

### `POST /agent-tools/invoices/sample-lines`

Payload:

```json
{
  "predicate": {
    "chargecodeKeyIn": ["RMEXEMPLO001"]
  },
  "limit": 20,
  "offset": 0
}
```

Use para revisar exemplos de linhas sem ler a base inteira.

### `POST /agent-tools/billing/qualification-validate`

Payload:

```json
{
  "finalPredicate": {
    "chargecodeKeyIn": ["RMEXEMPLO001", "RMEXEMPLO002"]
  },
  "candidateQualification": {
    "includeCandidateSetIds": ["cand-chargecode-RMEXEMPLO001"],
    "excludeCandidateSetIds": ["cand-description-produto-exemplo"],
    "rationale": "Charge codes diretos representam Produto Exemplo; busca por descricao e ampla demais."
  }
}
```

Use antes de retornar predicado final.

### `POST /agent-tools/rules/existing`

Payload tipico:

```json
{
  "dossierCode": "00000",
  "target": {
    "entityName": "Produto Exemplo",
    "entityKind": "product"
  },
  "limit": 20,
  "offset": 0
}
```

Lista regras existentes por dossie, alvo ou status.

### `POST /agent-tools/rules/validate`

Payload obrigatorio: a regra deve ir dentro de `ruleDraft`.

```json
{
  "ruleDraft": {
    "ruleName": "Produto Exemplo gratuito - sem cobranca",
    "ruleType": "fixed_price",
    "dossierCode": "00000",
    "target": {
      "entityName": "Produto Exemplo",
      "entityKind": "product",
      "affectedScope": "single_charge_line"
    },
    "financialImpact": {
      "monetary": true,
      "impactKind": "validation_only",
      "amountField": "c.chargetotalamount"
    },
    "predicate": {
      "chargecodeKeyIn": ["RMEXEMPLO001", "RMEXEMPLO002"]
    },
    "expected": {
      "kind": "free",
      "amount": 0,
      "amountField": "c.chargetotalamount"
    },
    "evidence": [
      {
        "source": "00000.pdf",
        "page": 1,
        "quote": "Produto Exemplo passara a ser gratuito",
        "rationale": "Evidencia a gratuidade do servico."
      }
    ],
    "confidence": 0.8
  }
}
```

Valida formato da regra sem persistir. Nao envie `{ "rule": ... }`; isso retorna erro 422.

### `POST /agent-tools/rules/conflicts`

Payload obrigatorio: a regra deve ir dentro de `ruleDraft`.

```json
{
  "ruleDraft": {
    "ruleName": "Produto Exemplo gratuito - sem cobranca",
    "ruleType": "fixed_price",
    "dossierCode": "00000",
    "target": {
      "entityName": "Produto Exemplo",
      "entityKind": "product",
      "affectedScope": "single_charge_line"
    },
    "predicate": {
      "chargecodeKeyIn": ["RMEXEMPLO001", "RMEXEMPLO002"]
    },
    "expected": {
      "kind": "free",
      "amount": 0,
      "amountField": "c.chargetotalamount"
    }
  }
}
```

Busca possiveis conflitos com regras existentes.

### `POST /agent-tools/audit/preview`

Use apenas para pequena amostra. Nao trate preview como resultado financeiro final.
