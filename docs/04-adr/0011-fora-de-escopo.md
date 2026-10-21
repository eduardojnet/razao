# ADR-0011: O que fica de fora da entrega de sexta

Status: proposta, atualizada no G2
Data: 2026-10-20
Regras e requisitos afetados: US-08, US-12, US-13, US-17

## Contexto

O prazo é sexta, 23/10, às 08:00. O handoff manda cortar de baixo para cima e nunca a documentação.

## Decisão

Entram como condicionais no G2, na ordem abaixo, cada um só depois do anterior com testes verdes:

1. conciliação automática (US-13);
2. limite de requisições por integrador (US-17);
3. lançamento com data passada pelo comerciante (US-08);
4. sugestão de contrapartida (ADR-0019);
5. reconstrução do consolidado (US-12).

As partidas dobradas seguem o critério de recuo registrado no handoff: plano completo, depois plano fixo mínimo, depois conta Caixa única.

Ficam fora, como evolução: fechamento de dia, conciliação bancária, multimoeda, lançamentos agendados, exportação para a contabilidade, aplicativo móvel e lançamento sem conta Caixa.

## Alternativas descartadas

- Entregar tudo: descartei porque o maior risco do projeto é deixar um obrigatório aberto por excesso de escopo.

## Consequências aceitas

Ganhamos foco nos obrigatórios e na prova dos NFRs. Pagamos diferenciais que podem ficar só no desenho. Mesmo cortado o código, a RN-07 e a RN-15 continuam valendo no desenho.

## Como provamos

O G2 registra, nesta ADR, o que entrou e o que foi cortado, com o link dos testes verdes.
