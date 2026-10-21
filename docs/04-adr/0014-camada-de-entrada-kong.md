# ADR-0014: Kong OSS sem banco na camada de entrada

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-10, US-17

## Contexto

O PO exige autenticação, limite de requisições por integrador e validação estrita na fronteira entre integrador e API. O avaliador precisa conseguir rodar a mesma entrada no ambiente local.

## Decisão

Decidimos pelo Kong OSS em modo declarativo, sem banco, igual no Docker Compose e no GKE. Toda chamada externa entra por ele. O Kong valida o token do Keycloak e aplica o limite por integrador, com a contagem no Redis, compartilhada entre réplicas. Integrador acima do limite recebe 429; os demais seguem atendidos.

## Alternativas descartadas

- Apigee: descartei pelo custo e porque não roda no ambiente local.
- Traefik e NGINX: descartei porque a contagem do limite não é compartilhada entre réplicas.
- Envoy com serviço de limite: descartei porque são dois componentes a operar para o mesmo efeito.

## Consequências aceitas

Ganhamos a mesma entrada nos dois ambientes e prova local do limite. Pagamos mais um componente no Docker Compose. A implementação do limite é condicional no G2 (ADR-0011).

## Como provamos

Teste em que um integrador acima do limite recebe 429 enquanto outro recebe 200 (teste previsto, condicional).
