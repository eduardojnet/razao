# ADR-0008: Valor monetário em decimal exato, trafegado como texto

Status: proposta
Data: 2026-10-20
Regras e requisitos afetados: RN-01, RN-02, RN-17

## Contexto

Valores em reais, com duas casas, em aritmética exata. Ponto flutuante é proibido (RN-02). O JSON não tem tipo decimal: um número em JSON é lido como ponto flutuante por muitos clientes.

## Decisão

Proponho:

- no código, `decimal.Decimal` com duas casas;
- no banco, coluna `NUMERIC(14,2)`, que cobre com folga o teto de R$ 1.000.000,00 (RN-17);
- na API e no evento, o valor como texto decimal com ponto, por exemplo `"150.00"`, validado contra mais de duas casas, zero e negativo.

## Alternativas descartadas

- Centavos em número inteiro: é exato, mas descartei porque obriga todo integrador a converter e esconde a casa decimal de quem lê o contrato.
- Número em JSON: descartei porque clientes comuns o convertem para ponto flutuante.

## Consequências aceitas

Ganhamos precisão de ponta a ponta e contrato legível. Pagamos validação de formato em toda entrada.

## Como provamos

Testes de validação: valor com três casas, zero, negativo e acima do teto devolvem 422 com o campo identificado (testes previstos).
