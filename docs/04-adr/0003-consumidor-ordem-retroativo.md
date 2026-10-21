# ADR-0003: Consumidor imune à ordem e à duplicidade, com recálculo de dia passado

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RF-06, RF-09, RN-07, RN-15, C-01, C-02, C-04

## Contexto

Os eventos chegam pelo menos uma vez e podem chegar fora de ordem. O estorno de um lançamento de ontem pesa em ontem (RN-15), e esse recálculo está no Must (premissa P-01). O consolidado diário precisa fechar igual em qualquer ordem de chegada.

## Decisão

Decidimos que o Consolidado aplica cada evento numa única transação que:

1. registra o identificador do evento numa tabela de eventos aplicados, com chave única; se já existir, o evento é ignorado (C-04);
2. soma o valor da partida em Caixa ao total de débitos ou de créditos do dia de competência;
3. ajusta o saldo inicial e o saldo final de todos os dias seguintes já materializados, pela mesma diferença.

Somar é comutativo: a ordem dos eventos não muda o resultado. O lançamento com data passada e o estorno herdando a competência usam o mesmo caminho. Como a atualização é atômica, o comerciante nunca vê um dia recalculado pela metade; a hora da última atualização mostra até onde o saldo foi aplicado (RN-14).

## Alternativas descartadas

- Recalcular do zero o período a cada evento: descartei porque o custo cresce com o histórico.
- Exigir ordem estrita por comerciante e processar em fila única: descartei como garantia principal, porque um evento fora de ordem travaria os seguintes. A chave de partição por comerciante (ADR-0016) continua reduzindo a desordem.
- Calcular o saldo acumulado só na leitura: descartei porque a soma do histórico em cada consulta compete com as 50 leituras por segundo.

## Consequências aceitas

Ganhamos um consumidor que fecha igual em qualquer ordem. Pagamos escrita em vários dias quando chega um evento antigo, limitada pela janela de 90 dias (RN-07) e pelo estorno de lançamento antigo. A situação "em recálculo" proposta no handoff deixa de ser necessária.

## Como provamos

Teste de referência do handoff com os eventos dos fatos 6 e 7 em ordem invertida, com o evento 6 entregue duas vezes e com o Consolidado parado durante os dois; nos três casos, saldo final de R$ 1.650,00 em 22/10 (teste previsto).
