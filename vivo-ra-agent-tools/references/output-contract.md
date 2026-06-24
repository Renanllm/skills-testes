# Contrato de Saida

Retorne um unico objeto JSON. Nao inclua texto fora do JSON.

O envelope externo deve usar os nomes de campo em portugues declarados no output format do workflow. As chaves tecnicas da DSL v0.3 devem ficar preservadas dentro dos campos JSON serializados, como `rule_draft_json`, `calculation_json`, `support_json` e `predicado_final_json`.

```json
{
    "status": "draft | needs_mapping | needs_agent_qualification | needs_review | blocked | completed",
    "dossie": {
        "arquivo": "20131.pdf",
        "codigo": "20131",
        "titulo": "Titulo inferido do dossie"
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
            "nome_regra": "",
            "tipo_regra": "fixed_price | no_charge | discount | usage_tariff | prorata | bundle_composition | presence_rule | free_period | exclusion | migration_price | other_monetary",
            "status_confrontabilidade": "confrontable_deterministic | confrontable_after_mapping | needs_agent_qualification | needs_mapping | needs_usage_quantity | needs_reference_price | needs_crm | needs_subscription_event | needs_entitlement_inventory | needs_review | not_supported_yet | blocked",
            "alvo_nome": "",
            "alvo_tipo": "product | variant | bundle | plan | offer | unknown",
            "escopo_afetado": "single_charge_line | product | bundle | plan | account | unknown",
            "campo_monetario": "c.chargetotalamount",
            "comportamento_esperado": "",
            "calculation_kind": "fixed_amount | no_charge | discount_amount | discount_percent | usage_event_tariff | usage_duration_tariff | usage_volume_tariff | daily_rate | prorata_by_period | bundle_component_sum | forbidden_charge | presence_required | custom_logic",
            "required_columns": ["charge_total_amount"],
            "valor_esperado": 0,
            "rule_draft_json": "{\"ruleName\":\"...\",\"calculation\":{\"kind\":\"no_charge\"}}",
            "calculation_json": "{\"kind\":\"no_charge\",\"amountField\":\"charge_total_amount\"}",
            "support_json": "{\"confrontabilityStatus\":\"confrontable_deterministic\",\"unsupportedReasons\":[]}",
            "predicado_final_json": "{\"chargecodeKeyIn\":[\"RMVIVORECADM\"]}",
            "candidate_sets_resumo": [
                {
                    "candidate_set_id": "cand-chargecode-RMEXAMPLE001",
                    "decisao": "include | exclude | pending",
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
            "titulo": "",
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
    "motivo": "Racional curto em portugues.",
    "billing_context_json": "{\"candidate_set_kind\":\"chargecode_key\",\"predicate\":{\"chargecodeKeyIn\":[\"RMEXAMPLE001\"]},\"chargecode_keys\":[\"RMEXAMPLE001\"],\"productcatalog_keys\":[\"1234567890\"],\"productcatalog_descriptions\":[\"Example Product\"],\"bundle_offer_captions\":[\"EXAMPLE BUNDLE\"],\"line_count\":10,\"invoice_count\":8,\"customer_count\":8,\"net_amount\":239,\"positive_signals\":[\"chargecode_token_match\"],\"negative_signals\":[],\"recommended_decision\":\"include\",\"source_tools\":[\"POST /agent-tools/billing/candidate-discovery\"]}"
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
        "chargecodeKey": true,
        "productcatalogDescription": true,
        "bundleOfferCaption": false
    },
    "recommended_decision": "include",
    "line_role_suggestion": "direct_product_charge | discount | different_variant | plan_with_benefit | context_only | unknown",
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
            "chargecodeKey": true,
            "productcatalogDescription": true,
            "bundleOfferCaption": false
        },
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

1. Toda regra financeira deve ter evidencia.
2. Toda regra financeira deve ter `financialImpact.monetary: true` dentro de `rule_draft_json`.
3. Toda regra financeira deve ter `calculation.kind`, `calculation.requiredColumns` e `support.confrontabilityStatus`.
4. Toda regra `confrontable_deterministic` deve ter predicado executavel.
5. Todo predicado monetario final deve incluir `chargecodeKeyIn` ou `billingLineIdentityIn[].chargecodeKey`.
6. Todo candidato amplo deve estar como `include`, `exclude` ou `pending`.
7. Toda declaracao monetaria sem suporte deterministico deve aparecer em `itens_mapeados_nao_suportados` ou em `regras_financeiras` com `status_confrontabilidade` nao deterministico.
8. Toda falha de tool deve aparecer em `status: "blocked"` ou `perguntas_abertas_globais`.
9. Nenhum campo deve conter o bearer token.
