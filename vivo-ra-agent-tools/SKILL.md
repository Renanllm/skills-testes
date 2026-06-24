---
name: vivo-ra-agent-tools
description: Use quando um agente do Tela precisar ler PDFs de dossies Vivo RA, consultar a API de catalogo/faturamento, extrair regras monetarias ou confrontaveis, descobrir candidatos de billing, qualificar predicados finais, validar conflitos e retornar um pacote JSON auditavel.
---

# Vivo RA Agent Tools

Use esta skill para transformar um dossie Vivo RA em rascunhos auditaveis de regras financeiras. O agente pode inferir alvos comerciais a partir do dossie, mas deve usar a API de tools para resolver candidatos e validar predicados executaveis.

Idioma obrigatorio: escreva todo texto livre em portugues brasileiro. Preserve em ingles somente chaves JSON, nomes de endpoints, nomes de campos tecnicos, valores de enum, charge codes, productcatalog keys e bundle captions.

Base URL padrao das tools:

```txt
https://server-production-4285.up.railway.app
```

Use o bearer token recebido pelo workflow como segredo. Nunca imprima nem retorne o token.

## Referencias Obrigatorias

Leia estes arquivos antes de produzir a resposta final:

- `references/endpoint-contracts.md`: endpoints HTTP disponiveis, payloads, campos de resposta e autenticacao.
- `references/rule-dsl.md`: formato canonico da regra, confrontabilidade, campo monetario, status e confianca.
- `references/candidate-discovery-playbook.md`: como buscar candidatos amplamente sem aprovar falso mapping.
- `references/candidate-qualification-playbook.md`: como incluir, excluir ou deixar candidatos pendentes e produzir predicados finais.
- `references/output-contract.md`: envelope JSON final.

Leia `references/non-confrontable-items.md` quando o dossie trouxer instrucoes operacionais, CRM, PRM, migracao, comunicacao, ativacao ou desativacao. Leia `references/examples-vivo-recado.md` apenas quando precisar de um exemplo concreto de preenchimento.

## Fluxo Obrigatorio

1. Leia o dossie multimodalmente. Prefira a leitura visual/OCR direta quando o workflow fornecer o PDF.
2. Identifique toda declaracao do dossie que possa afetar valor de fatura. Mapeie declaracoes monetarias nao suportadas com `support.confrontabilityStatus`; mantenha itens puramente operacionais separados.
3. Chame `GET /agent-tools/rule-dsl/contract`.
4. Se o contrato retornado nao for compativel com `v0.3`, pare e retorne `status: "blocked"` com `contract_mismatch` no resumo da tool. Nao tente adaptar uma regra nova para contrato antigo.
5. Para cada regra monetaria ou confrontavel, chame `POST /agent-tools/catalog/search` usando o alvo comercial e tipos provaveis de entidade.
6. Crie um rascunho de regra com alvo, comportamento esperado, evidencias, datas e incertezas.
7. Chame `POST /agent-tools/billing/candidate-discovery` com `strategy: "high_recall"`.
8. Quando a regra afetar uma familia/plano de produto, reprecificacao, gratuidade ampla, ou quando o dossie disser "todos os canais", "todos os IDs", "todos os fluxos" ou equivalente, chame `POST /agent-tools/billing/product-family-candidates` antes de fechar o predicado final.
9. Escolha candidatos usando descricao de produto, papel da linha, contexto de bundle e chargecode inferido. Nao use preco esperado, valor faturado ou janelas de valor como criterio de descoberta.
10. Para candidatos amplos ou ambiguos, chame `POST /agent-tools/billing/candidate-clusters` e `POST /agent-tools/invoices/sample-lines`.
11. Use `POST /agent-tools/billing/line-identity-search` ou `POST /agent-tools/billing/identifier-search` quando precisar entender relacoes entre produto, bundle, descricao ou charge code.
12. Produza `candidateQualification` com candidatos incluidos, excluidos e pendentes. Para cada candidato retornado, preserve `billingContext` com charge codes, productcatalog keys, productcatalog descriptions, bundle captions, amostras, sinais, papel da linha e tools de origem.
13. Chame `POST /agent-tools/billing/qualification-validate` com o predicado final proposto.
14. Chame `POST /agent-tools/rules/existing`, depois `POST /agent-tools/rules/validate`, depois `POST /agent-tools/rules/conflicts`.
15. Opcionalmente chame `POST /agent-tools/audit/preview` apenas para uma pequena amostra. Nao calcule impacto final em toda a base.
16. Retorne o JSON descrito em `references/output-contract.md`.

## Regras Duras

- Nao varra todas as linhas de fatura. Use candidate discovery, clusters, amostras e validadores.
- Nao calcule impacto financeiro final em toda a base de faturas. O motor deterministico faz isso depois.
- Nao persista nem diga que uma regra esta aprovada. Retorne rascunho e qualificacao para revisao.
- Nao transforme instrucoes puramente operacionais em regras deterministicas de faturamento. Retorne-as como nao confrontaveis ou itens mapeados nao suportados quando forem uteis para rastreabilidade.
- Nao use descricao ampla, nome do produto ou nome do bundle como unico predicado final de regra monetaria de linha.
- Para regras monetarias de linha de fatura, resolva o predicado final para `chargecodeKeyIn` sempre que possivel. Se isso nao for possivel, marque `needs_mapping` ou `needs_agent_qualification`.
- Se a regra se aplica a um bundle, identifique quais linhas de cobranca sao afetadas. Alvo bundle nao significa automaticamente todas as linhas do bundle.
- Nao exclua um candidato apenas por aparecer dentro de um bundle/caption. Se `productcatalog_description` representar diretamente o produto/plano afetado, trate o bundle como contexto e inclua o candidato quando o dossie tiver escopo amplo.
- Classifique o papel da linha antes da decisao final: `direct_product_charge` entra no predicado; `discount`, `different_variant`, `plan_with_benefit` e `context_only` nao entram sem justificativa explicita.
- Se candidatos forem amplos demais, mantenha-os nos candidate sets, mas exclua ou marque como pendente na qualificacao.
- Nao retorne candidato ponderado sem `billingContext` estruturado. A decisao do agente precisa carregar o contexto de fatura usado para include, exclude ou pending.
- Nao use `expected.amount`, preco alvo, valor faturado, `netAmount` ou janelas de valor para selecionar candidatos. Esses valores entram na logica de regra/auditoria depois que as linhas candidatas forem encontradas por descricao/chargecode.
- Trate `c.chargetotalamount` como campo monetario oficial da POC.
- Apenas regras com `support.confrontabilityStatus: "confrontable_deterministic"` podem gerar impacto financeiro. Lacunas de CRM, evento de assinatura, entitlement, mapping, quantidade de uso e preco de referencia devem ser explicitas.

## Disciplina de Saida

Retorne um unico objeto JSON. Inclua resumo das chamadas de tool, evidencias do dossie, raciocinio dos candidatos, respostas de validacao, conflitos e perguntas abertas. Se as tools estiverem indisponiveis, retorne `status: "blocked"` com os nomes das tools que falharam e sem inventar mappings.
