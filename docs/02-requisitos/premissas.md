# Premissas

Toda meta que não está no enunciado é premissa. Cada uma tem dono, motivo e o teste que vai medi-la. Se a medição contrariar a premissa, o valor medido vai para uma ADR e substitui o proposto.

## Premissas do PO revisadas pelo Arquiteto

Revisão de Eduardo José Gonçalves do Carmo, 20/10/2026.

| Premissa | Proposta do PO | Posição | Motivo | Como medimos |
| --- | --- | --- | --- | --- |
| Atraso de consolidação | p95 <= 5 s | Aceita | O outbox publica em ciclo curto e o consumidor aplica um evento por vez por comerciante; 5 s comportam os dois com folga em operação normal | Atraso entre data de registro e atualização do saldo, em percentis |
| Vazão de escrita em pico | 50 escritas/s com folga de 2x | Aceita | Não há número no enunciado; espelhar a leitura com folga evita subdimensionar o caixa | Teste de carga de registro a 100 escritas/s |
| Janela de lançamento retroativo | 90 dias | Aceita | Contém o custo de recalcular dias seguintes sem impedir a venda esquecida comum | Teste de lançamento além da janela devolvendo 422 |
| Período máximo do extrato | 366 dias | Aceita | Cobre um exercício completo, inclusive ano bissexto | Teste de extrato com 367 dias devolvendo 422 |
| Latência de registro | p95 <= 300 ms | Aceita | O registro grava lançamento e outbox numa única transação, sem chamada a outro serviço | Teste de carga de registro |
| Latência de consulta do consolidado | p95 <= 200 ms | Aceita | A leitura vem da projeção pré-calculada, com cache na frente | Teste de carga de 50 leituras/s |
| Legado atual | Planilha | Aceita | É o cenário mais comum de pequeno comerciante e permite importar histórico em CSV | Desenho de transição |

## Premissas do Arquiteto

Nenhuma ainda. Cada nova premissa entra aqui com a data e o nome de quem decidiu.
