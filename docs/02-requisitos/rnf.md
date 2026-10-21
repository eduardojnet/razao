# Requisitos não funcionais

O enunciado traz dois requisitos não funcionais em uma linha cada. Esta página diz como eu os leio, porque a leitura decide o estilo arquitetural. Os demais RNFs (RNF-01 a RNF-15) estão no [handoff do PO](../01-negocio/handoff-po.md); as metas que eu aceito ou contesto ficam na página de premissas, nesta mesma pasta.

## NFR 1: "o controle de lançamento não deve ficar indisponível se o consolidado cair"

O caminho de registro de lançamento não tem dependência síncrona do Consolidado: nem chamada, nem banco compartilhado, nem fila que bloqueie a gravação (RNF-01, C-05). Estendi a mesma regra ao Plano de Contas e à sugestão de contrapartida: nenhum outro serviço precisa estar no ar para o caixa registrar.

Aceitamos que o consolidado diário fique desatualizado enquanto o Consolidado estiver fora. Os eventos aguardam e são processados quando ele volta, sem perda. O atraso é medido e exposto ao comerciante pela hora da última atualização (RN-14).

Como provamos: teste de queda em `testes/resiliencia`. Com o Consolidado parado, 100 lançamentos retornam sucesso; ao religar, o saldo final converge para a soma dos 100.

## NFR 2: "em pico, o consolidado recebe 50 req/s, com no máximo 5% de perda"

São 50 requisições de leitura por segundo no consolidado, com taxa de sucesso de pelo menos 95%. Erro e tempo esgotado contam como perda (RNF-02, SLO-02).

Os 5% nunca valem para lançamento nem para evento. Uma leitura que falha o integrador repete; um lançamento perdido é dinheiro sem registro. Para lançamento e evento confirmado, a perda é zero (RNF-03).

A meta interna é 99% de sucesso. Os 5% do enunciado ficam como margem de segurança, não como alvo.

Como provamos: teste de carga em `testes/carga`, a 50 requisições por segundo durante 5 minutos, com a taxa de sucesso e o p95 de latência medidos e anexados ao repositório.
