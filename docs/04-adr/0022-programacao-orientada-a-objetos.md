# ADR-0022: Programação orientada a objetos como paradigma principal

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RNF-14

## Contexto

Três serviços escritos em prazo curto precisam de um único estilo, que combine com a arquitetura hexagonal (ADR-0015).

## Decisão

Decidi pela programação orientada a objetos em todo o código. O domínio tem entidades, objetos de valor (valor monetário, data de competência) e o agregado Lançamento. As portas são classes abstratas, e os adaptadores, suas implementações.

Outro paradigma só entra como exceção listada nesta ADR, com o motivo.

## Exceções avaliadas

| Ponto | Decisão | Motivo |
| --- | --- | --- |
| Rotas do FastAPI | Aceita | O framework registra rotas como funções; elas só delegam para os casos de uso |
| Migrações de banco | Aceita | A ferramenta de migração define cada passo como função |
| Fixtures de teste | Aceita | O pytest define fixtures como funções |

## Alternativas descartadas

- Estilo funcional: descartei porque se afasta das portas e dos adaptadores da hexagonal.

## Consequências aceitas

Ganhamos um estilo único e previsível. Pagamos mais classes em pontos simples.

## Como provamos

Revisão final do repositório contra esta ADR, antes da entrega.
