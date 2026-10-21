# Glossário

Código, API, eventos e documentação usam exatamente estes termos. Sigla nova no repositório entra nesta página no mesmo PR que a introduz.

## Linguagem do domínio

| Termo | Definição |
| --- | --- |
| Comerciante | Titular do caixa. Todo dado pertence a um comerciante. |
| Operador | Pessoa autenticada que registra, estorna e consulta em nome do comerciante. |
| Gestor | Pessoa autenticada que consulta lançamentos e consolidado do comerciante, sem registrar nem estornar. |
| Integrador | Sistema que chama a API em nome do comerciante, como uma frente de caixa. |
| Lançamento | Registro imutável com duas partidas, uma a débito e uma a crédito, de mesmo valor, sempre com a conta Caixa em uma delas (RN-21, RN-24). Tem data de competência, descrição e categoria opcional. |
| Partida | Cada um dos dois lados de um lançamento: a conta, o lado (débito ou crédito) e o valor. |
| Plano de contas | Estrutura de contas do comerciante. Grupos sintéticos fixos (Ativo, Patrimônio Líquido, Receitas, Despesas) e contas analíticas criadas pelo comerciante dentro deles (RN-22). |
| Conta analítica | Conta que recebe partidas. A conta Caixa, no grupo Ativo, existe para todo comerciante. |
| Conta sintética | Grupo do plano de contas. Não recebe partida (I-09). |
| Natureza da conta | Devedora (Ativo, Despesas): o débito aumenta o saldo. Credora (Patrimônio Líquido, Receitas): o crédito aumenta o saldo (RN-23). |
| Contrapartida | A conta do outro lado do lançamento, oposta à conta Caixa. Numa venda, Receitas; num pagamento, Despesas. |
| Crédito | Lado da partida que aumenta conta de natureza credora e reduz conta de natureza devedora. Na conta Caixa, o crédito reduz o saldo. |
| Débito | Lado da partida que aumenta conta de natureza devedora e reduz conta de natureza credora. Na conta Caixa, o débito aumenta o saldo. |
| Estorno | Lançamento que inverte as duas partidas de um lançamento anterior e o referencia. Nunca apaga o original (RN-03, I-03). |
| Data de competência | Dia de negócio ao qual o lançamento pertence. Pode ser diferente da data de registro. |
| Data de registro | Instante, com fuso, em que o sistema gravou o lançamento. Serve à auditoria. |
| Dia de negócio | Dia civil no fuso America/Sao_Paulo, de 00:00 a 23:59:59 (RN-05). |
| Saldo de abertura | Lançamento único por comerciante, com débito em Caixa e crédito em Patrimônio Líquido, que traz o dinheiro já existente em caixa no início do uso (RN-19). |
| Saldo inicial do dia | Saldo final do dia de negócio anterior. |
| Movimento do dia | Débitos em Caixa menos créditos em Caixa no dia. |
| Saldo final do dia | Saldo inicial mais movimento do dia. |
| Consolidado diário | Projeção diária da conta Caixa: saldo inicial, débitos, créditos, saldo final e quantidade de lançamentos (RN-06). |
| Lançamento retroativo | Lançamento com data de competência anterior ao dia atual. Obriga recalcular o dia afetado e os seguintes (RN-07). |
| Chave de idempotência | Identificador enviado pelo integrador que garante que a mesma operação, reenviada, gera um único lançamento (RN-10). |

### Convenção de crédito e débito

Crédito e débito têm um único sentido no RAZÃO: o contábil. Decidi em 20/10/2026, com a adoção de partidas dobradas (ADR-0012), substituindo a visão de extrato do handoff original, em que crédito aumentava o caixa.

Na conta Caixa, de natureza devedora, o débito aumenta o saldo e o crédito reduz. Uma venda é débito em Caixa e crédito em Receitas. Um pagamento a fornecedor é débito em Despesas e crédito em Caixa.

A API não tem campo de tipo. O registro de lançamento recebe a conta de débito (`contaDebito`) e a conta de crédito (`contaCredito`), e os totais do consolidado diário são os débitos e os créditos da conta Caixa nesse mesmo sentido.

## Termos proibidos

| Não use | Use |
| --- | --- |
| transação, movimentação, movimento (como substantivo do lançamento) | lançamento |
| cancelamento, exclusão, edição, ajuste | estorno |
| entrada, saída (como termo técnico) | crédito, débito |
| valor positivo, valor negativo | o lado da partida (RN-01) |
| tipo do lançamento, CRÉDITO/DÉBITO como tipo | conta de débito e conta de crédito |
| crédito ou débito no sentido de extrato bancário | crédito ou débito no sentido contábil |
| "data" sem qualificação | data de competência ou data de registro |
| "saldo" sem qualificação | saldo inicial, movimento do dia ou saldo final |
| dia UTC, "dia" sem fuso | dia de negócio |
| relatório, resumo | consolidado diário |
| lançamento atrasado, correção | lançamento retroativo |
| ID de requisição, token de duplicidade | chave de idempotência |
| cliente, usuário, tenant (em fala de negócio) | comerciante |

## Códigos do projeto

| Código | Significado | Para que serve |
| --- | --- | --- |
| CAP-xx | Capacidade de negócio | O que o negócio precisa ser capaz de fazer |
| RN-xx | Regra de negócio | Comportamento decidido pelo PO |
| I-xx | Invariante do lançamento | Condição que nunca pode ser violada no livro de lançamentos |
| C-xx | Invariante do consolidado | Condição que nunca pode ser violada na projeção de saldos |
| RF-xx | Requisito funcional | O que o sistema faz |
| RNF-xx | Requisito não funcional | Com que qualidade o sistema faz |
| EP-xx | Épico | Agrupa histórias de um mesmo tema |
| US-xx | História de usuário | Unidade de entrega com critério de aceite testável |
| SLO-xx | Objetivo de nível de serviço | Meta medida que transforma um RNF em número |
| TR-xx | Requisito de transição | O que a migração do legado precisa garantir |
| ADR-xxxx | Registro de decisão de arquitetura | Uma decisão, as alternativas descartadas e o teste que a prova |
| G1, G2, G3 | Portões de decisão | Pontos do cronograma em que se aprova, corta ou congela |

## Siglas

| Sigla | Significado | Para que serve |
| --- | --- | --- |
| ADR | Architecture Decision Record | Documenta uma decisão, as alternativas descartadas e como ela é provada |
| API | Application Programming Interface | Contrato pelo qual integradores falam com os serviços |
| BRL | Real brasileiro (ISO 4217) | Moeda única do MVP |
| C4 | Modelo C4 de diagramas | Diagramas de contexto (nível 1) e de contêineres (nível 2) |
| CQRS | Command Query Responsibility Segregation | Separar o modelo de escrita do modelo de leitura |
| CSV | Comma-Separated Values | Formato do arquivo de importação do legado |
| CTN | Código Tributário Nacional | Base do prazo de retenção de 5 anos (RN-13) |
| DDD | Domain-Driven Design | Separar domínios e fixar a linguagem do domínio |
| DoD | Definition of Done | O que precisa existir para a entrega ser aceita |
| GCP | Google Cloud Platform | Nuvem do ambiente alvo |
| GKE | Google Kubernetes Engine | Onde os serviços rodam em produção, no modo Autopilot |
| IA | Inteligência artificial | No RAZÃO, só sugere a conta de contrapartida; nunca grava lançamento (ADR-0019) |
| IaC | Infraestrutura como código | Terraform do ambiente alvo, escrito sem aplicar (ADR-0021) |
| IANA | Internet Assigned Numbers Authority | Mantém a base de fusos usada na RN-05 |
| ID | Identificador | Chave única de um lançamento, evento ou requisição |
| ISO | International Organization for Standardization | Padrões de data (ISO 8601) e de moeda (ISO 4217) |
| JSON | JavaScript Object Notation | Formato das mensagens da API e dos eventos |
| LGPD | Lei Geral de Proteção de Dados (Lei 13.709/2018) | Regras de tratamento de dado pessoal |
| MEI | Microempreendedor Individual | Comerciante pessoa física; seus dados são dados pessoais |
| MoSCoW | Must, Should, Could, Won't | Priorização do backlog |
| MVP | Minimum Viable Product | Escopo mínimo da entrega |
| NFR | Non-Functional Requirement | Requisito não funcional do enunciado; no projeto, RNF |
| OAuth2 | OAuth 2.0 | Padrão de autorização usado pelo Keycloak |
| OIDC | OpenID Connect | Camada de identidade sobre o OAuth2 |
| OpenAPI | Especificação aberta de contratos de API | Formato dos contratos dos serviços |
| OTel | OpenTelemetry | Padrão aberto de métricas, logs e rastreamento |
| p95 | Percentil 95 | Valor abaixo do qual ficam 95% das medições |
| PO | Product Owner | Dono do produto; decide o quê e por quê |
| POO | Programação Orientada a Objetos | Paradigma principal do código (ADR-0022) |
| PR | Pull Request | Única forma de integrar mudança na main |
| RFC | Request for Comments | Padrão técnico da internet, como a RFC 9457 de formato de erro |
| RPO | Recovery Point Objective | Quanto dado se pode perder numa falha; em Lançamentos, zero |
| RTO | Recovery Time Objective | Em quanto tempo o serviço precisa voltar a operar |
| SLO | Service Level Objective | Meta de qualidade medida, com orçamento de erro |
| STRIDE | Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege | Roteiro do modelo de ameaças |
| TLS | Transport Layer Security | Criptografia em trânsito |
| UTC | Tempo Universal Coordenado | Referência de horário; não define o dia de negócio |
