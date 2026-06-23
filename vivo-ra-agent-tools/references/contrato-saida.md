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

## Checklist Final

Antes de responder:

1. Toda regra financeira tem evidencia.
2. Toda regra financeira tem `financialImpact.monetary: true`.
3. Toda regra confrontavel tem predicado executavel.
4. Toda regra monetaria final tem `chargecodeKeyIn` ou explica por que nao.
5. Todo candidato amplo foi incluido, excluido ou deixado pendente.
6. Todo texto livre esta em portugues brasileiro.
7. Toda falha de tool aparece em `status`, `toolTrace` ou `globalOpenQuestions`.
