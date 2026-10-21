# ADR-0004: Leitura do consolidado a partir da projeção pré-calculada, com cache

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-02, RNF-05, SLO-02

## Contexto

O consolidado diário precisa sustentar 50 leituras por segundo com pelo menos 95% de sucesso, e a meta interna é 99%, com p95 de 200 ms.

## Decisão

Decidimos servir a leitura da projeção já calculada no banco do Consolidado (ADR-0003): a consulta de um dia é a leitura de uma linha pela chave comerciante e data. Na frente dela, um cache Redis por comerciante e dia, invalidado na mesma operação que aplica o evento.

Com o cache frio, a leitura vai direto ao banco pela chave primária, que é o mesmo caminho de antes do cache existir. O cache reduz carga; não é condição para a leitura funcionar.

O dimensionamento sai do teste: a capacidade por instância é medida no teste de carga e só então vira número nesta ADR.

## Alternativas descartadas

- Somente banco, sem cache: fica como plano B, e o próprio teste de carga decide se o cache é necessário.
- Réplica de leitura do banco: descartei para o MVP porque acrescenta atraso de replicação ao atraso de consolidação.
- Calcular na leitura a partir dos lançamentos: descartei porque transfere o custo para a hora do pico.

## Consequências aceitas

Ganhamos leitura de custo constante por dia consultado. Pagamos a invalidação do cache a cada evento aplicado. O primeiro gargalo esperado acima de 50 leituras por segundo é o número de conexões do banco do Consolidado, a confirmar no teste.

## Como provamos

Teste de carga em `testes/carga`, 50 leituras por segundo durante 5 minutos, com cache frio no início; relatório com taxa de sucesso e p95 (teste previsto).
