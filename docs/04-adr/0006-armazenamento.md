# ADR-0006: PostgreSQL por serviço e Redis na leitura do consolidado

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-03, RNF-09, RNF-10, I-04, I-07

## Contexto

Cada serviço é dono do seu dado, e o banco de um não pode ser acessível ao outro. Duas invariantes precisam valer sob concorrência: estorno único (I-04) e chave de idempotência única (I-07).

## Decisão

Decidimos por um PostgreSQL por serviço, com credencial própria e menor privilégio; no ambiente alvo, Cloud SQL. O Redis (Memorystore no ambiente alvo) fica só na leitura do consolidado (ADR-0004).

As invariantes I-04 e I-07 ficam presas em índice único no banco, não só em código: duas requisições simultâneas não conseguem gravar o segundo estorno.

Separo escrita e leitura por serviço: o Lançamentos é o modelo de escrita, e o Consolidado é o modelo de leitura derivado. É CQRS na fronteira entre serviços, sem dois modelos dentro do mesmo serviço.

## Alternativas descartadas

- PostgreSQL sem cache: fica como plano B da ADR-0004, decidido pelo teste de carga.
- AlloyDB: descartei pelo custo mensal para o volume do MVP.
- Spanner: descartei pelo custo e porque não há emulador fiel para o avaliador rodar localmente.

## Consequências aceitas

Ganhamos isolamento de dado e invariantes garantidas pelo banco. Pagamos três bancos para operar e fazer backup. O Consolidado continua reconstruível a partir do Lançamentos.

## Como provamos

Teste de concorrência: duas requisições paralelas de estorno do mesmo lançamento geram um estorno e uma resposta 409 (teste previsto).
