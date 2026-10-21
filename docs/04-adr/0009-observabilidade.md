# ADR-0009: OpenTelemetry com Cloud Operations

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-13, US-14, SLO-01 a SLO-04

## Contexto

Os NFRs só valem se forem medidos. O PO pede métricas técnicas e de negócio, logs estruturados e rastreamento que siga o lançamento até a atualização do saldo, inclusive no trecho assíncrono.

## Decisão

Decidimos instrumentar os serviços com OpenTelemetry. No ambiente alvo, o destino é o Cloud Operations (Trace, Monitoring e Logging); no ambiente local, Grafana, Prometheus e Tempo. O ID de correlação viaja na requisição e no cabeçalho do evento.

Descrição do lançamento, credenciais e tokens nunca vão para log.

## Alternativas descartadas

- OpenTelemetry com Grafana também em produção: descartei porque obriga a operar Prometheus, Loki e Tempo no cluster.
- Cloud Operations sem OpenTelemetry: descartei porque amarra o código ao provedor e deixa o ambiente local sem rastreamento.

## Consequências aceitas

Ganhamos troca de destino sem tocar no código. Pagamos dois conjuntos de painéis, local e alvo.

## Como provamos

Teste que registra um lançamento e encontra o mesmo ID de correlação na requisição, no evento e na atualização do saldo (teste previsto).
