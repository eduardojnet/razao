# ADR-0013: Retentativa, tempo limite e fila de falhas na integração por eventos

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RF-05, RF-06, C-04

## Contexto

O handoff pede métrica e alerta para evento rejeitado pelo consumidor e um procedimento para reprocessar sem duplicar.

## Decisão

Decidimos:

- o consumidor tenta aplicar o evento até 5 vezes, com espera crescente de 1, 2, 4, 8 e 16 segundos;
- toda chamada a banco e mensageria tem tempo limite explícito;
- depois da última tentativa, o evento vai para um tópico de falhas, e qualquer evento ali dispara alerta;
- o reprocessamento lê o tópico de falhas e reaplica pelo mesmo caminho, protegido pela tabela de eventos aplicados (ADR-0003), sem duplicar saldo.

Os números são confirmados no teste de resiliência; se a medição pedir outros, eles entram aqui.

## Alternativas descartadas

- Retentar indefinidamente: descartei porque um evento com defeito travaria os seguintes da mesma partição.
- Descartar o evento após falhas: descartei porque é perda de lançamento no consolidado.

## Consequências aceitas

Ganhamos isolamento do evento com defeito sem perder nenhum. Pagamos um procedimento manual de reprocessamento, descrito no runbook de fila de falhas.

## Como provamos

Teste que injeta um evento que falha, confirma que ele chega ao tópico de falhas, corrige a causa, reprocessa e mostra o saldo aplicado uma única vez (teste previsto).
