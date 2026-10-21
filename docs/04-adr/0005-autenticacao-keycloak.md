# ADR-0005: Autenticação e autorização com Keycloak

Status: aceita
Data: 2026-10-20
Regras e requisitos afetados: RN-11, RNF-10, US-15, US-16

## Contexto

Todo dado pertence a um comerciante, e um usuário nunca lê nem escreve dado de outro (RN-11). O avaliador precisa ver isso funcionar no ambiente local.

## Decisão

Decidimos usar o Keycloak como provedor OAuth2 e OIDC, igual no Docker Compose e no GKE. O comerciante vem sempre da credencial, nunca do corpo nem da rota. Os perfis são Operador, Gestor e Administrador da plataforma, como no handoff. Entre serviços, cada um tem identidade própria por credencial de cliente.

Tráfego externo com TLS; bancos, mensageria e backups criptografados em repouso no ambiente alvo.

## Alternativas descartadas

- Identity Platform na GCP em produção e Keycloak local: descartei porque seriam dois comportamentos, e o teste local não provaria o de produção.
- Autenticação própria nos serviços: descartei porque é problema resolvido pelo mercado e fonte conhecida de falha de segurança.

## Consequências aceitas

Ganhamos o mesmo controle de acesso em todos os ambientes. Pagamos a operação do Keycloak. Acesso a dado de outro comerciante devolve 404, sem revelar que o dado existe, e fica auditado.

## Como provamos

Testes de integração de segurança: requisição sem credencial recebe 401; operador do comerciante A consultando dado do comerciante B recebe 404 (testes previstos).
