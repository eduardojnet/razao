# ADR-0002: Outbox transacional entre gravação e publicação

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RF-05, RNF-03

## Contexto

O lançamento é gravado no banco do serviço e o evento vai para a mensageria. São dois sistemas. Se o processo cair entre as duas escritas, um lançamento confirmado pode ficar sem evento, e o consolidado diário nunca o veria.

## Decisão

Decidimos gravar o evento numa tabela de saída (outbox) na mesma transação do lançamento. Um publicador lê essa tabela e envia para a mensageria, marcando o que foi enviado. A regra vale para o Lançamentos e para o Plano de Contas.

Se o processo cair depois da transação e antes do envio, o evento continua gravado e sai quando o publicador voltar. A entrega é pelo menos uma vez; o consumidor elimina duplicidade (ADR-0003).

## Alternativas descartadas

- Captura de alterações do banco: descartei porque acrescenta uma peça de infraestrutura para operar, sem ganho para o volume do MVP.
- Event sourcing, com o evento como fonte da verdade: descartei porque muda o modelo inteiro e a forma de consultar lançamentos, com custo alto no prazo.
- Publicar direto na mensageria após o commit: descartei porque é exatamente o caso que perde evento.

## Consequências aceitas

Ganhamos zero perda de evento confirmado sem transação distribuída. Pagamos um atraso de publicação igual ao ciclo do publicador e a tabela de saída para manter. Para o auditor, a prova é que todo lançamento tem um evento na tabela de saída, com a marca de envio.

## Como provamos

Teste que interrompe o publicador entre gravar e publicar e mostra o evento saindo depois, sem perda e sem duplicar o saldo (teste previsto).
