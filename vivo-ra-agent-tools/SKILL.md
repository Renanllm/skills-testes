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
4. Se o contrato retornado nao for compativel com `v0.5`, pare e retorne `status: "blocked"` com `contract_mismatch` no resumo da tool. Nao tente adaptar uma regra nova para contrato antigo.
5. Para cada regra monetaria ou confrontavel, chame `POST /agent-tools/catalog/search` usando o alvo comercial e tipos provaveis de entidade.
6. Crie um rascunho de regra com alvo, comportamento esperado, evidencias, datas, condicoes CRM/bundle, aprovacao binaria e incertezas. IDs CRM sao opcionais, mas todo ID explicitamente declarado no dossie deve ser preservado no JSON estruturado, mesmo quando o CRM mock nao confirmar contrato.
7. Chame `POST /agent-tools/billing/candidate-discovery` com `strategy: "high_recall"`.
8. Quando a regra afetar uma familia/plano de produto, reprecificacao, gratuidade ampla, ou quando o dossie disser "todos os canais", "todos os IDs", "todos os fluxos" ou equivalente, chame `POST /agent-tools/billing/product-family-candidates` antes de fechar o predicado final.
9. Escolha candidatos usando primeiro `chargecode_description` e `bill_message_text`; depois use `productcatalog_description`, papel da linha, contexto de bundle e chargecode inferido para qualificar. Nao use preco esperado, valor faturado ou janelas de valor como criterio de descoberta.
10. Para candidatos amplos ou ambiguos, chame `POST /agent-tools/billing/candidate-clusters` e `POST /agent-tools/invoices/sample-lines`.
11. Use `POST /agent-tools/billing/line-identity-search` ou `POST /agent-tools/billing/identifier-search` quando precisar entender relacoes entre produto, bundle, descricao ou charge code.
12. Produza `candidateQualification` com candidatos incluidos, excluidos e pendentes. Para cada candidato retornado, preserve `billingContext` com charge codes, chargecode descriptions, bill message texts, productcatalog keys, productcatalog descriptions, bundle captions, amostras, sinais, papel da linha, origem do match e tools de origem.
13. Chame `POST /agent-tools/billing/qualification-validate` com o predicado final proposto.
14. Quando a regra depender de elegibilidade por cliente, produto/oferta CRM, bundle CRM, vigencia de contrato, praca/regiao, ativacao, gratuidade por tempo desde contratacao, ou quando houver candidatos concorrentes que precisem de contexto de cliente para desambiguar, chame `POST /agent-tools/crm/contracts/search`. Use esses dados como contexto externo disponivel; se o mock nao tiver dados suficientes, preencha `required_crm_checks` e mantenha a ressalva.
15. Chame `POST /agent-tools/rules/context` para o produto/familia alvo antes de fechar `ruleRelationship`. Use `targetName`, `targetAliases`, `chargecodeKeys`, IDs CRM, bundles e vigencia conhecidos; se tiver uma relacao proposta, envie `proposedRelationship` para validar ciclos. Depois chame `POST /agent-tools/rules/validate` e, quando precisar de detalhe adicional de predicado, `POST /agent-tools/rules/conflicts`. Use as respostas para preencher `ruleSet`, `ruleRelationship`, `stacking` e perguntas abertas quando a prioridade ainda depender de revisao humana.
16. Quando houver concorrencia de regras, CRM, bundle, praca/regiao, vigencia de contrato ou elegibilidade pendente, chame `POST /agent-tools/rules/applicability-preview` para validar quais linhas/unidades ficariam `eligible`, `unknown`, `needs_crm` ou `needs_bundle_eligibility`.
17. Opcionalmente chame `POST /agent-tools/audit/preview` apenas para uma pequena amostra. Nao calcule impacto final em toda a base.
18. Retorne o JSON descrito em `references/output-contract.md`.

## Regras Duras

- Regra nasce exclusivamente de uma declaracao monetaria, precificacao, desconto, gratuidade, tarifa, presenca/ausencia monetaria ou condicao financeira explicitamente presente no dossie. Candidato de billing, chargecode, product description ou bundle encontrado nas tools nunca cria uma regra nova sozinho.
- Antes de criar `regras_financeiras`, enumere mentalmente as declaracoes monetarias do dossie. Cada regra financeira deve corresponder a uma dessas declaracoes e preencher `source_claim_id` com um identificador estavel dessa declaracao, por exemplo `claim-001`.
- Crie uma regra separada apenas quando mudar o comportamento economico: valor esperado, formula de calculo, vigencia, condicao de elegibilidade, prioridade/stacking ou alvo comercial explicitamente declarado no dossie. Nao separe regra apenas porque encontrou outro candidato, bundle, caption ou chargecode.
- Se um candidato parecer produto/variante/bundle relacionado, mas o dossie nao declarar valor, formula ou condicao financeira propria para ele, mantenha-o como `pending` ou `exclude` em `candidate_sets_resumo`, ou como `itens_mapeados_nao_suportados` vinculado ao claim original. Nao promova esse candidato para `regras_financeiras`.
- Itens em `itens_mapeados_nao_suportados` sao lacunas/politicas extraidas para rastreabilidade. Eles devem ter `source_claim_id` quando derivarem de uma declaracao monetaria, mas nao representam regra auditavel nem predicado final.
- Nao varra todas as linhas de fatura. Use candidate discovery, clusters, amostras e validadores.
- Nao calcule impacto financeiro final em toda a base de faturas. O motor deterministico faz isso depois.
- Nao invente aprovacao. Extraia `approval_status: "approved" | "not_approved"` do proprio dossie: use `approved` somente quando houver GO/aprovacao/decisao de seguir explicitamente aplicavel a regra; use `not_approved` quando houver NOGO, cancelamento, decisao de nao seguir, "sem atuacao", "nao aprovado", "nao iniciado", item ainda em discussao ou ausencia de aprovacao explicita.
- Nao classifique como NOGO apenas porque apareceu a frase operacional "Emitir GO / NO GO da etapa". Essa frase e o nome da tarefa. A decisao real vem da coluna/status da aprovacao da fase, do status do projeto ou de uma declaracao textual de NOGO/cancelamento/impeditiva.
- Toda regra financeira e todo item monetario mapeado deve carregar `approval_status` e `approval_evidence`. Se a aprovacao estiver no nivel do dossie inteiro, replique a decisao em cada regra derivada daquele dossie e explique a evidencia.
- Regra `not_approved` pode ser mapeada para rastreabilidade, mas nao deve gerar predicado final ativo nem impacto financeiro. Use `ruleSituation: "needs_review"` ou `not_applicable` conforme o caso, e explique em `support.unsupportedReasons`/`situationRationale`.
- Nao transforme instrucoes puramente operacionais em regras deterministicas de faturamento. Retorne-as como nao confrontaveis ou itens mapeados nao suportados quando forem uteis para rastreabilidade.
- Nao use descricao ampla, nome do produto ou nome do bundle como unico predicado final de regra monetaria de linha.
- Para regras monetarias de linha de fatura, resolva o predicado final para `chargecodeKeyIn` sempre que possivel. Se isso nao for possivel, marque `needs_mapping` ou `needs_agent_qualification`.
- Se a regra se aplica a um bundle, identifique quais linhas de cobranca sao afetadas. Alvo bundle nao significa automaticamente todas as linhas do bundle.
- Nao exclua um candidato apenas por aparecer dentro de um bundle/caption. Se `productcatalog_description` representar diretamente o produto/plano afetado, trate o bundle como contexto e inclua o candidato quando o dossie tiver escopo amplo.
- Classifique o papel da linha antes da decisao final: `direct_product_charge` entra no predicado; `chargecode_description_match` e `sva_ambiguous` precisam ser qualificados pelo agente; `discount`, `different_variant`, `plan_with_benefit` e `context_only` nao entram sem justificativa explicita.
- Se candidatos forem amplos demais, mantenha-os nos candidate sets, mas exclua ou marque como pendente na qualificacao.
- Nao retorne candidato ponderado sem `billingContext` estruturado. A decisao do agente precisa carregar o contexto de fatura usado para include, exclude ou pending.
- Para toda regra financeira, preencha `chargecode_candidates_json`, `disambiguation_json`, `stacking_json` e `required_crm_checks` quando houver candidatos ou dados externos faltantes. Se nao houver concorrencia ou checks externos, use arrays vazios e explique `not_applicable`.
- Dados retornados por `POST /agent-tools/crm/contracts/search` nao criam uma regra nova sozinhos. Eles servem para qualificar elegibilidade, vigencia, CRM product/offer IDs, bundle CRM, praca/regiao e ambiguidades entre candidatos de billing.
- CRM, bundle CRM e elegibilidade de bundle nao criam preco. Eles dizem quando a regra se aplica, como desambiguar candidatos e quais checks externos faltam. O preco/formula/beneficio sempre deve vir de uma declaracao monetaria do dossie.
- IDs CRM sao opcionais. Nunca invente `crmProductIds`, `crmOfferIds` ou `bundleCrmIds`. Quando o dossie ou o mock CRM nao trouxer IDs, use arrays vazios em `externalConditions.crm`, registre `required_crm_checks` e explique a lacuna em `disambiguation`.
- Preserve IDs declarados no dossie separadamente de IDs confirmados no CRM mock. Se o dossie trouxer `Product ID`, `ID Produto`, `ID CRM Produto` ou equivalente, preencha `externalConditions.crm.crmProductIds` e `crmProductIdsFromDossier`. Se trouxer `Offer ID`, `ID Oferta`, `Oferta ID` ou equivalente, preencha `crmOfferIds` e `crmOfferIdsFromDossier`. Se trouxer `Bundle ID`, `ID Bundle`, `ID oferta bundle` ou equivalente, preencha `bundleCrmIds` e `bundleCrmIdsFromDossier`. Se trouxer `Service ID`, preencha `serviceIdsFromDossier` e tambem `declaredCrmIds` com `idType: "service_id"`.
- A ausencia do ID no CRM mock nunca apaga um ID declarado no dossie. Nesse caso mantenha o ID em `*FromDossier`, deixe o respectivo `*ConfirmedInMock` vazio, e use `declaredCrmIds[].verificationStatus: "declared_unverified"` ou `"not_found_in_crm_mock"`.
- Para cada ID declarado relevante, adicione um item em `externalConditions.crm.declaredCrmIds` com `id`, `idType`, `source: "dossier"`, `verificationStatus`, e evidencia curta (`source`, `page` quando souber, `quote`). Se o rotulo for apenas `ID` e o contexto nao provar produto/oferta/bundle, use `idType: "unknown_crm_id"` e explique a ambiguidade em `rationale`, mas nao descarte o ID.
- Para gratuidade ou desconto relativo a contratacao/ativacao, preencha `eligibilityWindow` dentro de `rule_draft_json` e, quando conveniente para persistencia, tambem `eligibility_window_json`. Use `anchor: "crm.activation_date"`, `activationDatePolicy: "relative_to_activation"`, duracao em dias/meses, campo de data da fatura usado e `requiredChecks: ["activation_date"]`.
- Para regras ligadas a bundle/oferta bundle, modele o bundle em `externalConditions.bundleEligibility` e/ou `externalConditions.crm.bundleCrmIds`; mantenha o predicado final nas linhas de cobranca afetadas. Se a elegibilidade do bundle nao puder ser comprovada, use `needs_bundle_eligibility`.
- Nao use `expected.amount`, preco alvo, valor faturado, `netAmount` ou janelas de valor para selecionar candidatos. Esses valores entram na logica de regra/auditoria depois que as linhas candidatas forem encontradas por descricao/chargecode.
- Trate `c.chargetotalamount` como campo monetario oficial da POC.
- Use apenas `ruleSituation: "executable" | "needs_review" | "not_applicable"` como situacao principal da regra. Nao invente status livres. Coloque motivos tecnicos em `dependencyCodes`, `support.unsupportedReasons`, `disambiguation` e `required_crm_checks`.
- Preserve `support.confrontabilityStatus` apenas como detalhe tecnico de compatibilidade. A UI e a esteira devem usar `ruleSituation` como situacao principal.
- Para toda regra financeira, preencha `ruleSet` e `ruleRelationship`. Quando nao souber a prioridade dentro do conjunto, use `relationshipType: "requires_manual_review"` e explique o motivo.
- O criterio `highest_expected_amount_for_underbilling` e fallback somente para recuperacao/underbilling quando ha concorrencia de regras no mesmo contexto e o CRM/taxonomia ainda nao desambigua. Nao use esse criterio para credito ao cliente, cobranca a maior, nem como precedencia universal.
- Quando a regra depender de elegibilidade de bundle, preencha `externalConditions.bundleEligibility` e adicione `needs_bundle_eligibility` em `dependencyCodes` se a fatura/CRM mock nao conseguir confirmar a elegibilidade.
- Quando o dossie trouxer data de vigencia, toda regra monetaria confrontavel deve carregar `effectiveFrom` e `effectiveTo` dentro de `rule_draft_json`, e `valid_from` e `valid_to` no envelope final. Use `null` para data fim ausente.
- Apenas regras com `support.confrontabilityStatus: "confrontable_deterministic"` podem gerar impacto financeiro. Lacunas de CRM, evento de assinatura, entitlement, mapping, quantidade de uso e preco de referencia devem ser explicitas.

## Disciplina de Saida

Retorne um unico objeto JSON. Inclua resumo das chamadas de tool, evidencias do dossie, raciocinio dos candidatos, respostas de validacao, conflitos e perguntas abertas. Se as tools estiverem indisponiveis, retorne `status: "blocked"` com os nomes das tools que falharam e sem inventar mappings.
