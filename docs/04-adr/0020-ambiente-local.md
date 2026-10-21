# ADR-0020: Ambiente local em Docker Compose

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-15

## Contexto

O avaliador precisa subir tudo com um comando, numa máquina que eu não controlo.

## Decisão

Decidimos pelo Docker Compose, com versões fixadas, subindo os três serviços, três PostgreSQL, Redis, Kafka, Keycloak, Kong e a pilha de observabilidade local.

Diferenças em relação ao ambiente alvo:

| Local | Alvo |
| --- | --- |
| Kafka em contêiner | Kafka gerenciado |
| Grafana, Prometheus e Tempo | Cloud Operations |
| Sugestão de contrapartida desligada por padrão | Vertex AI |
| Sem redundância | Réplicas e failover |

Kong e Keycloak são iguais nos dois.

## Alternativas descartadas

- Kind com os mesmos manifestos do GKE: descartei porque é pesado na máquina do avaliador e aumenta a chance de o ambiente não subir.

## Consequências aceitas

Ganhamos um ambiente que sobe com um comando. Pagamos a ausência de prova local da redundância, declarada no README.

## Como provamos

Validação em máquina limpa antes da entrega: clone, um comando e testes verdes (previsto).
