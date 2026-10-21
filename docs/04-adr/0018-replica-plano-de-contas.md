# ADR-0018: Cópia local do plano de contas no Lançamentos

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-01, I-09

## Contexto

Com o plano de contas num serviço próprio (ADR-0001), o Lançamentos precisa validar cada partida contra o plano (I-09). Uma chamada síncrona faria o caixa parar quando o Plano de Contas caísse.

## Decisão

Decidimos que o Plano de Contas publica `PlanoDeContasAlterado` a cada criação, alteração ou inativação de conta, e que o Lançamentos mantém uma cópia local do plano, alimentada por esses eventos. A validação das partidas usa só a cópia local.

## Alternativas descartadas

- Chamada síncrona com cache e disjuntor: descartei porque, com o cache frio, o registro falha.
- Chamada síncrona simples: descartei porque o caixa passaria a depender de dois serviços.

## Consequências aceitas

Ganhamos registro independente do Plano de Contas. Pagamos um atraso de segundos entre criar uma conta e poder lançar nela.

## Como provamos

Teste de queda: com o Plano de Contas parado, 100 lançamentos são aceitos, validados pela cópia local (teste previsto).
