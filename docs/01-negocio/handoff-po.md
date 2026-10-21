# RAZÃO - Handoff do Product Owner

Oct 20, 2026 · @Eduardo José Gonçalves do Carmo

## Resumo executivo e nome do projeto

**Nome do projeto: RAZÃO** - Plataforma de Livro-Caixa e Consolidação Diária.

O nome vem do *livro razão*, o registro contábil em que cada conta acumula seus débitos e créditos até formar o saldo. É exatamente o que o produto faz: registra lançamentos imutáveis e deriva deles o saldo diário. Curto, em português, sem colisão com marca de banco e com leitura dupla (razão contábil e razão como lógica verificável).

Nomenclatura técnica sugerida para o Arquiteto (ajustável por ele):

| Elemento | Nome |
| --- | --- |
| Repositório | `razao` |
| Serviço de lançamentos | `razao-lancamentos` |
| Serviço de consolidado | `razao-consolidado` |
| Evento de integração | `LancamentoRegistrado` |

**O que este documento entrega ao Arquiteto:** o problema de negócio refinado, as regras de negócio já decididas pelo PO, as ambiguidades do enunciado resolvidas, o backlog com critérios de aceite e a matriz do que precisa existir no repositório para o teste não ser descartado.

**Prazo:** entrega até sexta-feira, 23/10/2026, pela manhã. Hora proposta pelo PO: 08:00, com três portões de decisão antes (ver Cronograma).

**As três decisões de produto que mais pesam na avaliação:**

1. O livro de lançamentos é **imutável**: correção se faz por estorno, nunca por edição ou exclusão.
2. A tolerância de **5% de perda vale só para leituras do consolidado**. Perda de lançamento ou de evento é **zero**.
3. O consolidado é um **dado derivado e reconstruível** a partir dos lançamentos. Se cair, o caixa continua operando e o saldo é recalculado depois.

## Contexto, problema e objetivo

**Problema:** um comerciante precisa saber, com confiança, quanto entrou, quanto saiu e qual é o saldo de cada dia. Hoje o enunciado não diz como ele faz isso; o PO assume que é manual (planilha ou caderno) e sujeito a erro, retrabalho e perda de histórico.

**Objetivo de produto:** registrar todo débito e crédito no momento em que acontece, sem nunca perder um lançamento, e entregar o saldo diário consolidado sob demanda, mesmo em dia de pico.

**Persona principal:** o comerciante (dono ou operador de caixa). Persona secundária: o contador ou auditor que consulta o histórico e precisa de rastreabilidade.

**Jornada essencial:**

1. O operador registra uma venda (crédito) ou um pagamento (débito).
2. O sistema confirma na hora que o lançamento foi gravado.
3. O comerciante consulta o saldo do dia ou de um período.
4. Se errou um lançamento, registra o estorno; o histórico mostra o original e a correção.

**Métricas de sucesso do produto:**

| Métrica | Meta | Por que importa |
| --- | --- | --- |
| Lançamentos perdidos | 0 | Dado financeiro não tolera perda |
| Disponibilidade de lançamentos com consolidado fora | 100% do tempo de queda do consolidado | Requisito explícito do enunciado |
| Sucesso das leituras do consolidado a 50 req/s | >= 95% (meta interna >= 99%) | Requisito explícito do enunciado |
| Divergência entre soma dos lançamentos e saldo consolidado | 0 após o atraso máximo aceito | Prova de integridade |
| Atraso de consolidação (lançamento até saldo atualizado) | A definir pelo Arquiteto; proposta do PO: p95 <= 5 s | Consistência eventual precisa de limite declarado |

A meta de atraso é **premissa do PO a validar**, não requisito do enunciado.

## Glossário de domínio

Estes termos são a linguagem ubíqua do projeto: código, API, eventos e documentação usam exatamente estas palavras.

| Termo | Definição |
| --- | --- |
| Comerciante | Titular do caixa. Todo dado pertence a um comerciante. |
| Operador | Pessoa autenticada que registra, estorna e consulta em nome do comerciante. É o "usuário" dos requisitos de acesso e auditoria. |
| Integrador | Sistema que chama a API em nome do comerciante (ex.: frente de caixa). |
| Lançamento | Registro imutável que altera o saldo do caixa: tipo, valor, data de competência, descrição, categoria opcional. |
| Crédito | Lançamento que aumenta o saldo do caixa. |
| Débito | Lançamento que reduz o saldo do caixa. |
| Estorno | Lançamento de tipo oposto que anula um lançamento anterior, referenciando-o. Nunca apaga o original. |
| Data de competência | Dia de negócio ao qual o lançamento pertence. Pode ser diferente da data de registro. |
| Data de registro | Instante, com fuso, em que o sistema gravou o lançamento (auditoria). |
| Dia de negócio | Dia civil no fuso America/Sao\_Paulo, de 00:00 a 23:59:59. |
| Saldo de abertura | Lançamento único por comerciante, com categoria "Saldo de abertura", que traz o dinheiro já existente em caixa no início do uso (RN-19). |
| Saldo inicial do dia | Saldo final do dia de negócio anterior. |
| Movimento do dia | Total de créditos menos total de débitos do dia. |
| Saldo final do dia | Saldo inicial mais movimento do dia. |
| Consolidado diário | Visão derivada por dia: saldo inicial, total de créditos, total de débitos, saldo final, quantidade de lançamentos. |
| Lançamento retroativo | Lançamento com data de competência anterior ao dia atual. Obriga recalcular o dia afetado e os seguintes. |
| Chave de idempotência | Identificador enviado pelo integrador que garante que a mesma operação, reenviada, gera um único lançamento. |

**Proibidos como sinônimo:** transação, movimentação, relatório, cancelamento, exclusão, edição, ajuste, entrada e saída como termo técnico, "data" sem qualificação, "saldo" sem qualificação, cliente ou tenant em fala de negócio.

## Glossário de siglas e códigos

Toda sigla usada neste documento, no repositório e nas ADRs entra aqui. Sigla nova sem linha nesta tabela é erro de documentação.

**Códigos do projeto:**

| Código | Significado | Para que serve |
| --- | --- | --- |
| CAP-xx | Capacidade de negócio | O que o negócio precisa ser capaz de fazer |
| RN-xx | Regra de negócio | Comportamento decidido pelo PO; o Arquiteto implementa, não renegocia |
| I-xx | Invariante do Lançamento | Condição que nunca pode ser violada no livro de lançamentos |
| C-xx | Invariante do Consolidado | Condição que nunca pode ser violada na projeção de saldos |
| RF-xx | Requisito funcional | O que o sistema faz |
| RNF-xx | Requisito não funcional | Com que qualidade o sistema faz (disponibilidade, latência, segurança) |
| EP-xx | Épico | Agrupamento de histórias de um mesmo tema |
| US-xx | História de usuário (User Story) | Unidade de entrega com critério de aceite testável |
| SLO-xx | Objetivo de nível de serviço | Meta medida que transforma um RNF em número |
| TR-xx | Requisito de transição | O que a migração do legado precisa garantir |
| G1, G2, G3 | Portões de decisão | Pontos do cronograma em que se aprova, corta ou congela |

**Siglas técnicas e de negócio:**

| Sigla | Significado | Para que serve |
| --- | --- | --- |
| ADR | Architecture Decision Record (registro de decisão de arquitetura) | Documenta uma decisão, as alternativas descartadas e como é provada |
| API | Application Programming Interface | Contrato pelo qual integradores falam com os serviços |
| BRL | Real brasileiro (código ISO 4217) | Moeda única do MVP |
| CSV | Comma-Separated Values | Formato do arquivo de importação do legado |
| CTN | Código Tributário Nacional | Base do prazo de retenção de 5 anos (RN-13) |
| DDD | Domain-Driven Design | Método usado para separar domínios e a linguagem ubíqua |
| DoD | Definition of Done | Lista do que precisa existir para a entrega ser aceita |
| IANA | Internet Assigned Numbers Authority | Mantém a base oficial de fusos horários usada na RN-05 |
| ID | Identificador | Chave única de um lançamento, evento ou requisição |
| ISO | International Organization for Standardization | Padrões de data (ISO 8601) e de moeda (ISO 4217) |
| JSON | JavaScript Object Notation | Formato das mensagens da API e do evento |
| LGPD | Lei Geral de Proteção de Dados (Lei 13.709/2018) | Regras de tratamento de dado pessoal |
| MEI | Microempreendedor Individual | Comerciante pessoa física; seus dados são dados pessoais |
| MoSCoW | Must, Should, Could, Won't | Método de priorização do backlog |
| MVP | Minimum Viable Product (produto mínimo viável) | Escopo mínimo entregue até sexta |
| NFR | Non-Functional Requirement | Requisito não funcional do enunciado; no projeto, RNF |
| OpenAPI | Especificação aberta de contratos de API | Formato em que o Arquiteto publica os contratos |
| p95 | Percentil 95 | Valor abaixo do qual ficam 95% das medições |
| PO | Product Owner | Dono do produto; decide o quê e por quê |
| RFC | Request for Comments | Padrão técnico da internet (ex.: RFC 9457, formato de erro) |
| RPO | Recovery Point Objective | Quanto dado se pode perder numa falha (Lançamentos: zero) |
| RTO | Recovery Time Objective | Em quanto tempo o serviço precisa voltar a operar |
| SLO | Service Level Objective | Meta de qualidade medida, com orçamento de erro |
| STRIDE | Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege | Roteiro do modelo de ameaças |
| TLS | Transport Layer Security | Criptografia em trânsito |
| UTC | Tempo Universal Coordenado | Referência de horário; não define o dia de negócio |

## Domínios funcionais e capacidades de negócio

O negócio tem dois domínios centrais, com donos de dado separados, e três domínios de suporte. A fronteira entre Lançamentos e Consolidado é a que sustenta o requisito de disponibilidade.

&#91;embedded content: mapa de contexto · 2 domínios core, 3 de suporte\]

O único caminho entre os dois domínios core é o evento: se o Consolidado cair, o comerciante continua registrando e o saldo é atualizado quando ele voltar.

| Domínio | Tipo (DDD) | Responsabilidade | Dono do dado |
| --- | --- | --- | --- |
| Lançamentos | Core | Registrar, estornar e consultar lançamentos. Fonte da verdade. | Livro de lançamentos |
| Consolidado Diário | Core | Calcular e servir o saldo por dia e por período. Dado derivado. | Projeção de saldos |
| Identidade e Acesso | Genérico | Autenticar o usuário e garantir que ele só veja o caixa do seu comerciante. | Provedor de identidade |
| Auditoria | Suporte | Registrar quem fez o quê e quando, de forma consultável. | Trilha de auditoria |
| Observabilidade | Suporte | Métricas técnicas e de negócio, logs e rastreamento. | Plataforma |

**Capacidades de negócio (o que o negócio precisa ser capaz de fazer):**

| Código | Capacidade | Domínio |
| --- | --- | --- |
| CAP-01 | Registrar crédito | Lançamentos |
| CAP-02 | Registrar débito | Lançamentos |
| CAP-03 | Estornar lançamento | Lançamentos |
| CAP-04 | Consultar lançamentos por período e filtro | Lançamentos |
| CAP-05 | Publicar fato de lançamento registrado | Lançamentos |
| CAP-06 | Consolidar saldo do dia | Consolidado |
| CAP-07 | Consultar saldo de um dia | Consolidado |
| CAP-08 | Consultar extrato consolidado de um período | Consolidado |
| CAP-09 | Recalcular dias afetados por lançamento retroativo | Consolidado |
| CAP-10 | Reconstruir o consolidado a partir dos lançamentos | Consolidado |
| CAP-11 | Conciliar consolidado contra lançamentos | Consolidado |
| CAP-12 | Autenticar e autorizar por comerciante | Identidade |

**Relação entre os domínios:** Lançamentos é *upstream*; Consolidado é *downstream* e consome apenas eventos publicados. Consolidado nunca escreve em Lançamentos e Lançamentos nunca consulta Consolidado para registrar.

## Modelo de domínio e invariantes

O domínio tem um agregado com regras fortes (Lançamento) e uma projeção derivada (Consolidado Diário). As invariantes abaixo são o que os testes precisam proteger; se uma delas puder ser violada por qualquer caminho, o desenho não está aprovado.

### Agregado Lançamento (fonte da verdade)

| Atributo | Regra | Obrigatório |
| --- | --- | --- |
| Identificador | Gerado pelo sistema, único global | Sim |
| Comerciante | Vem da identidade autenticada, nunca do corpo da requisição | Sim |
| Tipo | CRÉDITO ou DÉBITO | Sim |
| Valor | Decimal positivo, 2 casas, até o teto da RN-17 | Sim |
| Data de competência | Dia de negócio; padrão = hoje (RN-20) | Sim (com padrão) |
| Descrição | 3 a 140 caracteres (RN-16) | Sim |
| Categoria | Texto controlado, opcional | Não |
| Estorno de | Identificador do lançamento original, quando for estorno | Só em estorno |
| Motivo do estorno | Texto livre, 3 a 140 caracteres | Só em estorno |
| Registrado em | Data de registro, com fuso | Sim |
| Registrado por | Usuário autenticado | Sim |
| Chave de idempotência | Enviada pelo integrador | Sim no registro e no estorno |
| ID de correlação | Recebido ou gerado na entrada | Sim |

**Invariantes do Lançamento:**

| Código | Invariante |
| --- | --- |
| I-01 | Valor sempre maior que zero; o efeito no saldo vem do tipo. |
| I-02 | Depois de gravado, nenhum atributo muda. Não há operação de edição nem de exclusão. |
| I-03 | O estorno tem tipo oposto, mesmo valor e mesmo comerciante do original. |
| I-04 | Um original tem no máximo um estorno, inclusive sob requisições simultâneas. |
| I-05 | Um estorno não pode ser estornado. |
| I-06 | A situação "estornado" do original é derivada da existência do estorno, sem alterar o registro original. |
| I-07 | Chave de idempotência é única por comerciante durante a janela de retenção (proposta: 24 horas). |

### Projeção Consolidado Diário (dado derivado)

| Atributo | Regra |
| --- | --- |
| Comerciante + data | Chave da projeção |
| Saldo inicial | Saldo final do dia anterior; zero antes do primeiro lançamento |
| Total de créditos | Soma dos créditos do dia, estornos incluídos pelo seu tipo |
| Total de débitos | Soma dos débitos do dia, estornos incluídos pelo seu tipo |
| Saldo final | Saldo inicial + créditos - débitos |
| Quantidade de lançamentos | Inclui estornos |
| Quantidade de estornos | Para leitura do comerciante e do contador |
| Atualizado em | Instante da última alteração aplicada |
| Situação | ATUALIZADO ou EM\_RECALCULO (proposta; viabilidade é do Arquiteto) |

**Invariantes do Consolidado:**

| Código | Invariante |
| --- | --- |
| C-01 | Saldo final = saldo inicial + créditos - débitos, em todo dia. |
| C-02 | Saldo inicial do dia D = saldo final do dia D-1, sem lacuna. |
| C-03 | Passado o atraso máximo, o saldo final de D é igual à soma direta de todos os lançamentos até D. |
| C-04 | Cada evento altera a projeção no máximo uma vez, qualquer que seja a ordem de chegada. |
| C-05 | O Consolidado nunca é fonte para decisão de escrita em Lançamentos. |

**Ciclo de vida:** o lançamento nasce REGISTRADO e permanece assim. Se houver estorno, a leitura o apresenta como ESTORNADO, com o vínculo para o estorno. Não existem outros estados no MVP.

## Regras de negócio decididas pelo PO

Estas regras não são escolha técnica: o Arquiteto implementa, não renegocia. Cada uma resolve uma ambiguidade do enunciado.

| Código | Regra | Ambiguidade que resolve |
| --- | --- | --- |
| RN-01 | Todo lançamento tem tipo (CRÉDITO ou DÉBITO), valor estritamente positivo, data de competência e descrição. O sinal vem do tipo, nunca do valor. | Valor negativo vs tipo |
| RN-02 | Valores em BRL com 2 casas decimais, aritmética decimal exata. Ponto flutuante é proibido. | Precisão monetária |
| RN-03 | Lançamento é imutável. Não existe editar nem excluir. Correção = estorno, que referencia o original. | Edição e auditoria |
| RN-04 | Um lançamento só pode ser estornado uma vez. Estorno de estorno não é permitido. | Estorno em cadeia |
| RN-05 | O dia de negócio segue o fuso America/Sao\_Paulo. | Fuso do "dia" |
| RN-06 | Saldo diário = saldo inicial (saldo final do dia anterior) + créditos - débitos do dia. O consolidado diário mostra os quatro números. | "Saldo diário" pode ser só o movimento ou o acumulado |
| RN-07 | Lançamento retroativo é aceito até um limite configurável (proposta: 90 dias) e dispara recálculo do dia afetado e de todos os seguintes. | Lançamento com data passada |
| RN-08 | Lançamento com data de competência futura é recusado. | Agendamento fora de escopo |
| RN-09 | Saldo negativo é permitido e sinalizado no consolidado diário. O sistema registra o caixa real; não bloqueia o comerciante. | Bloqueio por saldo |
| RN-10 | Toda requisição de registro aceita chave de idempotência. A mesma chave com o mesmo conteúdo devolve o lançamento já criado; com conteúdo diferente, é recusada. | Duplicidade em retentativa |
| RN-11 | Todo dado pertence a um comerciante. Um usuário nunca lê nem escreve dado de outro comerciante. | Multiempresa |
| RN-12 | Dia sem movimento aparece no extrato com movimento zero e saldo igual ao inicial. | Lacunas no consolidado diário |
| RN-13 | Lançamentos e consolidados são retidos por no mínimo 5 anos. | Retenção |
| RN-14 | O consolidado exposto informa a que horas foi atualizado pela última vez. | Transparência da consistência eventual |

**Regras complementares (detalhamento de 20/10):**

| Código | Regra | Ambiguidade que resolve |
| --- | --- | --- |
| RN-15 | O estorno herda a data de competência do original, corrigindo o dia certo. Se o original estiver fora da janela da RN-07, o estorno é lançado na data atual. | Em que dia o estorno pesa |
| RN-16 | Descrição obrigatória, de 3 a 140 caracteres. A API e o README orientam a não incluir dado pessoal. | Tamanho e conteúdo da descrição |
| RN-17 | Teto por lançamento configurável (proposta: R$ 1.000.000,00). Acima disso, recusa com 422. Protege contra erro de digitação. | Valor absurdo |
| RN-18 | Ordem oficial de listagem: data de competência e, dentro do dia, data de registro. | Ordenação do extrato |
| RN-19 | O saldo de abertura entra como lançamento comum, com categoria "Saldo de abertura", na data de início de uso. | Comerciante que já tem dinheiro em caixa |
| RN-20 | Se a data de competência não for enviada, vale a data atual no fuso da RN-05. | Campo omitido |

**Notas do PO:**

- RN-13 segue o horizonte de 5 anos dos prazos de decadência e prescrição tributária (CTN, arts. 173 e 174). Retenção maior é decisão de negócio, não técnica.
- O limite de 90 dias em RN-07 é premissa do PO para conter o custo de recálculo; o Arquiteto pode propor outro valor com justificativa.
- Categoria do lançamento (ex.: venda, fornecedor, aluguel) é opcional no MVP e livre-texto controlado; plano de contas fica fora de escopo.

## Exemplo de cálculo do saldo: teste de aceitação de referência

Este cenário deve virar um teste automatizado com exatamente estes números. Ele cobre saldo de abertura, dia sem movimento, lançamento retroativo e estorno no mesmo fluxo.

**Sequência de fatos (ordem de registro):**

1. 19/10: crédito R$ 1.000,00, categoria "Saldo de abertura" (RN-19).
2. 19/10: débito R$ 300,00, pagamento de fornecedor.
3. 20/10: crédito R$ 500,00 e crédito R$ 250,00, vendas.
4. 20/10: débito R$ 100,00, frete.
5. 21/10: nenhum lançamento.
6. 22/10: crédito retroativo R$ 200,00 com competência 20/10 (venda esquecida, RN-07).
7. 22/10: estorno do débito de R$ 100,00 de 20/10; gera crédito R$ 100,00 com competência 20/10 (RN-15).

**Consolidado esperado antes dos fatos 6 e 7 (posição em 21/10):**

| Dia | Saldo inicial (R$) | Créditos (R$) | Débitos (R$) | Saldo final (R$) |
| --- | --- | --- | --- | --- |
| 19/10 | 0,00 | 1.000,00 | 300,00 | 700,00 |
| 20/10 | 700,00 | 750,00 | 100,00 | 1.350,00 |
| 21/10 | 1.350,00 | 0,00 | 0,00 | 1.350,00 |

**Consolidado esperado depois dos fatos 6 e 7 (posição em 22/10):**

| Dia | Saldo inicial (R$) | Créditos (R$) | Débitos (R$) | Saldo final (R$) | Estornos |
| --- | --- | --- | --- | --- | --- |
| 19/10 | 0,00 | 1.000,00 | 300,00 | 700,00 | 0 |
| 20/10 | 700,00 | 1.050,00 | 100,00 | 1.650,00 | 1 |
| 21/10 | 1.650,00 | 0,00 | 0,00 | 1.650,00 | 0 |
| 22/10 | 1.650,00 | 0,00 | 0,00 | 1.650,00 | 0 |

**Prova de integridade (C-03):** créditos totais R$ 2.050,00 menos débitos totais R$ 400,00 = R$ 1.650,00, igual ao saldo final de 22/10.

**Variações que o mesmo teste deve cobrir:** os eventos dos fatos 6 e 7 chegando ao Consolidado em ordem invertida; o evento do fato 6 entregue duas vezes; o Consolidado parado durante os fatos 6 e 7 e religado depois. Em todos, a tabela final é idêntica.

## Casos de borda e comportamento esperado

Cada linha é um teste. O comportamento é decisão do PO; o mecanismo que o garante é decisão do Arquiteto.

| Caso | Comportamento esperado | Regra |
| --- | --- | --- |
| Lançamento registrado às 23:59:59 sem data de competência | Pertence ao dia corrente em America/Sao\_Paulo | RN-05, RN-20 |
| Servidor em UTC e registro às 22:00 de São Paulo | Continua no dia de São Paulo, não no dia seguinte UTC | RN-05 |
| Mudança futura de fuso por lei | Usar a base de fusos IANA, nunca deslocamento fixo de -3 h | RN-05 |
| Valor com 3 casas decimais, zero ou negativo | 422 com o campo identificado | RN-01, RN-02 |
| Valor acima do teto | 422 | RN-17 |
| Valor trafegado em JSON | Sem perda de precisão no trânsito; formato definido pelo Arquiteto | RN-02 |
| Data de competência futura | 422 | RN-08 |
| Data de competência além da janela retroativa | 422 | RN-07 |
| Dois lançamentos simultâneos no mesmo dia | Ambos registrados e somados | C-01 |
| Dois pedidos simultâneos de estorno do mesmo lançamento | Exatamente um vence; o outro recebe 409 | I-04 |
| Estorno de um estorno | 409 | I-05 |
| Estorno de original fora da janela retroativa | Estorno lançado na data atual | RN-15 |
| Mesma chave de idempotência e mesmo conteúdo | Devolve o lançamento original, sem criar outro | RN-10 |
| Mesma chave e conteúdo diferente | Recusa (409 ou 422, a definir pelo Arquiteto) | RN-10 |
| Requisição sem chave de idempotência | 400 | RN-10 |
| Evento chega fora de ordem | Saldo final idêntico ao da ordem correta | C-04 |
| Evento entregue duas vezes | Aplicado uma vez | C-04 |
| Consolidado parado por horas | Lançamentos seguem; eventos acumulam; ao voltar, saldo converge e o atraso é medido e alertado | NFR 1 |
| Consulta de dia anterior ao primeiro lançamento | Saldos zerados, sem erro | C-02 |
| Consulta de dia futuro | 422 | RN-08 |
| Extrato maior que 366 dias | 422 | RF-08 |
| Comerciante sem nenhum lançamento | Extrato com zeros em todos os dias | RN-12 |
| Usuário tenta acessar dado de outro comerciante por ID | 404, sem revelar existência; tentativa auditada | RN-11 |
| Descrição contendo CPF ou e-mail | Aceita (não há detecção confiável), mas nunca escrita em log | RN-16, RNF-11 |
| Saldo final negativo | Exibido e sinalizado, sem bloqueio | RN-09 |
| Segundo "Saldo de abertura" para o mesmo comerciante | 409 | RN-19 |

## Requisitos funcionais refinados

O enunciado pede dois serviços em uma linha cada. Refinados, viram onze requisitos verificáveis.

| Código | Requisito | Serviço | Origem |
| --- | --- | --- | --- |
| RF-01 | Registrar lançamento de crédito ou débito, devolvendo identificador e data de registro. | Lançamentos | Enunciado |
| RF-02 | Estornar um lançamento existente, gerando novo lançamento de sinal oposto vinculado ao original. | Lançamentos | PO (RN-03) |
| RF-03 | Consultar um lançamento pelo identificador. | Lançamentos | PO |
| RF-04 | Listar lançamentos por intervalo de datas de competência, com filtro por tipo e paginação. | Lançamentos | PO |
| RF-05 | Publicar, para cada lançamento gravado, um evento de integração com garantia de entrega pelo menos uma vez. | Lançamentos | PO (NFR) |
| RF-06 | Consumir os eventos e atualizar o consolidado do dia de competência de forma idempotente (evento repetido não altera saldo). | Consolidado | Enunciado + PO |
| RF-07 | Consultar o consolidado de um dia: saldo inicial, créditos, débitos, saldo final, quantidade e hora da última atualização. | Consolidado | Enunciado |
| RF-08 | Consultar o extrato consolidado de um período (máximo proposto: 366 dias), um item por dia, incluindo dias sem movimento. | Consolidado | PO |
| RF-09 | Recalcular automaticamente o dia afetado e os seguintes ao receber lançamento retroativo. | Consolidado | PO (RN-07) |
| RF-10 | Reconstruir todo o consolidado de um comerciante a partir do livro de lançamentos, sob comando operacional. | Consolidado | PO |
| RF-11 | Executar conciliação periódica: comparar o saldo consolidado com a soma direta dos lançamentos e alertar qualquer divergência. | Consolidado | PO |

**Contrato mínimo do evento `LancamentoRegistrado` (proposta do PO, formato final é do Arquiteto):** identificador do evento, identificador do lançamento, comerciante, tipo, valor, data de competência, data de registro, referência ao lançamento estornado (quando houver), versão do esquema.

## Contratos propostos pelo PO

**Modelo, não especificação final.** O PO define o que cada operação precisa receber e devolver; nomes, formato e versionamento finais são do Arquiteto, registrados em OpenAPI no repositório.

**Regras transversais de contrato:**

- O comerciante é sempre extraído da credencial. Nenhum endpoint aceita comerciante no corpo ou na rota.
- Todo erro segue um formato único e legível por máquina; proposta: Problem Details (RFC 9457).
- Toda resposta devolve o ID de correlação recebido ou gerado.
- Versão na rota ou no cabeçalho desde a primeira entrega.
- Listagens com paginação por cursor e limite máximo por página.

**Operações:**

| Operação | Proposta | Perfil | Sucesso | Erros previstos |
| --- | --- | --- | --- | --- |
| Registrar lançamento | `POST /v1/lancamentos` | Operador | 201 | 400, 401, 409, 422, 429 |
| Estornar lançamento | `POST /v1/lancamentos/{id}/estornos` | Operador | 201 | 401, 404, 409, 422, 429 |
| Consultar lançamento | `GET /v1/lancamentos/{id}` | Operador, Gestor | 200 | 401, 404 |
| Listar lançamentos | `GET /v1/lancamentos?de=&ate=&tipo=&cursor=` | Operador, Gestor | 200 | 401, 422 |
| Consultar consolidado do dia | `GET /v1/consolidados/{data}` | Operador, Gestor | 200 | 401, 422, 429 |
| Extrato consolidado | `GET /v1/consolidados?de=&ate=` | Operador, Gestor | 200 | 401, 422, 429 |
| Reconstruir consolidado | `POST /v1/admin/reconstrucoes` | Administrador da plataforma | 202 | 401, 403, 409 |
| Saúde e prontidão | Endpoints de verificação de cada serviço | Plataforma | 200 | 503 |

**Exemplo de registro (modelo):**

```json
POST /v1/lancamentos
Idempotency-Key: 7b1e4c2a-0f7d-4a8e-9a51-2c3d4e5f6a7b

{
  "tipo": "CREDITO",
  "valor": "150.00",
  "dataCompetencia": "2026-10-20",
  "descricao": "Venda balcão",
  "categoria": "Vendas"
}
```

**Exemplo de consolidado do dia (modelo):**

```json
GET /v1/consolidados/2026-10-20

{
  "data": "2026-10-20",
  "saldoInicial": "700.00",
  "totalCreditos": "1050.00",
  "totalDebitos": "100.00",
  "saldoFinal": "1650.00",
  "quantidadeLancamentos": 5,
  "quantidadeEstornos": 1,
  "situacao": "ATUALIZADO",
  "atualizadoEm": "2026-10-22T14:03:11-03:00"
}
```

O valor aparece como texto decimal no modelo apenas para deixar visível a exigência de precisão; o Arquiteto pode escolher outra representação sem perda (ex.: centavos inteiros) e registrar em ADR.

**Evento `LancamentoRegistrado` (modelo):**

| Campo | Conteúdo |
| --- | --- |
| eventoId | Único; base da idempotência do consumidor |
| versaoEsquema | Inteiro; muda a cada alteração incompatível |
| ocorridoEm | Instante do registro |
| correlacaoId | Mesmo da requisição de origem |
| comercianteId | Dono do lançamento |
| lancamentoId | Identificador do lançamento |
| tipo | CRÉDITO ou DÉBITO |
| valor | Mesma representação exata da API |
| dataCompetencia | Dia de negócio afetado |
| estornoDe | Identificador do original, quando for estorno |
| categoria | Quando houver |

**Requisitos do PO sobre o evento:** descrição não trafega no evento (minimização de dado); a ordem por comerciante deve ser preservada ou o consumidor deve ser imune à ordem, à escolha do Arquiteto; um consumidor novo deve conseguir reprocessar o histórico sem envolver o serviço de Lançamentos.

## Requisitos não funcionais refinados

O enunciado traz dois NFRs. O PO os interpreta assim, e esta interpretação deve estar escrita no README e nas ADRs, porque o avaliador vai procurar exatamente isso.

**NFR do enunciado 1:** "o controle de lançamento não deve ficar indisponível se o consolidado cair".

- Leitura do PO: o caminho de escrita de lançamentos **não pode ter dependência síncrona** do Consolidado. Nem chamada, nem banco compartilhado, nem fila que bloqueie a gravação.
- Consequência aceita: o saldo fica temporariamente desatualizado; os eventos aguardam e são processados quando o Consolidado volta, sem perda.
- Prova exigida: teste automatizado que derruba o Consolidado e mostra lançamentos sendo aceitos e, depois, o saldo convergindo.

**NFR do enunciado 2:** "em pico, o consolidado recebe 50 req/s, com no máximo 5% de perda".

- Leitura do PO: são **requisições de leitura** do consolidado. A meta é sustentar 50 req/s com taxa de sucesso >= 95% (erro ou *timeout* <= 5%).
- **Limite inegociável:** os 5% nunca se aplicam a eventos de lançamento. Uma leitura falha pode ser repetida pelo integrador; um lançamento perdido é dinheiro sem registro.
- Meta interna do PO: >= 99% de sucesso, deixando os 5% como margem de segurança e não como alvo.
- Prova exigida: teste de carga reproduzível no repositório com o resultado anexado.

**Demais NFRs:**

| Código | Atributo | Meta | Status |
| --- | --- | --- | --- |
| RNF-01 | Disponibilidade de Lançamentos | Independente do Consolidado; meta 99,9% | Enunciado + PO |
| RNF-02 | Vazão do Consolidado (leitura) | 50 req/s sustentados, sucesso >= 95% | Enunciado |
| RNF-03 | Durabilidade | Zero perda de lançamento e de evento confirmado | PO |
| RNF-04 | Latência de registro | p95 <= 300 ms | Premissa do PO |
| RNF-05 | Latência de consulta do consolidado | p95 <= 200 ms | Premissa do PO |
| RNF-06 | Atraso de consolidação | p95 <= 5 s em operação normal | Premissa do PO |
| RNF-07 | Vazão de Lançamentos | Não informada; dimensionar para 50 escritas/s com folga de 2x | Premissa do PO |
| RNF-08 | Escalabilidade | Ambos os serviços escalam horizontalmente, sem estado local | PO |
| RNF-09 | Recuperação | RPO de Lançamentos = 0; RTO de Lançamentos <= 15 min; RTO do Consolidado <= 60 min; Consolidado reconstruível a partir do livro | PO |
| RNF-10 | Segurança | Autenticação em toda API, autorização por comerciante, criptografia em trânsito (TLS) e em repouso (bancos, mensageria e backups), segredos fora do código | PO |
| RNF-11 | Privacidade | Descrição do lançamento não deve conter dado pessoal; LGPD tratada no desenho | PO |
| RNF-12 | Auditabilidade | Todo lançamento rastreável a usuário, horário e requisição | PO |
| RNF-13 | Observabilidade | Métricas, logs estruturados e rastreamento distribuído com ID de correlação ponta a ponta | PO |
| RNF-14 | Testabilidade | Testes unitários, de integração, de contrato, de resiliência e de carga automatizados | Enunciado + PO |
| RNF-15 | Execução local | Ambiente completo sobe com um único comando | Enunciado (README) |

**Resiliência no desenho alvo (exigência do PO para RNF-09):**

- Sem ponto único de falha no caminho de escrita: mais de uma instância do serviço de Lançamentos e armazenamento com réplica e failover automático.
- Mensageria com persistência e replicação; evento confirmado não se perde se um nó cair.
- Backup diário dos dois bancos, retido pelo prazo da RN-13 e com restauração testada ao menos uma vez antes de produção.
- Monitoramento proativo: os alertas da seção de operação disparam antes de o SLO ser violado.
- RTO do Consolidado é maior que o de Lançamentos de propósito: enquanto ele está fora, o caixa continua registrando (NFR 1 do enunciado).
- No ambiente local a redundância não é exigida; o desenho alvo mostra como ela existe em produção.

Os RTOs são premissas do PO; o Arquiteto pode revisar com justificativa em ADR.

Todas as metas marcadas como **premissa do PO** podem ser revistas pelo Arquiteto, desde que ele registre a nova meta e o motivo em ADR.

## Operação: SLOs, observabilidade, segurança e privacidade

Os NFRs só valem se forem medidos. Esta seção diz o que o PO quer enxergar e controlar; ferramentas são escolha do Arquiteto.

### SLOs e orçamento de erro

| SLO | Indicador | Meta | Orçamento de erro |
| --- | --- | --- | --- |
| SLO-01 | Requisições de registro bem-sucedidas | 99,9% em 30 dias | 43,2 min de indisponibilidade por mês |
| SLO-02 | Leituras do consolidado bem-sucedidas | >= 99% (piso do enunciado: 95%) | 1% das leituras |
| SLO-03 | Atraso de consolidação | p95 <= 5 s | 5% dos eventos acima de 5 s |
| SLO-04 | Divergência na conciliação | 0 | Nenhum |

Consumo acelerado do orçamento de SLO-01 é motivo para parar novas mudanças, não apenas para alertar.

### Métricas de negócio obrigatórias

- Lançamentos registrados por minuto, por tipo.
- Estornos por hora.
- Respostas de idempotência (reenvios reconhecidos).
- Eventos publicados e ainda não consolidados (fila pendente), por idade do mais antigo.
- Eventos rejeitados pelo consumidor (fila de falhas).
- Atraso de consolidação, em percentis.
- Divergências encontradas pela conciliação.
- Acessos negados por isolamento de comerciante.

### Alertas mínimos

| Alerta | Condição proposta | Severidade |
| --- | --- | --- |
| Escrita degradada | Taxa de erro de registro > 0,5% por 5 min | Crítica |
| Consolidação atrasada | p95 do atraso > 5 s por 10 min | Alta |
| Fila pendente crescendo | Evento mais antigo com mais de 5 min | Alta |
| Evento em fila de falhas | Qualquer quantidade > 0 | Alta |
| Divergência de conciliação | Qualquer valor > 0 | Crítica |
| Leitura degradada | Sucesso < 99% por 5 min | Média |

Limiares são premissas do PO; o Arquiteto ajusta e registra.

### Logs e rastreamento

- Logs estruturados, com ID de correlação, serviço, operação e resultado.
- Descrição do lançamento, credenciais e tokens nunca aparecem em log.
- Rastreamento distribuído cobrindo também o trecho assíncrono: da requisição até a atualização do saldo.
- Dois painéis: um técnico (latência, erro, saturação) e um de negócio (volume, atraso, divergências).

### Perfis de acesso

| Perfil | Pode | Não pode |
| --- | --- | --- |
| Operador de caixa | Registrar, estornar e consultar do seu comerciante | Reconstruir, ver outro comerciante |
| Gestor | Consultar lançamentos e consolidado do seu comerciante | Registrar e estornar |
| Administrador da plataforma | Disparar reconstrução e conciliação | Registrar lançamento em nome de comerciante |

### Segurança: superfícies que o desenho deve cobrir

| Fronteira | Exigência do PO |
| --- | --- |
| Integrador para API | Autenticação por padrão aberto de mercado, TLS, limite de requisições por integrador, validação estrita de entrada |
| Serviço para serviço | Identidade própria de cada serviço; nenhum serviço confia em chamada anônima |
| Serviço para mensageria | Credencial por serviço; produtor só publica, consumidor só consome; mensagens criptografadas em trânsito e em repouso |
| Serviço para banco | Credencial por serviço, menor privilégio, banco de um serviço inacessível ao outro, dado e backup criptografados em repouso |
| Segredos | Fora do código e do repositório; o repositório público contém só valores de exemplo marcados como tais |
| Cadeia de dependências | Verificação de vulnerabilidade de dependências e de imagem no pipeline |
| Operação administrativa | Perfil separado e trilha de auditoria própria |

Entregável esperado: modelo de ameaças STRIDE por fronteira, uma linha por ameaça, com mitigação e risco residual.

### Privacidade (LGPD)

- O comerciante pode ser pessoa física (empresário individual ou MEI); nesse caso, seus dados de caixa e identificação são dados pessoais.
- O usuário operador é sempre pessoa física: nome e e-mail na trilha de auditoria são dados pessoais.
- Bases legais prováveis: execução de contrato (art. 7º, V) para a operação e cumprimento de obrigação legal (art. 7º, II) para a retenção da RN-13.
- Minimização: descrição fora do evento e dos logs; comerciante identificado por ID técnico nos fluxos internos.

### Procedimentos operacionais (runbooks) mínimos

1. Consolidado fora do ar: confirmar que a escrita segue, acompanhar a fila pendente, religar, medir a convergência.
2. Evento em fila de falhas: identificar a causa, corrigir, reprocessar sem duplicar.
3. Divergência de conciliação: isolar comerciante e dia, reconstruir a projeção, registrar causa.
4. Reconstrução completa: quando usar, quanto tempo leva no volume de teste, como validar o resultado.
5. Falha de instância ou de banco de Lançamentos: como o failover acontece, como confirmar que nenhum lançamento confirmado se perdeu, dentro do RTO de 15 min.
6. Restauração de backup: passo a passo, tempo medido no teste e validação por conciliação.

## Backlog: épicos, histórias e critérios de aceite

Quatro épicos. Os critérios de aceite estão em formato Dado / Quando / Então para virarem testes automatizados quase sem tradução.

### EP-01 Lançamentos

**US-01 Registrar crédito.** Como comerciante, quero registrar um crédito para que meu caixa reflita a venda.

- Dado um usuário autenticado do comerciante A, quando envia crédito de R$ 150,00 com data de hoje, então recebe 201 com identificador e o lançamento aparece na listagem.
- Dado valor zero, negativo ou com mais de 2 casas, quando envia, então recebe 422 com o campo inválido identificado.
- Dado data de competência futura, quando envia, então recebe 422 (RN-08).

**US-02 Registrar débito.** Mesmos critérios de US-01 com tipo DÉBITO; o saldo do dia é reduzido após consolidação.

**US-03 Estornar lançamento.** Como comerciante, quero corrigir um lançamento errado sem apagar o histórico.

- Dado um crédito de R$ 150,00, quando solicita estorno, então é criado um débito de R$ 150,00 vinculado ao original e o original permanece consultável.
- Dado um lançamento já estornado, quando solicita novo estorno, então recebe 409 (RN-04).
- Dado um lançamento de outro comerciante, quando solicita estorno, então recebe 404, sem revelar que ele existe (RN-11).

**US-04 Listar lançamentos.** Como comerciante, quero ver os lançamentos de um período.

- Dado um intervalo de datas, quando consulta, então recebe só os lançamentos do seu comerciante, paginados, em ordem de data de competência e registro.

**US-05 Não duplicar em retentativa.** Como integrador, quero reenviar uma requisição sem criar lançamento em dobro.

- Dada a mesma chave de idempotência e o mesmo conteúdo, quando reenvia, então recebe o mesmo lançamento e nenhum novo é criado.
- Dada a mesma chave com conteúdo diferente, quando envia, então recebe 409 ou 422.

### EP-02 Consolidado Diário

**US-06 Consultar saldo do dia.** Como comerciante, quero saber o saldo de um dia.

- Dado saldo final de ontem R$ 1.000,00 e hoje crédito R$ 500,00 e débito R$ 200,00, quando consulta hoje, então vê saldo inicial 1.000,00, créditos 500,00, débitos 200,00, saldo final 1.300,00 e a hora da última atualização.
- Dado um dia sem lançamentos, quando consulta, então vê movimento zero e saldo final igual ao inicial (RN-12).

**US-07 Extrato consolidado por período.** Como comerciante, quero ver o saldo dia a dia de um período.

- Dado um intervalo de até 366 dias, quando consulta, então recebe um item por dia, sem lacunas.

**US-08 Lançamento retroativo.** Como comerciante, quero lançar algo que esqueci e ver os saldos corrigidos.

- Dado saldo consolidado de D-3 até hoje, quando registra crédito de R$ 100,00 em D-3, então os saldos finais de D-3 até hoje aumentam R$ 100,00 dentro do atraso máximo aceito.

**US-09 Evento repetido.** Como sistema, quero que um evento entregue duas vezes conte uma vez só.

- Dado um evento já processado, quando chega de novo, então o saldo não muda.

### EP-03 Resiliência e operação

**US-10 Consolidado fora do ar.** Como comerciante, quero continuar registrando mesmo se o consolidado diário falhar.

- Dado o Consolidado parado, quando registro 100 lançamentos, então todos retornam sucesso; quando o Consolidado volta, então o saldo converge para a soma dos 100 sem perda.

**US-11 Pico de leitura.** Dado 50 req/s de consulta ao consolidado durante 5 minutos, então a taxa de sucesso é >= 95% e o resultado do teste fica no repositório.

**US-12 Reconstrução.** Dado o consolidado apagado, quando executo a reconstrução, então os saldos são iguais aos anteriores.

**US-13 Conciliação.** Dado um saldo consolidado divergente da soma dos lançamentos, quando a conciliação roda, então um alerta é emitido com o comerciante e o dia afetados.

**US-14 Observabilidade.** Dado um lançamento registrado, então consigo seguir pelo mesmo ID de correlação a requisição, o evento e a atualização do saldo.

### EP-04 Segurança e acesso

**US-15 Autenticação.** Dada uma requisição sem credencial válida, então recebe 401.

**US-16 Isolamento por comerciante.** Dado um usuário do comerciante A, quando consulta dados do comerciante B, então recebe 404 e o acesso negado fica registrado.

**US-17 Proteção contra abuso.** Dado um integrador acima do limite de requisições, então recebe 429 sem afetar os demais integradores.

## Priorização e escopo do MVP

O critério de corte é simples: primeiro tudo que, se faltar, descarta o teste; depois o que prova os NFRs; por último o que diferencia.

| Prioridade | Itens | Critério |
| --- | --- | --- |
| Must | US-01 a US-07, US-09, US-10, US-11, US-14, US-15, US-16 | Obrigatório do enunciado ou prova direta do NFR; US-05 e US-14 promovidas por custo baixo e sinal alto |
| Should | US-08, US-12 | Diferencia em banco: retroativo e reconstrução; entram só se o portão G2 for cumprido |
| Could | US-13, US-17, interface web simples | Ganho real, mas não muda a nota mínima; cortados para sexta e documentados como evolução |
| Won't (agora) | Itens da lista abaixo | Registrados como evolução futura |

**Fora de escopo, registrar no README como evolução futura:**

- Fechamento de dia (bloquear lançamentos em dia já fechado pelo contador).
- Conciliação bancária e integração com extrato, PIX e adquirentes.
- Plano de contas e categorização contábil formal.
- Multimoeda.
- Lançamentos agendados ou recorrentes.
- Exportação para contabilidade (ex.: arquivo para o escritório contábil).
- Aplicativo móvel.

**Decisão do PO para o prazo de sexta 08:00:**

- Entra tudo que é Must, incluindo idempotência e rastreio ponta a ponta.
- US-08 (retroativo) e US-12 (reconstrução) entram apenas se o portão G2, quinta 12:00, for cumprido com todos os Must verdes. Se o G2 falhar, ambos são cortados sem discussão e viram ADR de evolução.
- Mesmo cortado o código, RN-07 e RN-15 continuam valendo no desenho: o Arquiteto mostra como o retroativo seria tratado.
- Could fica fora. A arquitetura de transição e a estimativa de custo entram como documento, porque custam pouco e são diferenciais pedidos.

**Regra de escopo para o Arquiteto:** se o tempo apertar, corta-se de baixo para cima. Nenhum item Should entra antes de todos os Must estarem testados e documentados.

## Indo além: diferenciais de produto

O enunciado avisa que quer ver tomada de decisão, não só código. Estes pontos mostram leitura de negócio financeiro que um candidato comum não traz. Cada um custa pouco e conversa direto com quem avalia em banco.

| Diferencial | Por que pesa para um banco | Custo estimado |
| --- | --- | --- |
| Livro imutável com estorno | É assim que razão contábil e core bancário funcionam; edição de registro financeiro é sinal de alerta. | Baixo |
| Separação "5% de leitura" vs "0% de lançamento" | Mostra que o candidato leu o NFR com critério de risco, não ao pé da letra. | Zero (decisão escrita) |
| Idempotência na escrita | Retentativa sem duplicidade é requisito básico de qualquer API de pagamento. | Baixo |
| Saldo acumulado com saldo inicial | Consolidado diário sem saldo inicial não fecha com o extrato real. | Baixo |
| Lançamento retroativo com recálculo | Situação real de comércio (nota esquecida); expõe o problema de ordem de eventos. | Médio |
| Consolidado reconstruível | Transforma a queda do Consolidado em evento operacional, não em incidente de dado. | Médio |
| Conciliação automática | Controle de integridade contínuo, linguagem de auditoria e controles internos. | Baixo |
| Teste de caos e de carga no repositório | Evidência em vez de promessa: o avaliador roda e vê. | Médio |
| Arquitetura de transição | O enunciado pede como diferencial; banco sempre tem legado. Proposta do PO: comerciante sai de planilha, importa histórico e opera em paralelo por um período antes do corte. | Baixo (documento) |
| Estimativa de custo | Pedido como diferencial; mostra visão de dono. Comparar ao menos dois tamanhos de ambiente. | Baixo (documento) |
| Modelo de ameaças | Critérios de segurança para integração são diferencial pedido; uma análise STRIDE por fronteira responde a isso de forma organizada. | Baixo (documento) |
| SLOs com orçamento de erro | Traduz os NFRs para a linguagem que operação de banco usa. | Baixo (documento) |

## Arquitetura de transição: requisitos do PO

O enunciado não descreve o legado, então o PO fixa um cenário e o Arquiteto desenha a transição sobre ele. Entra como documento e diagrama; a implementação da importação fica fora do prazo de sexta.

**Cenário de legado assumido (premissa do PO):** o comerciante controla o caixa em planilha, uma linha por movimento, com data, descrição, tipo e valor. Há histórico de meses e um saldo atual que ele considera correto.

**Requisitos da transição:**

| Código | Requisito |
| --- | --- |
| TR-01 | Importar o histórico em lote a partir de arquivo CSV com layout publicado. |
| TR-02 | Validar cada linha pelas mesmas regras da API (RN-01, RN-02, RN-16, RN-17) e devolver a lista de linhas rejeitadas com o motivo. |
| TR-03 | Importação idempotente: reimportar o mesmo arquivo não duplica lançamentos. |
| TR-04 | Alternativa à importação completa: registrar só o saldo de abertura (RN-19) e começar do zero. |
| TR-05 | Operação em paralelo por período definido (proposta: 14 dias), com comparação diária entre o saldo da planilha e o do RAZÃO. |
| TR-06 | Critério de corte: 5 dias úteis seguidos sem divergência. Depois disso, a planilha vira somente leitura. |
| TR-07 | Plano de volta: até o corte, a planilha continua sendo a referência oficial. |

**Entregáveis esperados do Arquiteto:** diagrama do estado atual, do estado de transição (convivência) e do estado alvo; e um parágrafo sobre como o mesmo padrão valeria se o legado fosse um sistema, com substituição gradual por funcionalidade.

## Estimativa de custos: o que o PO exige ver

O PO não aceita número sem premissa. A estimativa deve permitir que um diretor entenda quanto custa operar e o que faz o custo subir.

**Conteúdo obrigatório:**

1. Premissas de volume explícitas (comerciantes, lançamentos por dia, leituras por segundo em pico, retenção de 5 anos).
2. Três cenários: ambiente local (custo zero, para avaliação); produção mínima com alta disponibilidade; produção em pico sustentado de 50 leituras/s.
3. Custo por componente: computação, banco, mensageria, armazenamento, rede, observabilidade, segredos, licenças.
4. Licenças: preferência por software de licença aberta; qualquer licença paga justificada.
5. Região de referência informada (proposta: São Paulo, por proximidade e residência do dado) e data da consulta de preço.
6. Fonte de cada preço: calculadora oficial do provedor, com link.
7. Custo unitário: custo por comerciante ativo por mês e custo por milhão de lançamentos.
8. Alavancas de redução: o que cortar primeiro se o orçamento for menor, e o risco de cada corte.

Valores devem ser apresentados como estimativa com data, nunca como fato.

## Matriz de conformidade com o desafio

O enunciado diz que o teste é descartado se os obrigatórios não forem minimamente atendidos. Esta lista é a Definition of Done: nada é entregue com caixa aberta na parte obrigatória.

**Obrigatórios (descarte se faltar):**

- [ ] Mapeamento de domínios funcionais e capacidades de negócio, versionado no repositório
- [ ] Requisitos funcionais e não funcionais refinados, com a interpretação dos NFRs escrita
- [ ] Desenho da arquitetura alvo completo (contexto, contêineres e fluxos principais)
- [ ] Justificativa de tecnologias e do estilo arquitetural, uma ADR por decisão relevante
- [ ] Código dos dois serviços na linguagem escolhida
- [ ] Testes automatizados (unitários e de integração no mínimo), executáveis por um comando
- [ ] README com o que a aplicação faz, como funciona e como rodar localmente
- [ ] Repositório público no GitHub
- [ ] Toda a documentação dentro do repositório, incluindo este handoff exportado em Markdown para docs/01-negocio/handoff-po.md

**Prova dos NFRs (sem isso o requisito fica só declarado):**

- [ ] Teste que derruba o Consolidado e mostra Lançamentos operando
- [ ] Teste de carga de 50 req/s no Consolidado com resultado da taxa de sucesso

**Diferenciais pedidos pelo enunciado:**

- [ ] Arquitetura de transição (migração de legado)
- [ ] Estimativa de custos de infraestrutura e licenças
- [ ] Monitoramento e observabilidade
- [ ] Critérios de segurança para integração entre serviços
- [ ] Seção de evoluções futuras e do que ficou de fora

**Critério de pronto do PO:** um avaliador que nunca viu o projeto clona o repositório, sobe tudo com um comando, roda os testes e encontra cada item acima em até 10 minutos de leitura.

## Estrutura do repositório, README e ADRs

O avaliador precisa achar cada obrigatório sem procurar. **Modelo de organização, ajustável pelo Arquiteto** desde que a navegação continue óbvia:

```text
razao/
  README.md
  docs/
    01-negocio/         handoff-po.md (este documento), glossario, dominios
    02-requisitos/      regras, RF, RNF, historias, casos de borda
    03-arquitetura/     alvo, transicao, fluxos, contratos
    04-adr/             0001-estilo-arquitetural.md, ...
    05-seguranca/       modelo de ameacas
    06-operacao/        SLOs, observabilidade, runbooks
    07-custos/          estimativa e premissas
    08-evolucoes.md     o que ficou de fora e por que
  servicos/
    lancamentos/
    consolidado/
  testes/
    carga/              roteiro e resultado do teste de 50 req/s
    resiliencia/        roteiro e resultado do teste de queda
  <arquivo de orquestracao local>
```

**Roteiro obrigatório do README:**

1. O problema em duas frases e o que o RAZÃO resolve.
2. Mapa para o avaliador: tabela "requisito do desafio" e "onde está" (link direto para cada item da matriz de conformidade).
3. Arquitetura em um diagrama, com link para o detalhamento.
4. Pré-requisitos com versões exatas.
5. Como subir tudo em no máximo três comandos (meta: um).
6. Como rodar cada tipo de teste, incluindo carga e queda, e onde ver o resultado.
7. Exemplos de uso prontos para copiar (registro, estorno, consulta do consolidado).
8. Interpretação dos dois NFRs do enunciado, em um parágrafo.
9. Decisões principais com link para as ADRs.
10. Limitações conhecidas e o que seria feito em produção.

**Modelo de ADR (estrutura mínima):**

Regras de escrita: uma decisão por ADR, primeira pessoa ("Decidimos", "Descartei"), e a seção "Como provamos" sempre aponta para um teste que existe. ADR sem teste citado não está pronta.

```markdown
# ADR-000X: <decisão em uma frase>

Status: proposta | aceita | substituída por ADR-00YY
Data: AAAA-MM-DD
Regras e requisitos afetados: RN-xx, I-xx, C-xx, RF-xx, RNF-xx

## Contexto
O problema e as forças em jogo, em poucas frases.

## Decisão
Decidimos X porque Y.

## Alternativas descartadas
Descartei Z porque W. (pelo menos duas, com o motivo real)

## Consequências aceitas
Ganhamos X, pagamos Y. O que muda se a premissa cair.

## Como provamos
Caminho do teste existente no repositório que comprova a decisão.
```

**ADRs mínimas esperadas:** estilo arquitetural e número de implantáveis; integração assíncrona e garantia entre gravação e publicação; armazenamento de cada serviço; estratégia de leitura para 50 req/s; tratamento de ordem e de retroativo; representação de valor monetário; autenticação e autorização; observabilidade; estratégia de testes, incluindo carga e queda; o que ficou de fora para sexta.

**Idioma:** documentação em português. Termos de domínio no código seguem o glossário.

## Matriz de rastreabilidade

Cada história aponta para o que a sustenta e para o teste que a prova. No G1, o Arquiteto devolve esta tabela com a coluna "Componente" preenchida.

| História | Requisitos | Regras e invariantes | Teste que prova | Componente |
| --- | --- | --- | --- | --- |
| US-01 | RF-01 | RN-01, RN-02, RN-08, RN-16, RN-17, RN-20 | Unitário de domínio + integração de API |  |
| US-02 | RF-01 | RN-01, RN-02 | Unitário de domínio + integração de API |  |
| US-03 | RF-02 | RN-03, RN-04, RN-15, I-03 a I-06 | Integração + concorrência de estorno |  |
| US-04 | RF-04 | RN-11, RN-18 | Integração de API |  |
| US-05 | RF-01 | RN-10, I-07 | Integração com reenvio |  |
| US-06 | RF-07 | RN-05, RN-06, RN-12, C-01, C-02 | Teste de referência do exemplo de cálculo |  |
| US-07 | RF-08 | RN-12 | Integração de API |  |
| US-08 | RF-09 | RN-07, RN-15, C-03, C-04 | Exemplo de cálculo com eventos fora de ordem |  |
| US-09 | RF-06 | C-04 | Integração com evento duplicado |  |
| US-10 | RF-05, RNF-01, RNF-03 | C-05 | Teste de resiliência com Consolidado parado |  |
| US-11 | RNF-02 | SLO-02 | Teste de carga 50 req/s por 5 min |  |
| US-12 | RF-10, RNF-09 | C-03 | Reconstrução e comparação |  |
| US-13 | RF-11 | C-03, SLO-04 | Divergência provocada |  |
| US-14 | RNF-13 | - | Verificação de rastreio ponta a ponta |  |
| US-15 | RNF-10 | - | Integração de segurança |  |
| US-16 | RNF-10 | RN-11 | Integração de segurança com dois comerciantes |  |
| US-17 | RNF-10 | - | Integração com excesso de requisições |  |

## Handoff para o Arquiteto de Soluções

O PO decide **o quê** e **por quê**. O Arquiteto decide **como**. Esta seção separa as duas coisas para não haver retrabalho nem decisão técnica disfarçada de requisito.

**Já decidido pelo PO (não reabrir sem falar com o PO):** RN-01 a RN-20, as invariantes I-01 a I-07 e C-01 a C-05, o exemplo de cálculo de referência, os casos de borda, a interpretação dos dois NFRs do enunciado, a fronteira entre os domínios Lançamentos e Consolidado, a priorização MoSCoW e a Definition of Done.

**Decisões que pertencem ao Arquiteto (cada uma vira ADR):**

1. Estilo arquitetural e quantos implantáveis existem (o PO exige só que Lançamentos não dependa do Consolidado em tempo de execução).
2. Linguagem, frameworks e banco de cada serviço.
3. Mecanismo de integração assíncrona e como garantir que gravação do lançamento e publicação do evento não divirjam.
4. Estratégia de ordenação e de recálculo para lançamentos retroativos.
5. Estratégia para sustentar 50 req/s de leitura (cache, réplica, pré-cálculo) e o que acontece quando o cache está frio.
6. Modelo de autenticação e autorização entre usuário e serviços e entre serviços.
7. Pilha de observabilidade e quais métricas de negócio são expostas.
8. Ambiente local (um comando) e ambiente alvo em nuvem, com estimativa de custo.
9. Ferramentas de teste de carga e de resiliência.
10. Desenho da arquitetura de transição a partir de um legado presumido (planilha ou sistema antigo).

**Premissas do PO a validar pelo Arquiteto:**

| Premissa | Valor proposto | Se o Arquiteto discordar |
| --- | --- | --- |
| Atraso de consolidação | p95 <= 5 s | Registrar novo valor e motivo em ADR |
| Vazão de escrita em pico | 50 escritas/s com folga de 2x | Idem |
| Janela de retroativo | 90 dias | Idem |
| Período máximo de extrato | 366 dias | Idem |
| Latências p95 | 300 ms escrita, 200 ms leitura | Idem |
| Legado atual | Planilha | Descrever o legado assumido |

**Perguntas em aberto que o Arquiteto deve responder por escrito:**

- Como o sistema prova, para um auditor, que nenhum evento foi perdido entre Lançamentos e Consolidado?
- O que o comerciante vê quando consulta o consolidado durante um recálculo retroativo?
- Qual é o comportamento se o Consolidado ficar fora por horas e acumular volume grande de eventos?
- Qual é o primeiro gargalo esperado acima de 50 req/s e qual o plano para ele?
- Como um novo serviço consumidor (ex.: uma apuração fiscal) passaria a usar os eventos sem tocar em Lançamentos?

**Entregáveis esperados do Arquiteto de volta ao PO:** diagramas de contexto e de contêineres, fluxos de registro e de consolidação, ADRs, modelo de dados de cada serviço, contrato das APIs e do evento, estratégia de testes, plano de observabilidade, modelo de ameaças, estimativa de custo e arquitetura de transição.

## Critérios de revisão do desenho pelo PO (portão G1)

No G1 o PO responde cada pergunta com sim ou não. Um "não" nos itens 1 a 6 bloqueia o início do código.

1. Existe algum caminho em que o registro de lançamento espera o Consolidado, direta ou indiretamente? (esperado: não)
2. Está explicado como gravação do lançamento e publicação do evento nunca divergem, inclusive se o processo cair entre as duas?
3. O consumidor é idempotente e imune a ordem, e isso está ligado a um teste?
4. A estratégia de 50 leituras/s tem número de capacidade estimado e plano para cache frio?
5. O comerciante vem sempre da credencial, em todos os endpoints, e o dado está criptografado em trânsito e em repouso?
6. Cada RN e cada invariante tem lugar definido onde é aplicada?
7. O consolidado é reconstruível, com passo a passo, e o desenho mostra redundância, failover, backup e os RTOs?
8. Existe ADR para cada decisão mínima listada?
9. O ambiente local sobe com um comando?
10. O plano cabe até sexta 08:00, com o que será cortado no G2 já identificado?

**Resultado do G1:** aprovado, aprovado com ressalvas (lista de ajustes com prazo), ou refazer.

## Pauta da sessão de esclarecimento (30 minutos)

| Tempo | Tema | Saída |
| --- | --- | --- |
| 0 a 5 min | Objetivo, prazo de sexta 08:00 e os três portões | Datas confirmadas |
| 5 a 15 min | Regras que mais mudam o desenho: RN-03, RN-06, RN-07, RN-10, RN-15 e a leitura dos NFRs | Dúvidas resolvidas ou registradas |
| 15 a 25 min | Premissas do PO e perguntas em aberto; o que é decisão do Arquiteto | Premissas aceitas ou contestadas |
| 25 a 30 min | Combinados: hora do G1, formato da entrega do desenho, canal de dúvidas | Ata de até 5 linhas |

A ata é registrada como comentário neste documento, para não depender de memória.

## Cronograma até a entrega

Cada portão tem dono e decisão: G1 libera o código, G2 decide se o Should entra, G3 congela o código e deixa só documentação e verificação.

&#91;embedded content: roadmap até a entrega · 5 fases, 3 portões\]

| Portão | Quando | Dono | Critério | Se falhar |
| --- | --- | --- | --- | --- |
| G1 | qua 21/10 12:00 | PO | Critérios de revisão do desenho, itens 1 a 6 em "sim" | Ajuste em até 2 h; código só começa depois |
| G2 | qui 22/10 12:00 | PO | Todos os Must implementados e com testes verdes | US-08 e US-12 cortados; tarde dedicada a provas de NFR |
| G3 | qui 22/10 22:00 | Arquiteto | Código congelado; testes de carga e queda executados | Corta-se a funcionalidade inacabada, nunca a documentação |
| Entrega | sex 23/10 08:00 | Arquiteto | Repositório público validado em máquina limpa | Sem margem: o G3 existe para evitar este cenário |

A fase Must vai do G1 aprovado na quarta até quinta 12:00, incluindo a noite de quarta. É a fase com menor folga.

## Riscos do projeto

O maior risco não é técnico: é gastar o tempo do teste em sofisticação e deixar um obrigatório aberto.

| Risco | Impacto | Probabilidade | Mitigação |
| --- | --- | --- | --- |
| Obrigatório incompleto (README, testes, docs fora do repositório) | Descarte do teste | Média | Definition of Done revisada antes da entrega |
| Excesso de engenharia (muitos serviços, ferramentas e camadas) | Prazo estourado, código raso | Alta | Corte MoSCoW de baixo para cima; ADR explica o que foi deixado para depois |
| NFR declarado sem prova | Avaliador desconsidera o requisito | Média | Teste de queda e teste de carga no repositório, com resultado |
| Leitura literal dos 5% aplicada a lançamentos | Sinal de risco financeiro para o avaliador | Baixa, se este documento for seguido | Regra explícita em README e ADR |
| Ambiente local que não sobe na máquina do avaliador | Avaliação parcial ou nula | Média | Um comando, versões fixadas, teste em máquina limpa |
| Consistência eventual não explicada | Pergunta difícil na entrevista técnica | Média | Hora da última atualização exposta e limite de atraso documentado |
| Prazo curto: entrega sexta 08:00 | Must incompleto na entrega | Alta | Portões G1, G2 e G3; corte automático do Should se o G2 falhar |

**Dado faltante:** a hora exata da entrega na sexta. Por que importa: define a margem do congelamento de código. Conduta provisória: entregar até 08:00 e confirmar a hora com quem enviou o teste.

## Próximos passos

| Passo | Responsável | Quando | Situação |
| --- | --- | --- | --- |
| Confirmar o prazo de entrega | PO | 20/10 | Concluído: sexta 23/10 pela manhã |
| Entregar este documento e fazer a sessão de 30 min | PO e Arquiteto | Até 20/10 22:00 | Aceito |
| Devolver desenho alvo, contratos e ADRs v1 | Arquiteto | Até 21/10 11:00 | Aceito |
| Validar no G1 contra a matriz e as premissas | PO | 21/10 12:00 | Aceito |
| Implementar na ordem da priorização | Arquiteto | A partir do G1 aprovado | Aceito |
