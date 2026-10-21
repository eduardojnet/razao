# ADR-0015: Arquitetura hexagonal dentro de cada serviço

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-14

## Contexto

As regras de negócio precisam ser testadas sem banco nem mensageria, e trocar uma peça de infraestrutura não pode tocar uma regra.

## Decisão

Decidimos pela arquitetura hexagonal em cada serviço. O domínio não importa FastAPI, driver de banco nem cliente de mensageria. As portas são classes abstratas; os adaptadores (PostgreSQL, Kafka, Redis, Keycloak e o modelo da sugestão de contrapartida) são implementações.

## Alternativas descartadas

- Clean Architecture: mesma ideia, com mais camadas e mais cerimônia para serviços pequenos.
- Camadas tradicionais (controlador, serviço, repositório): descartei porque a regra tende a vazar para o banco e para o framework.

## Consequências aceitas

Ganhamos regras testáveis sem infraestrutura e adaptadores trocáveis. Pagamos portas e adaptadores a mais para escrever.

## Como provamos

Os testes unitários de domínio rodam sem nenhum contêiner (previsto).
