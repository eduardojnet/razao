# ADR-0021: Terraform do ambiente alvo, escrito sem aplicar

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-09

## Contexto

O desenho alvo na GCP precisa de prova além do diagrama, e a estimativa de custo precisa de uma lista exata de recursos.

## Decisão

Decidimos escrever o Terraform do ambiente alvo sem aplicá-lo: GKE Autopilot, três Cloud SQL, Memorystore, Kafka gerenciado, Secret Manager e Vertex AI, na região São Paulo.

## Alternativas descartadas

- Aplicar em projeto GCP real: descartei pelo custo de nuvem, pelas credenciais e pelo tempo de depuração no prazo.
- Só diagrama: descartei porque deixa o desenho alvo sem prova executável.

## Consequências aceitas

Ganhamos prova do desenho e base exata para a estimativa. Pagamos a limitação, declarada no README: não há ambiente vivo na nuvem.

## Como provamos

`terraform validate` sem erro no pipeline, sem nenhuma credencial no repositório (previsto).
