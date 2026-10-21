# ADR-0007: Python 3.12 com FastAPI nos três serviços

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RN-02, RNF-14

## Contexto

A linguagem precisa tratar valor monetário sem perda de precisão, ser testável e ser defensável por quem escreveu o código.

## Decisão

Escolhi Python 3.12 com FastAPI nos três serviços.

Para compensar o tipo dinâmico, assumo três regras:

- valor monetário sempre em `decimal.Decimal`, com `float` proibido na API, no evento e no banco (ADR-0008);
- tipagem estrita verificada no editor e no pipeline;
- invariantes de concorrência presas no banco (ADR-0006), não no código.

## Alternativas descartadas

- Java 21 com Spring Boot 3: tem tipo decimal nativo e é padrão em core bancário. Descartei por decisão minha de produtividade no prazo.
- C# com .NET 8: equivalente técnico ao Java. Descartei por estar fora da minha base de trabalho.

## Consequências aceitas

Ganhamos velocidade de escrita. Pagamos um sinal mais fraco para quem avalia sistemas bancários e mais disciplina de tipagem e teste.

## Como provamos

Teste que envia valor com três casas decimais e recebe 422; teste que confirma que nenhum valor trafega como número de ponto flutuante (testes previstos).
