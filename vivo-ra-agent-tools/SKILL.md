---
name: vivo-ra-agent-tools
description: Use quando um agente do Tela precisar ler dossies Vivo RA, consultar a API de catalogo/faturamento, extrair somente regras monetarias ou confrontaveis contra fatura, qualificar candidatos de billing e devolver um pacote JSON auditavel em portugues brasileiro.
---

# Vivo RA Agent Tools

Use esta skill para transformar um dossie Vivo RA em rascunhos auditaveis de regras financeiras.

Idioma obrigatorio: escreva todo texto livre em portugues brasileiro. Preserve em ingles apenas chaves JSON, nomes de endpoints, nomes de campos tecnicos, valores de enum, charge codes, productcatalog keys e bundle captions.

Base URL das tools:

```txt
https://server-production-4285.up.railway.app
```

Use o bearer token recebido pelo workflow como segredo. Nunca imprima, retorne ou registre o token em `toolTrace`.

## Referencias Obrigatorias

Leia antes da resposta final:

- `references/contratos-endpoints.md`: endpoints disponiveis, headers, payloads e uso.
- `references/dsl-regras.md`: formato canonico da regra, campo monetario oficial e statuses.
- `references/playbook-candidatos.md`: como descobrir e qualificar candidatos sem aprovar falso mapping.
- `references/contrato-saida.md`: envelope JSON final esperado.

Leia `references/itens-nao-confrontaveis.md` quando o dossie trouxer instrucoes operacionais, PRM, CRM, comunicacao, migracao ou status.

## Fluxo Obrigatorio

1. Leia o PDF multimodalmente.
2. Separe itens monetarios/confrontaveis de itens somente operacionais.
3. Chame `GET /agent-tools/rule-dsl/contract`.
4. Para cada regra monetaria, pesquise o alvo em `POST /agent-tools/catalog/search`.
5. Crie um rascunho de regra com alvo, comportamento esperado, vigencia, evidencia e incertezas. Para preco fixo, desconto ou gratuidade, preencha `expected.amount`.
6. Para cada regra monetaria, chame `POST /agent-tools/billing/candidate-discovery` com `strategy: "high_recall"` e envie o `ruleDraft` completo, nao apenas `targetName`.
7. Analise `targetSearchTerms`, `billingSearchTerms`, `derivedRuleTargets`, `candidateSets`, `positiveSignals`, `negativeSignals` e `recommendedDecision`. Use apenas descricao de produto, contexto de bundle e chargecode inferido para escolher candidatos; nao use preco/valor faturado como criterio de descoberta.
8. Para candidatos amplos, ambiguos, `pending` ou `include`, chame `POST /agent-tools/billing/candidate-clusters` e `POST /agent-tools/invoices/sample-lines`.
9. Use `POST /agent-tools/billing/identifier-search` e `POST /agent-tools/billing/line-identity-search` quando precisar entender relacao entre produto, bundle, descricao e charge code.
10. Produza `candidateQualification` separando incluidos, excluidos e pendentes. Para cada candidato em `candidate_sets_resumo`, preencha `billing_context` com charge codes, productcatalog keys, productcatalog descriptions, bundle captions, amostras, sinais e tools usadas.
11. Chame `POST /agent-tools/billing/qualification-validate` com o predicado final proposto.
12. Chame `POST /agent-tools/rules/existing`, `POST /agent-tools/rules/validate` e `POST /agent-tools/rules/conflicts`.
13. Nao calcule impacto financeiro final na base inteira. No maximo use `POST /agent-tools/audit/preview` para pequena amostra.
14. Retorne um unico JSON seguindo `references/contrato-saida.md`.

## Regras Duras

- Nao varra todas as linhas de fatura.
- Nao calcule recuperacao/credito total por conta propria.
- Nao persista nem diga que a regra esta aprovada.
- Nao transforme item operacional em regra financeira.
- Nao use nome do produto, descricao ampla ou bundle como unico predicado final de regra monetaria.
- Nao exclua candidato com `recommendedDecision: "include"` ou `"pending"` sem antes revisar amostras ou clusters.
- Nao conclua que uma variante nao existe so porque o nome completo da variante nao aparece em `productcatalog_description`; use `derivedRuleTargets`, `billingSearchTerms`, `lineIdentityCandidates`, clusters e amostras.
- Nao use `expected.amount`, preco alvo, valor faturado ou janela de valor para escolher candidatos. Esses valores servem para definir a regra/auditoria depois que as linhas candidatas ja foram encontradas por descricao/chargecode.
- Nao retorne candidato ponderado sem `billing_context` estruturado. A decisao do agente precisa carregar o contexto de fatura usado para include/exclude/pending.
- Para regra monetaria de linha de fatura, resolva o predicado final para `chargecodeKeyIn` sempre que possivel.
- Se nao houver identificador executavel, marque `needs_mapping` ou `needs_agent_audit`.
- Se a regra fala de bundle, identifique quais linhas de cobranca sao afetadas. Bundle nao significa automaticamente todas as linhas do bundle.
- Use `c.chargetotalamount` como campo monetario oficial da POC.

## Saida

Retorne somente um objeto JSON como resposta final do step. Nao crie arquivo, nao use `Write` para salvar `result.json` e nao coloque o resultado final em anexo. Inclua evidencias, candidatos, qualificacao, validacoes, conflitos, trace de ferramentas e perguntas abertas. Se uma tool falhar, retorne `status: "blocked"` ou `needs_agent_audit` com a falha, sem inventar mapping.
