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
  "chargecodeKeyIn": ["RMVIVORECADM"],
  "productcatalogKeyIn": ["1125259208"],
  "bundleOfferCaptionIn": ["Servico Digital 9 TBF"],
  "descriptionContains": "Vivo Recado",
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
  "query": "Vivo Recado",
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
  "query": "Vivo Recado",
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
  "query": "Vivo Recado",
  "productcatalogKeys": ["1125259208"],
  "bundleOfferCaptions": ["Servico Digital 9 TBF"],
  "chargecodeKeys": ["RMVIVORECADM"],
  "limit": 20
}
```

Use quando a linha so fizer sentido pela composicao `bundle + productcatalog_description + chargecode_key`.

### `POST /agent-tools/billing/candidate-discovery`

Payload:

```json
{
  "targetName": "Vivo Recado",
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
    "chargecodeKeyIn": ["RMVIVORECADM", "RMVIVORECADVT"]
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
    "chargecodeKeyIn": ["RMVIVORECADM"]
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
    "chargecodeKeyIn": ["RMVIVORECADM", "RMVIVORECADVT"]
  },
  "candidateQualification": {
    "includeCandidateSetIds": ["cand-chargecode-RMVIVORECADM"],
    "excludeCandidateSetIds": ["cand-description-vivo-recado"],
    "rationale": "Charge codes diretos representam Vivo Recado; busca por descricao e ampla demais."
  }
}
```

Use antes de retornar predicado final.

### `POST /agent-tools/rules/existing`

Lista regras existentes por dossie, alvo ou status.

### `POST /agent-tools/rules/validate`

Valida formato da regra sem persistir.

### `POST /agent-tools/rules/conflicts`

Busca possiveis conflitos com regras existentes.

### `POST /agent-tools/audit/preview`

Use apenas para pequena amostra. Nao trate preview como resultado financeiro final.
