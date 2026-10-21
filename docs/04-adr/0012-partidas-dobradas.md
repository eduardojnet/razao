# ADR-0012: Escrituração por partidas dobradas com plano de contas

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RN-01, RN-06, RN-19, RN-21 a RN-24, I-03, I-08, I-09

## Contexto

O handoff original registrava cada lançamento só na conta caixa, com plano de contas fora de escopo. Um sistema que se apresenta como livro razão, avaliado por um banco, fica mais defensável quando cada lançamento tem contrapartida.

## Decisão

Decidi adotar partidas dobradas com plano de contas no MVP. Todo lançamento tem duas partidas de mesmo valor, uma a débito e uma a crédito, e uma delas é sempre a conta Caixa. O plano tem grupos fixos (Ativo, Patrimônio Líquido, Receitas, Despesas), e o comerciante cria contas analíticas dentro deles. Crédito e débito passam a ter só o sentido contábil, e a API recebe a conta de débito e a conta de crédito.

As regras estão no registro de alterações do handoff, de 20/10/2026.

## Alternativas descartadas

- Conta caixa única, como no handoff original: descartei porque não fecha balancete e não mostra contrapartida.
- Plano fixo mínimo, sem contas do comerciante: descartei como alvo, mas é o primeiro recuo se o prazo não comportar.
- Lançamento com mais de duas partidas: descartei porque complica a I-08 e o estorno sem ganho para o caixa de um comerciante.

## Consequências aceitas

Ganhamos escrituração real e balancete. Pagamos um terceiro serviço, a réplica local do plano (ADR-0018) e mais validação no registro. O consolidado diário continua sendo a projeção da conta Caixa, e o teste de referência continua valendo. Critério de recuo no G1 e no G2 registrado no handoff.

## Como provamos

Teste que recusa lançamento com partidas desbalanceadas (I-08) e partida em conta sintética (I-09); teste de referência com saldo final de R$ 1.650,00 (testes previstos).
