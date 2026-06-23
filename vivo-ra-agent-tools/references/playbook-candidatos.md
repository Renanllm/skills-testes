# Playbook de Candidatos

A descoberta deve priorizar recall. A qualificacao decide o que entra, sai ou fica pendente.

## Descoberta

1. Comece pelo nome comercial do alvo, por exemplo `Produto Exemplo`.
2. Pesquise catalogo com `product`, `variant`, `bundle`, `plan`, `offer`.
3. Chame candidate discovery com `strategy: "high_recall"` e envie o `ruleDraft` completo, incluindo `target` e `expected.amount` quando existir.
4. Analise buckets nesta ordem:
   1. `directIdentifierMatches`
   2. `expectedAmountCandidates`
   3. `lineIdentityCandidates`
   4. `bundleNeighborCandidates`
   5. `semanticDescriptionCandidates`

Use `targetSearchTerms` para entender a decomposicao comercial do alvo e `billingSearchTerms` para entender quais termos foram usados nas buscas de fatura. Quando o dossie trouxer uma variante comercial, como plano/tier/familia/premium/standard, confira `derivedRuleTargets`.

## Significado dos Buckets

### `directIdentifierMatches`

Melhor sinal. Charge codes ou identificadores que batem diretamente com tokens do alvo. Ainda assim, valide cobertura e amostras antes de propor o predicado final.

### `lineIdentityCandidates`

Contexto composto de `productcatalog_key`, `productcatalog_description`, `bundle_offer_caption` e `chargecode_key`.

Use para entender linhas em que o produto so fica claro pela combinacao.

### `expectedAmountCandidates`

Sinal de recall para regras com valor fixo, gratuidade ou desconto. Combina valor esperado com termos amplos do produto base. Nao use sozinho como predicado final, mas nunca ignore quando houver linhas.

### `bundleNeighborCandidates`

Podem ser componentes vizinhos dentro de bundle. Exclua a menos que o dossie diga que a regra afeta o bundle inteiro ou aquele componente especifico.

### `semanticDescriptionCandidates`

Sinal mais fraco. Busca textual pode capturar bundle, descricao ampla ou servicos vizinhos. Nunca use sozinho como predicado final monetario.

## Qualificacao

Para cada candidato, decida:

- `include`: contexto e evidencia sustentam que a linha e afetada pela regra.
- `exclude`: vizinho, match amplo, produto diferente, item operacional ou escopo errado.
- `pending`: pode ser relacionado, mas falta evidencia/mapping.

Use os campos `positiveSignals`, `negativeSignals`, `matchedOn` e `recommendedDecision` como entrada da decisao. Se `recommendedDecision` vier `include` ou `pending`, revise `sample-lines` ou `candidate-clusters` antes de excluir.

O predicado final de regra monetaria deve preferir:

```json
{
  "chargecodeKeyIn": ["RMEXEMPLO001", "RMEXEMPLO002"]
}
```

Use `productcatalogKeyIn` ou `bundleOfferCaptionIn` apenas com justificativa explicita.

## Alertas

Marque `needs_mapping` ou `needs_agent_audit` quando:

- o unico sinal e bundle caption ampla;
- ha muitos productcatalog keys diferentes sem explicacao;
- existe possivel regra com maior prioridade;
- nao esta claro se a regra afeta produto isolado ou bundle inteiro;
- o predicado final nao consegue chegar em `chargecodeKeyIn`.
