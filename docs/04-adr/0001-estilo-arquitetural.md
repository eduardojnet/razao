# ADR-0001: Três microsserviços integrados por eventos

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-01, RNF-08, C-05

## Contexto

O NFR 1 do enunciado exige que o controle de lançamentos continue operando com o consolidado fora. Isso pede, no mínimo, duas unidades implantáveis que não dependam uma da outra em tempo de execução. A adoção de plano de contas (ADR-0012) trouxe um terceiro contexto com dono de dado próprio.

## Decisão

Decidimos por três microsserviços: `razao-lancamentos`, `razao-consolidado` e `razao-plano-contas`. Eles se integram só por eventos. Nenhum serviço chama outro no caminho de registro de lançamento.

## Alternativas descartadas

- Monolito modular: descartei porque uma queda do processo derruba o registro junto com o consolidado, o oposto do NFR 1.
- Serverless por função: descartei porque os consumidores de eventos precisam rodar continuamente e o ambiente local para o avaliador ficaria distante do de produção.
- Dois serviços, com o plano de contas dentro de Lançamentos: descartei por decisão de fronteira. O plano de contas tem ciclo de vida e dono próprios.

## Consequências aceitas

Ganhamos isolamento de falha e escala independente por serviço. Pagamos consistência eventual no consolidado diário e mais peças para operar. O terceiro serviço exige a réplica local do plano no Lançamentos (ADR-0018).

## Como provamos

Testes de queda em `testes/resiliencia`: com o Consolidado parado e com o Plano de Contas parado, o registro de lançamento segue respondendo sucesso (testes previstos).
