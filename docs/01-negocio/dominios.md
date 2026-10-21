# Domínios e capacidades

## Domínios

| Domínio | Tipo | Responsabilidade | Dono do dado | Por que esta classificação |
| --- | --- | --- | --- | --- |
| Lançamentos | Core | Registrar, estornar e consultar lançamentos. Fonte da verdade. | Livro de lançamentos | É onde mora o valor do produto e o risco: lançamento perdido é dinheiro sem registro |
| Consolidado Diário | Core | Calcular e servir o saldo da conta Caixa por dia e por período. Dado derivado. | Projeção de saldos | É o que o comerciante consulta todo dia; carrega o NFR de 50 leituras por segundo |
| Plano de Contas | Suporte | Manter as contas do comerciante e sugerir a conta de contrapartida. | Plano de contas | É necessário para a escrituração, mas a estrutura de contas é conhecimento contábil padrão, não diferencial do produto |
| Identidade e Acesso | Genérico | Autenticar o usuário e restringir cada um ao seu comerciante. | Provedor de identidade | Problema resolvido pelo mercado; uso o Keycloak em vez de construir |
| Auditoria | Suporte | Registrar quem fez o quê e quando, de forma consultável. | Trilha de auditoria | Exigência do contador e do auditor, sem regra de negócio própria |
| Observabilidade | Suporte | Métricas técnicas e de negócio, logs e rastreamento. | Plataforma | Necessária para provar os NFRs; não é o produto |

## Capacidades de negócio

| Código | Capacidade | Domínio |
| --- | --- | --- |
| CAP-01 | Registrar lançamento que aumenta o caixa (débito em Caixa) | Lançamentos |
| CAP-02 | Registrar lançamento que reduz o caixa (crédito em Caixa) | Lançamentos |
| CAP-03 | Estornar lançamento | Lançamentos |
| CAP-04 | Consultar lançamentos por período e filtro | Lançamentos |
| CAP-05 | Publicar o fato de lançamento registrado | Lançamentos |
| CAP-06 | Consolidar o saldo do dia | Consolidado Diário |
| CAP-07 | Consultar o saldo de um dia | Consolidado Diário |
| CAP-08 | Consultar o extrato consolidado de um período | Consolidado Diário |
| CAP-09 | Recalcular os dias afetados por lançamento com data passada | Consolidado Diário |
| CAP-10 | Reconstruir o consolidado a partir dos lançamentos | Consolidado Diário |
| CAP-11 | Conciliar o consolidado contra os lançamentos | Consolidado Diário |
| CAP-12 | Autenticar e autorizar por comerciante | Identidade e Acesso |
| CAP-13 | Manter o plano de contas do comerciante | Plano de Contas |
| CAP-14 | Sugerir a conta de contrapartida | Plano de Contas |

CAP-01 e CAP-02 seguem a numeração do handoff. Com partidas dobradas, as duas são o mesmo registro com a conta Caixa em lados diferentes (RN-21, RN-24). CAP-13 e CAP-14 nasceram da decisão de 20/10/2026 de adotar plano de contas.

## Relação entre os domínios

Lançamentos é upstream do Consolidado: publica eventos e nunca consulta o Consolidado para registrar. Plano de Contas é upstream de Lançamentos: publica as alterações de conta, que o Lançamentos guarda em cópia local. Nenhum domínio escreve no dado de outro. O mapa está em [mapa de contextos](../03-arquitetura/mapa-de-contextos.md).
