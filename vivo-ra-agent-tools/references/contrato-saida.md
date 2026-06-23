# Contrato de Saida

Retorne um unico objeto JSON. Nao inclua prosa fora do JSON.

Nao crie arquivo para a resposta final. Nao use `Write`, nao salve `/home/user/result.json` e nao devolva anexo. O objeto deve ser a resposta final do step para que o workflow capture o output format estruturado.

```json
{
  "status": "draft | needs_mapping | needs_agent_audit | needs_review | blocked | completed",
  "dossier": {
    "fileName": "00000.pdf",
    "dossierCode": "00000",
    "title": "Produto Exemplo - Gratuito"
  },
  "summary": {
    "financialRuleCount": 1,
    "nonConfrontableCount": 2,
    "candidateSetCount": 4,
    "finalPredicateCount": 1,
    "openQuestionCount": 1,
    "narrative": "Resumo em portugues brasileiro."
  },
  "financialRules": [
    {
      "ruleDraft": {},
      "confrontabilityStatus": "confrontable | needs_mapping | needs_agent_audit | needs_review | not_applicable",
      "candidateSets": [],
      "candidateQualification": {
        "status": "agent_qualified_needs_human_confirmation",
        "included": [],
        "excluded": [],
        "pending": [],
        "finalPredicate": {},
        "rationale": "Racional em portugues brasileiro."
      },
      "qualificationValidation": {},
      "ruleValidation": {},
      "conflicts": [],
      "evidence": [],
      "assumptions": [],
      "openQuestions": []
    }
  ],
  "nonConfrontableItems": [
    {
      "title": "",
      "reason": "",
      "evidence": []
    }
  ],
  "toolTrace": [
    {
      "tool": "POST /agent-tools/billing/candidate-discovery",
      "purpose": "Encontrar candidatos amplos para Produto Exemplo",
      "resultSummary": "Resumo em portugues brasileiro."
    }
  ],
  "globalOpenQuestions": []
}
```

## Formato Obrigatorio de `candidate_sets_resumo`

Cada item de `regras_financeiras[].candidate_sets_resumo[]` deve preservar o contexto de billing usado pelo agente para ponderar o candidato. Nao retorne apenas id, decisao e motivo.

```json
{
  "candidate_set_id": "cand-chargecode-RMEXEMPLO001",
  "decisao": "include | exclude | pending",
  "motivo": "Racional curto em portugues brasileiro.",
  "billing_context": {
    "candidate_set_kind": "chargecode_key | billing_line_identity | semantic_description_match | bundle_neighbor",
    "predicate": {
      "chargecodeKeyIn": ["RMEXEMPLO001"]
    },
    "chargecode_keys": ["RMEXEMPLO001"],
    "productcatalog_keys": ["1234567890"],
    "productcatalog_descriptions": ["Produto Exemplo"],
    "bundle_offer_captions": ["BUNDLE EXEMPLO"],
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
    "observacoes": "Resumo do contexto de fatura visto pelo agente."
  }
}
```

Campos que nao existirem nas tools podem ser arrays vazios ou `null`, mas o objeto `billing_context` deve existir em todos os candidatos. Nao crie candidatos por valor esperado ou janela de preco. Para candidatos sem linha encontrada por descricao/chargecode, use contagens `0` e explique em `observacoes`.

## Checklist Final

Antes de responder:

1. Toda regra financeira tem evidencia.
2. Toda regra financeira tem `financialImpact.monetary: true`.
3. Toda regra confrontavel tem predicado executavel.
4. Toda regra monetaria final tem `chargecodeKeyIn` ou explica por que nao.
5. Todo candidato amplo foi incluido, excluido ou deixado pendente.
6. Todo texto livre esta em portugues brasileiro.
7. Toda falha de tool aparece em `status`, `toolTrace` ou `globalOpenQuestions`.
8. Todo `candidate_sets_resumo[]` tem `billing_context` com product descriptions, product keys, bundle captions e charge codes disponiveis nas tools.
