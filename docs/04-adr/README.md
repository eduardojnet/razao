# Registros de decisão de arquitetura

Uma decisão por ADR. "Aceita" é decisão tomada; "proposta" é mecanismo que ainda depende de aceite ou de medição.

| ADR | Decisão | Status |
| --- | --- | --- |
| [ADR-0001](0001-estilo-arquitetural.md) | Três microsserviços integrados por eventos | aceita |
| [ADR-0002](0002-outbox-transacional.md) | Outbox transacional entre gravação e publicação | aceita |
| [ADR-0003](0003-consumidor-ordem-retroativo.md) | Consumidor imune à ordem e à duplicidade, com recálculo de dia passado | proposta |
| [ADR-0004](0004-leitura-50-req-s.md) | Leitura do consolidado a partir da projeção pré-calculada, com cache | proposta |
| [ADR-0005](0005-autenticacao-keycloak.md) | Autenticação e autorização com Keycloak | aceita |
| [ADR-0006](0006-armazenamento.md) | PostgreSQL por serviço e Redis na leitura do consolidado | aceita |
| [ADR-0007](0007-linguagem.md) | Python 3.12 com FastAPI nos três serviços | aceita |
| [ADR-0008](0008-valor-monetario.md) | Valor monetário em decimal exato, trafegado como texto | proposta |
| [ADR-0009](0009-observabilidade.md) | OpenTelemetry com Cloud Operations | aceita |
| [ADR-0010](0010-estrategia-de-testes.md) | Estratégia de testes, com Locust na carga | aceita para o Locust; proposta para as demais ferramentas |
| [ADR-0011](0011-fora-de-escopo.md) | O que fica de fora da entrega de sexta | proposta, atualizada no G2 |
| [ADR-0012](0012-partidas-dobradas.md) | Escrituração por partidas dobradas com plano de contas | aceita |
| [ADR-0013](0013-falha-integracao-assincrona.md) | Retentativa, tempo limite e fila de falhas na integração por eventos | proposta |
| [ADR-0014](0014-camada-de-entrada-kong.md) | Kong OSS sem banco na camada de entrada | aceita |
| [ADR-0015](0015-arquitetura-hexagonal.md) | Arquitetura hexagonal dentro de cada serviço | aceita |
| [ADR-0016](0016-mensageria-kafka.md) | Kafka como mensageria, com partição por comerciante | aceita |
| [ADR-0017](0017-computacao-gke.md) | GKE Autopilot no ambiente alvo | aceita |
| [ADR-0018](0018-replica-plano-de-contas.md) | Cópia local do plano de contas no Lançamentos | aceita |
| [ADR-0019](0019-sugestao-contrapartida.md) | Sugestão da conta de contrapartida por IA | aceita |
| [ADR-0020](0020-ambiente-local.md) | Ambiente local em Docker Compose | aceita |
| [ADR-0021](0021-infraestrutura-como-codigo.md) | Terraform do ambiente alvo, escrito sem aplicar | aceita |
| [ADR-0022](0022-programacao-orientada-a-objetos.md) | Programação orientada a objetos como paradigma principal | aceita |
