# razao

## Problema

<!-- A preencher. -->

## Mapa para o avaliador

<!-- A preencher. -->

## Arquitetura

<!-- A preencher. -->

## Pré-requisitos

<!-- A preencher. -->

## Como subir

<!-- A preencher. -->

## Como testar

<!-- A preencher. -->

## Exemplos

<!-- A preencher. -->

## Interpretação dos NFRs

O controle de lançamento não depende de nenhum outro serviço no ar para registrar: se o Consolidado cair, o caixa continua aceitando lançamentos e o saldo se acerta quando ele volta, sem perda. A tolerância de 5% do enunciado vale só para as 50 leituras por segundo do consolidado, que o integrador pode repetir; para lançamento e evento a perda é zero, porque lançamento perdido é dinheiro sem registro. Detalhe e forma de prova em [docs/02-requisitos/rnf.md](docs/02-requisitos/rnf.md).

## Decisões

O registro de lançamento não depende de nenhum outro serviço no ar: são três microsserviços (Lançamentos, Consolidado e Plano de Contas) que se falam só por eventos no Kafka, com outbox transacional para que nenhum lançamento confirmado fique sem evento ([ADR-0001](docs/04-adr/0001-estilo-arquitetural.md), [ADR-0002](docs/04-adr/0002-outbox-transacional.md)). O livro de lançamentos usa partidas dobradas com plano de contas, e o consolidado diário é a projeção da conta Caixa ([ADR-0012](docs/04-adr/0012-partidas-dobradas.md)). O consumidor do consolidado é imune à ordem e à duplicidade dos eventos, o que cobre o estorno de dia passado ([ADR-0003](docs/04-adr/0003-consumidor-ordem-retroativo.md)).

Todas as decisões, com alternativas descartadas e forma de prova, estão no [índice das ADRs](docs/04-adr/README.md).

## Limitações conhecidas

<!-- A preencher. -->

## O que eu faria em produção

<!-- A preencher. -->
