# Itens Nao Confrontaveis

Nao crie regras de billing para declaracoes operacionais sem efeito direto em fatura.

## Normalmente Nao Confrontaveis

- ativar/desativar IDs em PRM, CRM ou sistemas internos;
- tarefas de migracao, implantacao, aprovacao ou manutencao;
- comunicacao ao cliente, FAQ, material comercial ou orientacao de canal;
- objetivo da campanha sem valor esperado;
- manutencao de catalogo sem regra de cobranca;
- instrucao de suporte sem credito, desconto ou cobranca.

## Potencialmente Confrontaveis

Somente vire regra quando houver efeito de fatura:

- produto gratis por periodo ou permanentemente;
- preco fixo;
- desconto;
- produto nao deve ser cobrado dentro de bundle;
- preco antigo/novo apos data;
- cobranca condicionada a elegibilidade;
- regra cancela ou sobrepoe outra.

## Formato

```json
{
  "title": "IDs de Produto Exemplo ficam Offline no PRM",
  "reason": "Instrucao operacional; nao define valor esperado em fatura.",
  "evidence": [
    {
      "source": "00000.pdf",
      "page": 3,
      "quote": "trecho curto"
    }
  ]
}
```
