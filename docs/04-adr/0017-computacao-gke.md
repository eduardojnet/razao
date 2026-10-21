# ADR-0017: GKE Autopilot no ambiente alvo

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-08, RNF-09

## Contexto

Os serviços têm APIs e consumidores de eventos que precisam rodar continuamente. O Lançamentos precisa de mais de uma instância para não ter ponto único de falha.

## Decisão

Decidimos pelo GKE Autopilot, na região São Paulo. APIs e consumidores rodam no mesmo cluster, sem estado local. O Lançamentos tem no mínimo duas réplicas.

## Alternativas descartadas

- Cloud Run: descartei porque os consumidores contínuos exigiriam instância mínima sempre ligada, combinação mais natural com Pub/Sub.
- Cloud Run nas APIs e GKE nos consumidores: descartei porque seriam duas plataformas para explicar, operar e orçar.

## Consequências aceitas

Ganhamos uma plataforma única para todos os componentes, inclusive Kong e Keycloak. Pagamos mais operação que no Cloud Run.

## Como provamos

O Terraform do ambiente alvo declara o cluster e as réplicas (ADR-0021). Não há ambiente vivo; a limitação está declarada no README.
