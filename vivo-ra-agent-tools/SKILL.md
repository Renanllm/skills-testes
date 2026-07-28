---
name: vivo-ra-agent-tools
description: Use quando um agente do Tela precisar criar rascunhos auditaveis de regras financeiras Vivo RA usando agent tools, DSL v0.6, CRM oficial enriquecido, aplicabilidade estrita, bundle, DDD, vigencia e hierarquia de regras.
---

# Vivo RA Agent Tools

Use esta skill para transformar a triagem de um dossie Vivo RA em um pacote JSON auditavel de regras financeiras. O preco, desconto, gratuidade, tarifa, ausencia/presenca monetaria ou formula sempre nasce do dossie. Billing, CRM e catalogo servem para descobrir onde auditar e quando aplicar a regra.

Idioma obrigatorio: escreva todo texto livre em portugues brasileiro. Preserve em ingles somente chaves JSON, endpoints, enums, charge codes, productcatalog keys, SOC, bundle captions e nomes tecnicos.

Base URL padrao:

```txt
https://server-production-4285.up.railway.app
```

Use o bearer token recebido pelo workflow como segredo. Nunca imprima nem retorne o token.

## Contrato Curto

- API esperada: `ruleDslContract.version: "v0.6"`.
- Campo monetario oficial da POC: `c.chargetotalamount`.
- Situacoes permitidas de regra: `executable`, `needs_review`, `not_applicable`.
- Uma declaracao monetaria sem regra deve voltar como `claim_disposition: "not_created"`, com `not_created_reason_code` e `not_created_reason`.
- Dossie sem aprovacao clara deve preencher `dossie.invalid_reason_code: "missing_clear_approval"` e `dossie.invalid_reason`.
- Regra valida precisa ter aprovacao clara e impacto financeiro direto auditavel: reprecificacao, desconto, gratuidade, tarifa, ausencia/presenca monetaria ou formula.
- Nao crie regra para inclusao operacional de servico sem cobranca, comunicacao, rollout, PRM/CRM sem efeito financeiro, ou item que nao consiga ser auditado em fatura.

## Fluxo Recomendado

1. Use a triagem previa do workflow como fonte principal de claims. Nao releia o PDF inteiro no caminho feliz.
2. Chame `GET /agent-tools/rule-dsl/contract` apenas para checar `version` e `officialAmountField`; nao pagine nem leia o contrato inteiro. Se nao for `v0.6`, retorne `status: "blocked"` com `contract_mismatch`.
3. Enumere as declaracoes monetarias do dossie e atribua `source_claim_id` estavel: `claim-001`, `claim-002`, etc.
4. Para cada claim, pesquise catalogo com `POST /agent-tools/catalog/search` usando nomes/IDs declarados.
5. Descubra candidatos com `POST /agent-tools/billing/candidate-discovery`.
6. Se houver ID CRM, SOC, contrato, bundle, DDD ou vigencia no dossie, envie `applicability`, `eligibilityFilters` e `selectionPolicy.strictCandidateEligibility: true`.
7. Se a busca estrita retornar zero candidatos, faca no maximo uma busca ampla sem filtros estritos para registrar mapping possivel. Nao inclua candidato amplo como executavel sem cobertura estrita; retorne `needs_mapping` ou `needs_crm` com a lacuna explicita.
8. Para reprecificacao/familia/plano/gratuidade ampla, use `POST /agent-tools/billing/product-family-candidates` uma vez, com os mesmos filtros de aplicabilidade quando existirem.
9. Para candidato ambiguo, use `candidate-clusters` ou `sample-lines` somente quando isso mudar include/exclude/pending.
10. Consulte `POST /agent-tools/crm/contracts/search` quando a regra depender de produto/oferta CRM, SOC, `serviceAgreementKey`, bundle CRM, DDD, ativacao ou vigencia de contrato.
11. Antes de fechar hierarquia, use `POST /agent-tools/rules/context`: primeiro por ID CRM/SOC/contrato, depois por bundle especifico, por ultimo por produto/familia.
12. Valide cada `ruleDraft` com `POST /agent-tools/rules/validate`.
13. Use `rules/conflicts`, `rules/applicability-preview` e `audit/preview` apenas quando houver ambiguidade real que nao foi resolvida por `rules/context` e `rules/validate`.

## Disciplina Anti-Loop

- Nao chame o mesmo endpoint com payload equivalente mais de uma vez.
- Para cada claim, use no maximo uma busca estrita e uma busca ampla de fallback.
- Se a busca estrita por CRM/bundle/DDD/vigencia voltar vazia, registre a lacuna. Nao tente compensar com varias consultas semanticas.
- Use limites compactos: `catalog/search.limit <= 5`, busca estrita `candidate-discovery.limit <= 10` e fallback amplo `candidate-discovery.limit <= 5`.
- Use no maximo 5 `targetAliases` por chamada. Nao copie todos os aliases retornados por catalogo para `candidate-discovery`.
- Quando chamar endpoints via Bash/curl, nao imprima JSON bruto grande. Salve em arquivo apenas se necessario e imprima um resumo com contagens, top candidatos e warnings.
- Antes de salvar arquivo em `/home/user/generated-files`, execute `mkdir -p /home/user/generated-files`.
- Nao abra `references/*.md` no caminho feliz. Use os payloads minimos abaixo. Se houver erro de schema, corrija usando a mensagem de erro do endpoint; nao procure arquivo de referencia.
- Nao imprima candidato completo (`candidateSets[0]`, objeto inteiro de resposta ou JSON bruto). Resuma com contagens e ate 3 itens `{candidateSetId, chargecodeDescription, matchedAliases, recommendedDecision, confidence}`.
- Orcamento alvo: ate 12 invocacoes Bash por run, incluindo contrato, catalogo, candidato estrito, fallback amplo, CRM, contexto, validacoes e escrita do resultado.
- Nao use `expected.amount`, valor faturado, `netAmount`, `positiveAmount`, `negativeAmount`, `minAmount` ou `maxAmount` para descobrir candidatos. Valores monetarios entram na regra depois que o candidato foi encontrado por descricao, bill message, chargecode ou vinculo CRM.
- Nao varra toda a base de faturas. O motor deterministico calcula impacto financeiro depois.
- Nao leia todos os arquivos de referencia por padrao. Leia somente a referencia necessaria para uma duvida especifica.

## Payloads Minimos

Use estes formatos sem consultar referencias quando bastarem:

- Contrato: `GET /agent-tools/rule-dsl/contract | jq '{version, officialAmountField}'`.
- Catalogo: `POST /agent-tools/catalog/search` com `{"query":"Spotify","entityKinds":["product","variant","bundle","plan","offer"],"limit":5}`.
- Candidato estrito: `POST /agent-tools/billing/candidate-discovery` com `targetName`, ate 5 `targetAliases`, `entityKinds`, `identifiers` como array de strings, nunca objeto (ex.: `["0055013624","0055013625"]`), `strategy:"high_recall"`, `applicability`, `eligibilityFilters`, `selectionPolicy:{scopeKind,candidateSearchOrder,strictCandidateEligibility:true}` e `limit:10`.
- Fallback amplo: mesmo endpoint, sem filtros estritos, `strictCandidateEligibility:false` e `limit:5`; use somente para registrar mapping possivel, nao para `executable`.
- CRM: `POST /agent-tools/crm/contracts/search` com IDs declarados (`crmProductIds`, `crmOfferIds`, `bundleCrmIds`, SOC, contrato, DDD, status, `activeOn`) e `limit:20`.
- Contexto: `POST /agent-tools/rules/context` com `targetName`, aliases, `chargecodeKeys`, IDs CRM/SOC/contrato/bundle/DDD, `applicability`, vigencia e relacao proposta quando existir.
- Validacao: `POST /agent-tools/rules/validate` com o `ruleDraft` final; `target` deve ser objeto com `entityName`, nao string.

## Aplicabilidade Estrita

Use estes campos quando existirem no dossie ou forem necessarios para separar regras concorrentes:

- `applicability.crmProductcatalogIdIn`
- `applicability.billingOfferSocCodeIn`
- `applicability.serviceAgreementKeyIn`
- `applicability.assignedBillingOfferKeyIn`
- `applicability.subscriberDddIn`
- `applicability.subscriberStatusKeyIn`
- `applicability.billingOfferStatusIn`
- `applicability.bundleOfferCaptionIn`
- `applicability.billingEffectiveDate`
- `applicability.serviceEffectiveDate`
- `applicability.requireAll: true` quando todas as condicoes forem obrigatorias.

Espelhe o bloco em `predicate.applicability`, `applicability_json`, `selection_policy_json` e `rule_draft_json.selectionPolicy` quando a regra final depender desses filtros.

Regras com ID CRM esperado so podem incluir candidatos com cobertura nas linhas que tenham esse ID CRM. Regras de bundle/pacote so podem incluir candidatos com evidencia do bundle esperado. Regras de componente separado de bundle exigem evidencia direta do produto cobrado e evidencia de pertencimento ao bundle por `bundleOfferCaption`, SOC, contrato ou link CRM.

## IDs CRM

IDs CRM sao opcionais; nunca invente `crmProductIds`, `crmOfferIds` ou `bundleCrmIds`.

Preserve IDs declarados no dossie separadamente de IDs confirmados:

- `Product ID`, `ID Produto`, `ID CRM Produto`: `crmProductIds` e `crmProductIdsFromDossier`.
- `Offer ID`, `ID Oferta`, `Oferta ID`: `crmOfferIds` e `crmOfferIdsFromDossier`.
- `Bundle ID`, `ID Bundle`, `Codigo da oferta`: `bundleCrmIds` e `bundleCrmIdsFromDossier`.
- `Service ID`: `serviceIdsFromDossier`.
- ID generico: `declaredCrmIds[].idType: "unknown_crm_id"` e explique a ambiguidade.

Se o CRM oficial nao confirmar o ID, mantenha o ID em `*FromDossier`, deixe confirmacoes vazias e use `declaredCrmIds[].verificationStatus: "declared_unverified"` ou `"not_found_in_crm"`.

## Hierarquia

Numeros menores em `ruleRelationship.priorityRank` significam maior precedencia.

Ordem de especificidade:

1. ID CRM/SOC/contrato + DDD + janela de data.
2. ID CRM/SOC/contrato.
3. Bundle especifico.
4. Produto/familia.
5. Regra somente por chargecode/billing.

Use `highest_expected_amount_for_underbilling` somente como fallback para recuperacao/underbilling quando ha concorrencia no mesmo contexto e CRM/taxonomia nao desambigua. Nao use para credito ao cliente nem como precedencia universal.

## Gratuidade e Janelas

Para gratuidade ou desconto relativo a contratacao/ativacao:

- `ruleType: "free_period"` quando for periodo gratuito.
- `externalConditions.crm.activationDatePolicy: "relative_to_activation"`.
- `eligibilityWindow.anchor: "crm.activation_date"`.
- duracao em dias ou meses.
- `requiredChecks: ["activation_date"]`.

Se a data de ativacao/contratacao nao estiver disponivel, retorne `needs_crm` ou `needs_subscription_event`; nao marque como executavel.

## Etiqueta Padrao

Para `Etiqueta padrao` ou oferta conjunta:

- use `documentType: "standard_offer_label"`;
- trate `Codigo da oferta` como `bundleCrmId`;
- preserve em `externalConditions.bundleEligibility.bundleCrmIds`, `externalConditions.crm.bundleCrmIds`, `bundleCrmIdsFromDossier` e `declaredCrmIds`;
- crie regras por componente auditavel quando `Precos individuais dos servicos da oferta conjunta` trouxer valor financeiro;
- busque candidatos pelo nome do componente em `chargecode_description` e `bill_message_text`; use `productcatalog_description` e `bundle_offer_caption` apenas para qualificar.

Na POC, documento oficial de etiqueta padrao pode ser `approved` salvo se houver NOGO, cancelamento, bloqueio, nao aprovado, sem atuacao ou ainda em discussao explicitamente no documento.

## Referencias Sob Demanda

Leia somente quando precisar:

- `references/endpoint-contracts.md`: payload/resposta de endpoint especifico.
- `references/rule-dsl.md`: detalhe da DSL v0.6.
- `references/candidate-discovery-playbook.md`: duvida de descoberta de candidatos.
- `references/candidate-qualification-playbook.md`: duvida de include/exclude/pending.
- `references/output-contract.md`: apenas se o schema estruturado do workflow nao bastar.
- `references/non-confrontable-items.md`: itens operacionais, CRM, PRM, migracao ou comunicacao.
- `references/examples-vivo-recado.md`: exemplo concreto.
