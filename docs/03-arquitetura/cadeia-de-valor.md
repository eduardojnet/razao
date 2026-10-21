# Em que momento o comerciante sabe que o lançamento está garantido?

Na confirmação do registro, que acontece depois da gravação e antes de qualquer consolidação. Tudo à direita desse ponto pode atrasar sem que o lançamento se perca.

```mermaid
flowchart LR
    R["Registrar<br/>(operador ou integrador)"]
    V["Validar<br/>RN-01, RN-02, RN-08, RN-10,<br/>I-08, I-09 pela cópia local do plano"]
    G["Gravar lançamento e evento<br/>na mesma transação"]
    OK(["Confirmar ao operador<br/>201 com identificador"])
    P["Publicar evento<br/>LancamentoRegistrado"]
    C["Consolidar<br/>conta Caixa do dia de competência"]
    S["Servir consolidado diário<br/>com hora da última atualização"]

    R --> V --> G --> OK
    G -. "assíncrono" .-> P --> C --> S
```

| Etapa | Domínio | Se falhar |
| --- | --- | --- |
| Validar | Lançamentos | O operador recebe o erro com a regra violada; nada é gravado |
| Gravar | Lançamentos | O operador recebe erro e repete com a mesma chave de idempotência; não há duplicidade (RN-10) |
| Confirmar | Lançamentos | Ponto de garantia: a partir daqui o lançamento existe e o evento vai sair |
| Publicar | Lançamentos | O evento já está gravado junto com o lançamento e sai quando a publicação voltar (ADR-0002) |
| Consolidar | Consolidado Diário | Os eventos aguardam; o saldo atrasa e converge quando o Consolidado volta (NFR 1) |
| Servir | Consolidado Diário | A leitura falha e o integrador repete; é aqui que valem os 5% do NFR 2 |
