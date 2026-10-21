# ADR-0016: Kafka como mensageria, com partição por comerciante

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RF-05, RF-06, C-04

## Contexto

O PO exige que um consumidor novo, como uma apuração fiscal, consiga reprocessar o histórico de eventos sem envolver o serviço de Lançamentos, e que a ordem por comerciante seja preservada ou o consumidor seja imune a ela.

## Decisão

Decidimos pelo Kafka: gerenciado no ambiente alvo, em contêiner no ambiente local. Os tópicos são `LancamentoRegistrado` e `PlanoDeContasAlterado`. A chave de partição é o comerciante, o que mantém a ordem dos eventos de um mesmo comerciante. A retenção dos tópicos permite que um consumidor novo leia o histórico desde o início.

[NVI] Os limites de retenção e de partições do serviço gerenciado na GCP precisam ser confirmados na documentação oficial antes da estimativa de custo.

## Alternativas descartadas

- Pub/Sub com arquivo de eventos: descartei porque a retenção curta obrigaria a arquivar eventos fora da mensageria para permitir o reprocessamento.
- RabbitMQ: descartei porque não oferece reprocessamento do histórico.

## Consequências aceitas

Ganhamos ordem por comerciante e reprocessamento nativo. Pagamos um cluster gerenciado cobrado mesmo ocioso.

## Como provamos

Teste em que um consumidor novo lê o tópico desde o início e reconstrói o consolidado igual ao atual (teste previsto, ligado à US-12).
