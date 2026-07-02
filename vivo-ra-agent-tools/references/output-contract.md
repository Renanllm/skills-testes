# Contrato de Saida

Retorne um unico objeto JSON. Nao inclua texto fora do JSON.

O envelope externo deve usar os nomes de campo em portugues declarados no output format do workflow. As chaves tecnicas da DSL v0.5 devem ficar preservadas dentro dos campos JSON serializados, como `rule_draft_json`, `calculation_json`, `support_json`, `chargecode_candidates_json`, `disambiguation_json`, `stacking_json` e `predicado_final_json`.

```json
{
    "status": "draft | needs_mapping | needs_agent_qualification | needs_review | blocked | completed",
    "dossie": {
        "arquivo": "20131.pdf",
        "codigo": "20131",
        "titulo": "Titulo inferido do dossie",
        "approval_status": "approved | not_approved",
        "approval_evidence": "Trecho curto que comprova GO/NOGO/aprovacao/nao aprovacao. Use null se inexistente."
    },
    "resumo": {
        "narrativa": "Resumo curto em portugues brasileiro.",
        "quantidade_regras_financeiras": 1,
        "quantidade_itens_mapeados_nao_suportados": 2,
        "quantidade_itens_nao_confrontaveis": 1,
        "quantidade_candidate_sets": 4,
        "quantidade_predicados_finais": 1,
        "quantidade_perguntas_abertas": 1
    },
    "regras_financeiras": [
        {
            "source_claim_id": "claim-001",
            "nome_regra": "",
            "approval_status": "approved | not_approved",
            "approval_evidence": "Trecho curto do dossie que comprova a aprovacao ou nao aprovacao da regra.",
            "situacao_regra": "executable | needs_review | not_applicable",
            "dependency_codes": ["needs_crm", "needs_bundle_eligibility"],
            "situation_rationale": "Resumo curto da situacao e dependencias.",
            "tipo_regra": "fixed_price | no_charge | discount | usage_tariff | prorata | bundle_composition | presence_rule | free_period | exclusion | migration_price | other_monetary",
            "status_confrontabilidade": "confrontable_deterministic | confrontable_after_mapping | needs_agent_qualification | needs_mapping | needs_usage_quantity | needs_reference_price | needs_crm | needs_subscription_event | needs_entitlement_inventory | needs_review | not_supported_yet | blocked",
            "alvo_nome": "",
            "alvo_tipo": "product | variant | bundle | plan | offer | unknown",
            "escopo_afetado": "single_charge_line | product | bundle | plan | account | unknown",
            "campo_monetario": "c.chargetotalamount",
            "valid_from": "YYYY-MM-DD ou null",
            "valid_to": "YYYY-MM-DD ou null",
            "comportamento_esperado": "",
            "calculation_kind": "fixed_amount | no_charge | discount_amount | discount_percent | usage_event_tariff | usage_duration_tariff | usage_volume_tariff | daily_rate | prorata_by_period | bundle_component_sum | forbidden_charge | presence_required | custom_logic",
            "required_columns": ["charge_total_amount"],
            "valor_esperado": 0,
            "rule_draft_json": "{\"ruleName\":\"...\",\"approvalStatus\":\"approved\",\"approvalEvidence\":\"GO da regra no dossie\",\"ruleSituation\":\"executable\",\"dependencyCodes\":[],\"calculation\":{\"kind\":\"no_charge\"},\"ruleSet\":{\"key\":\"product:vivo-recado\"},\"ruleRelationship\":{\"relationshipType\":\"independent\"}}",
            "calculation_json": "{\"kind\":\"no_charge\",\"amountField\":\"charge_total_amount\"}",
            "support_json": "{\"confrontabilityStatus\":\"confrontable_deterministic\",\"unsupportedReasons\":[]}",
            "external_conditions_json": "{\"crm\":{\"policy\":\"not_required\"},\"bundleEligibility\":{\"policy\":\"not_required\"}}",
            "eligibility_window_json": "{\"anchor\":\"crm.activation_date\",\"durationMonths\":3,\"invoiceDateField\":\"period_start_date\",\"expectedDuringWindow\":{\"amount\":0,\"calculationKind\":\"no_charge\"},\"requiredChecks\":[\"activation_date\"]}",
            "rule_set_json": "{\"key\":\"product:vivo-recado\",\"targetProductName\":\"Vivo Recado\",\"targetChargecodes\":[\"RMVIVORECADM\"]}",
            "rule_relationship_json": "{\"relationshipType\":\"independent\",\"priorityRank\":100,\"rationale\":\"Sem regra concorrente identificada.\"}",
            "chargecode_candidates_json": "[{\"chargecodeKey\":\"RMVIVORECADM\",\"chargecodeDescription\":\"Vivo Recado\",\"decision\":\"include\",\"sourcePriority\":\"chargecode_description\"}]",
            "disambiguation_json": "{\"missingDisambiguators\":[\"crm_product_id\"],\"requiredCrmChecks\":[\"crm_product_id\"],\"rationale\":\"A fatura nao informa o ID CRM do produto contratado.\"}",
            "stacking_json": "{\"stackingPolicy\":\"requires_manual_review\",\"competingRulePolicy\":\"highest_expected_amount_for_underbilling\",\"rationale\":\"Mesmo chargecode pode ter mais de uma regra comercial.\"}",
            "required_crm_checks": ["crm_product_id", "service_id", "activation_date", "region"],
            "predicado_final_json": "{\"chargecodeKeyIn\":[\"RMVIVORECADM\"]}",
            "candidate_sets_resumo": [
                {
                    "candidate_set_id": "cand-chargecode-RMEXAMPLE001",
                    "decisao": "include | exclude | pending",
                    "candidate_source_priority": "chargecode_description | bill_message_text | productcatalog_description | bundle_caption | catalog_alias | inferred_chargecode",
                    "line_role": "direct_product_charge | plan_with_benefit | discount | different_variant | context_only | sva_ambiguous | chargecode_description_match | unknown",
                    "motivo": "Racional curto em portugues.",
                    "billing_context_json": "{\"chargecode_keys\":[\"RMEXAMPLE001\"],\"productcatalog_descriptions\":[\"Example Product\"]}"
                }
            ],
            "validacao_resumo": "",
            "conflitos_resumo": "",
            "evidencias": [],
            "premissas": [],
            "perguntas_abertas": []
        }
    ],
    "itens_mapeados_nao_suportados": [
        {
            "source_claim_id": "claim-002",
            "titulo": "",
            "approval_status": "approved | not_approved",
            "approval_evidence": "Trecho curto do dossie que comprova a aprovacao ou nao aprovacao do item.",
            "status_suporte": "needs_crm | needs_subscription_event | needs_entitlement_inventory | needs_usage_quantity | needs_reference_price | not_supported_yet | needs_mapping | not_monetary",
            "unsupported_reasons": [],
            "required_external_data": [],
            "tipo_regra_possivel": "",
            "alvo_nome": "",
            "calculation_hint_json": "{}",
            "motivo": "",
            "evidencia": ""
        }
    ],
    "itens_nao_confrontaveis": [
        {
            "titulo": "",
            "motivo": "",
            "evidencia": ""
        }
    ],
    "tool_trace": [
        {
            "tool": "POST /agent-tools/billing/candidate-discovery",
            "finalidade": "Encontrar candidatos amplos",
            "resumo_resultado": "2 candidatos diretos; 1 candidato amplo pendente"
        }
    ],
    "perguntas_abertas_globais": []
}
```

## Contexto Obrigatorio de Candidatos

Todo item em `candidate_sets_resumo` deve preservar o contexto de billing que o agente usou para decidir. Nao retorne apenas id, decisao e motivo.

O formato preferencial para o workflow e `billing_context_json` como string JSON serializada. Se uma versao futura do workflow aceitar objeto profundo, o mesmo conteudo pode aparecer como `billing_context`.

```json
{
    "candidate_set_id": "cand-chargecode-RMEXAMPLE001",
    "decisao": "include | exclude | pending",
    "candidate_source_priority": "chargecode_description",
    "line_role": "direct_product_charge",
    "motivo": "Racional curto em portugues.",
    "billing_context_json": "{\"candidate_set_kind\":\"chargecode_key\",\"predicate\":{\"chargecodeKeyIn\":[\"RMEXAMPLE001\"]},\"chargecode_keys\":[\"RMEXAMPLE001\"],\"chargecode_descriptions\":[\"Example Product\"],\"bill_message_texts\":[\"Example Product\"],\"productcatalog_keys\":[\"1234567890\"],\"productcatalog_descriptions\":[\"Example Product\"],\"bundle_offer_captions\":[\"EXAMPLE BUNDLE\"],\"line_count\":10,\"invoice_count\":8,\"customer_count\":8,\"net_amount\":239,\"positive_signals\":[\"exact_chargecode_description\"],\"negative_signals\":[],\"matched_on\":{\"chargecodeDescription\":true,\"billMessageText\":false,\"productcatalogDescription\":true,\"bundleCaption\":false,\"expectedAmount\":false},\"recommended_decision\":\"include\",\"source_tools\":[\"POST /agent-tools/billing/candidate-discovery\"]}"
}
```

O JSON serializado em `billing_context_json` deve conter, quando disponivel:

```json
{
    "candidate_set_kind": "chargecode_key | billing_line_identity | semantic_description_match | bundle_neighbor",
    "predicate": {
        "chargecodeKeyIn": ["RMEXAMPLE001"]
    },
    "chargecode_keys": ["RMEXAMPLE001"],
    "chargecode_descriptions": ["Example Product"],
    "bill_message_texts": ["Example Product"],
    "productcatalog_keys": ["1234567890"],
    "productcatalog_descriptions": ["Example Product"],
    "bundle_offer_captions": ["EXAMPLE BUNDLE"],
    "billing_line_identity_ids": ["bli-..."],
    "sample_invoice_line_ids": ["invoice-line-id"],
    "line_count": 10,
    "invoice_count": 8,
    "customer_count": 8,
    "net_amount": 239,
    "positive_signals": ["chargecode_token_match", "product_description_match"],
    "negative_signals": [],
    "matched_on": {
        "chargecodeDescription": true,
        "billMessageText": false,
        "chargecodeKey": true,
        "productcatalogDescription": true,
        "bundleOfferCaption": false,
        "expectedAmount": false
    },
    "candidate_source_priority": "chargecode_description",
    "recommended_decision": "include",
    "line_role_suggestion": "direct_product_charge | discount | different_variant | plan_with_benefit | context_only | sva_ambiguous | chargecode_description_match | unknown",
    "ambiguity_reason": null,
    "source_tools": [
        "POST /agent-tools/billing/candidate-discovery",
        "POST /agent-tools/billing/product-family-candidates",
        "POST /agent-tools/billing/candidate-clusters",
        "POST /agent-tools/invoices/sample-lines"
    ],
    "observacoes": "Contexto observado pelo agente."
}
```

Formato objeto equivalente, quando suportado:

```json
{
    "billing_context": {
        "candidate_set_kind": "chargecode_key | billing_line_identity | semantic_description_match | bundle_neighbor",
        "predicate": {
            "chargecodeKeyIn": ["RMEXAMPLE001"]
        },
        "chargecode_keys": ["RMEXAMPLE001"],
        "chargecode_descriptions": ["Example Product"],
        "bill_message_texts": ["Example Product"],
        "productcatalog_keys": ["1234567890"],
        "productcatalog_descriptions": ["Example Product"],
        "bundle_offer_captions": ["EXAMPLE BUNDLE"],
        "billing_line_identity_ids": ["bli-..."],
        "sample_invoice_line_ids": ["invoice-line-id"],
        "line_count": 10,
        "invoice_count": 8,
        "customer_count": 8,
        "net_amount": 239,
        "positive_signals": ["chargecode_token_match", "product_description_match"],
        "negative_signals": [],
        "matched_on": {
            "chargecodeDescription": true,
            "billMessageText": false,
            "chargecodeKey": true,
            "productcatalogDescription": true,
            "bundleOfferCaption": false,
            "expectedAmount": false
        },
        "candidate_source_priority": "chargecode_description",
        "recommended_decision": "include",
        "source_tools": [
            "POST /agent-tools/billing/candidate-discovery",
            "POST /agent-tools/billing/candidate-clusters",
            "POST /agent-tools/invoices/sample-lines"
        ],
        "observacoes": "Contexto observado pelo agente."
    }
}
```

Campos indisponiveis podem ser arrays vazios ou `null`, mas `billing_context_json` deve existir para todo candidato. Nao crie candidatos a partir de valor esperado, preco ou janelas de valor.

## Checks Finais Obrigatorios

Antes de retornar:

1. Toda regra financeira deve ter `source_claim_id` apontando para uma declaracao monetaria do dossie.
2. Nenhuma regra financeira pode existir apenas porque uma tool retornou um candidato de billing. Candidatos qualificam uma regra existente; nao criam regra nova.
3. Toda regra financeira deve ter evidencia.
4. Todo dossie, regra financeira e item monetario mapeado deve ter `approval_status: "approved" | "not_approved"` e `approval_evidence`.
5. Use `approved` somente com GO/aprovacao/decisao de seguir explicita no dossie. Use `not_approved` para NOGO, cancelado, sem atuacao, nao aprovado, nao iniciado, ainda em discussao ou ausencia de aprovacao explicita.
6. Regra `not_approved` nao deve ter predicado final ativo nem ser apresentada como apta a gerar impacto financeiro.
7. Toda regra financeira deve ter `financialImpact.monetary: true` dentro de `rule_draft_json`.
8. Toda regra financeira deve ter `calculation.kind`, `calculation.requiredColumns` e `support.confrontabilityStatus`.
9. Toda regra financeira deve ter `situacao_regra`, `dependency_codes`, `rule_set_json`, `rule_relationship_json` e os campos equivalentes dentro de `rule_draft_json`.
10. `situacao_regra` deve ser apenas `executable`, `needs_review` ou `not_applicable`; status tecnicos ficam em `dependency_codes` e `support_json`.
11. CRM, bundle CRM e elegibilidade de bundle devem aparecer em `external_conditions_json` quando forem necessarios para aplicar ou desambiguar a regra, mas nunca devem criar preco/formula sem declaracao monetaria do dossie. IDs CRM sao opcionais: use arrays vazios quando ausentes e registre `required_crm_checks`.
12. Toda regra `confrontable_deterministic` deve ter predicado executavel.
13. Toda regra financeira confrontavel deve carregar a vigencia do dossie em `valid_from`/`valid_to` no envelope e em `effectiveFrom`/`effectiveTo` dentro de `rule_draft_json`. Use `null` para data fim ausente.
14. Toda regra financeira com gratuidade/desconto/janela relativa a contratacao ou ativacao deve carregar `eligibilityWindow` dentro de `rule_draft_json`; se usar campo espelho, preencha tambem `eligibility_window_json`.
15. Todo predicado monetario final deve incluir `chargecodeKeyIn` ou `billingLineIdentityIn[].chargecodeKey`.
16. Todo candidato amplo deve estar como `include`, `exclude` ou `pending`.
17. Todo candidato deve preservar `candidate_source_priority`, `line_role`, `billing_context_json`, `matched_on`, sinais positivos/negativos e racional.
18. Toda regra deve preservar `chargecode_candidates_json`, `disambiguation_json`, `stacking_json` e `required_crm_checks` quando esses dados forem conhecidos ou quando faltarem dados externos.
19. Toda declaracao monetaria sem suporte deterministico deve aparecer em `itens_mapeados_nao_suportados` ou em `regras_financeiras` com `status_confrontabilidade` nao deterministico.
20. Todo item em `itens_mapeados_nao_suportados` derivado de declaracao monetaria deve preservar `source_claim_id`.
21. Toda falha de tool deve aparecer em `status: "blocked"` ou `perguntas_abertas_globais`.
22. Nenhum campo deve conter o bearer token.
