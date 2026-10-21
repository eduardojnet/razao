# ADR-0010: Estratégia de testes, com Locust na carga

Status: aceita para o Locust; proposta para as demais ferramentas
Data: 2026-10-20
Regras e requisitos afetados: RNF-14, US-10, US-11

## Contexto

O enunciado exige testes automatizados. O PO exige prova executável dos dois NFRs: a queda do consolidado e a carga de 50 leituras por segundo.

## Decisão

| Tipo | O que prova | Ferramenta |
| --- | --- | --- |
| Unitário de domínio | RN, I e C, sem infraestrutura | pytest (proposta) |
| Integração | API, banco e mensageria reais | pytest com contêineres descartáveis (proposta) |
| Contrato | Compatibilidade de API e eventos | Validação dos esquemas OpenAPI e de evento (proposta) |
| Resiliência | NFR 1 | pytest parando contêineres do Docker Compose (proposta) |
| Carga | NFR 2 | Locust (aceita) |

Todo teste de regra leva o código no nome, por exemplo `test_i04_estorno_unico_sob_concorrencia`. Os roteiros e os resultados ficam em `testes/carga` e `testes/resiliencia`.

## Alternativas descartadas

- k6 na carga: descartei por decisão minha de manter o roteiro na mesma linguagem do projeto.
- Prova manual dos NFRs: descartei porque não é reproduzível pelo avaliador.

## Consequências aceitas

Ganhamos prova que o avaliador roda. Pagamos o relatório do Locust, menos direto que o do k6, para anexar.

## Como provamos

Esta ADR é provada pela própria suíte: um único comando roda os testes unitários e de integração (previsto).
