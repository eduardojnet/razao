# ADR-0019: Sugestão da conta de contrapartida por IA

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-01, RNF-11, CAP-14

## Contexto

Com partidas dobradas, o operador precisa escolher a conta de contrapartida em cada lançamento. Isso é atrito no caixa.

## Decisão

Decidimos oferecer uma sugestão da conta de contrapartida, feita por um modelo de linguagem (Gemini, via Vertex AI), dentro do serviço Plano de Contas e chamada antes do registro.

Regras:

- a sugestão nunca grava lançamento; o operador confirma;
- a sugestão fica fora do caminho de registro; se o modelo estiver indisponível, a escolha é manual e o registro segue;
- a resposta só pode ser uma conta analítica existente no plano do comerciante;
- a qualidade é medida pela taxa de aceite da sugestão.

[NVI] A disponibilidade do modelo na região São Paulo e o custo por sugestão precisam ser confirmados. O que é enviado ao modelo é tratado na avaliação LGPD.

## Alternativas descartadas

- Detecção de lançamento atípico: descartei porque, sem histórico, aprende pouco.
- Consulta do consolidado em linguagem natural: descartei por ser o candidato mais forte a enfeite.
- Nenhum uso de IA no produto: descartei por decisão de produto.

## Consequências aceitas

Ganhamos menos atrito na escolha da contrapartida. Pagamos custo por chamada e o cuidado com dado pessoal na descrição. A implementação é condicional no G2 (ADR-0011).

## Como provamos

Teste com o modelo indisponível: a sugestão responde erro e o registro manual segue com sucesso (teste previsto, condicional).
