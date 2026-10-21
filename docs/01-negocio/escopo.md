# Escopo

O RAZÃO resolve o controle de caixa de um comerciante: registrar cada crédito e débito no momento em que acontece, sem perder nenhum, e entregar o consolidado diário sob demanda, mesmo em dia de pico. Não é um sistema bancário. Regulação financeira fica fora, como evolução.

## Atores

| Ator | O que faz |
| --- | --- |
| Comerciante | Titular do caixa. Todo dado pertence a ele (RN-11). |
| Operador | Registra, estorna e consulta em nome do comerciante. |
| Gestor | Consulta lançamentos e consolidado; não registra nem estorna. |
| Integrador | Sistema que chama a API em nome do comerciante, como uma frente de caixa. |
| Administrador da plataforma | Dispara reconstrução e conciliação; não registra lançamento em nome de comerciante. |
| Contador ou auditor | Consulta o histórico e exige rastreabilidade de cada lançamento e estorno. |

## Dentro

- Registro de crédito e débito, estorno e consulta de lançamentos (EP-01).
- Consolidado diário e extrato por período (EP-02).
- Resiliência, observabilidade e operação (EP-03).
- Autenticação e isolamento por comerciante (EP-04).
- Escrituração por partidas dobradas com plano de contas. Decidi incluir em 20/10/2026, ampliando o handoff, que deixava o plano de contas fora (ADR-0012). Se o prazo não comportar, o escopo recua para um plano fixo mínimo e, depois, para a conta Caixa única.
- Sugestão da conta de contrapartida, sempre fora do caminho de registro e sem gravar lançamento (ADR-0019).

## Fora

- Fechamento de dia pelo contador, com bloqueio de lançamentos em dia fechado.
- Conciliação bancária e integração com extrato, PIX e adquirentes.
- Multimoeda.
- Lançamentos agendados ou recorrentes.
- Exportação para a contabilidade.
- Aplicativo móvel.
- Frente de caixa e emissão de documento fiscal.
- Requisitos de regulação bancária.

O motivo de cada item e o gatilho para ele entrar estão em [evoluções futuras](../08-evolucoes.md).

## Fronteira

Lançamentos é a fonte da verdade. O Consolidado é dado derivado e reconstruível a partir do livro de lançamentos. O Plano de Contas fornece as contas, que o Lançamentos mantém em cópia local. O único caminho entre os serviços é o evento: nenhum deles consulta outro para registrar (C-05).
