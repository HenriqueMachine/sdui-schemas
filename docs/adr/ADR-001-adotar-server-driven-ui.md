# ADR-001 — Adotar Server-Driven UI (SDUI)

**Status:** Aceito  
**Data:** 2026-06-26

## Contexto

Apps mobile dependem de releases nas lojas (Apple/Google) para alterar qualquer comportamento visual ou de fluxo. Isso cria fricção entre o ciclo de produto e o ciclo de release, tornando experimentos lentos e hotfixes de UI inviáveis sem uma nova versão.

## Decisão

Adotar o padrão **Server-Driven UI (SDUI)**, também conhecido como Backend-Driven Content (BDC):

- O **backend Kotlin é o dono da descrição da UI**: decide quais componentes aparecem, em que ordem, com quais dados e quais ações estão disponíveis.
- O **app Flutter é o interpretador**: recebe uma árvore JSON descrevendo a tela e a renderiza usando um registry de componentes.
- **Lógica de negócio fica no backend**; lógica puramente de UI (campo digitado, item selecionado, toggle local) fica no cliente.
- O app **não tem telas hardcoded** — tem um `ComponentRegistry` e um `SDUIRenderer` que percorre árvores JSON e produz Widgets Flutter.

## Consequências

### Positivas
- Alterações de layout, copy e fluxo de navegação não exigem nova versão nas lojas.
- O servidor pode personalizar a UI por usuário, experimento ou versão de app.
- O contrato JSON é versionado e validado — divergências são detectadas cedo.

### Negativas
- O app depende de conectividade para renderizar telas (sem cache nessa fase).
- Debugging é mais complexo: um bug de UI pode estar no servidor (payload errado) ou no cliente (renderer com bug).
- Componentes novos exigem atualização do app — apenas a composição é dinâmica, não o código dos widgets.

## Alternativas consideradas

- **Telas hardcoded com API de dados:** mais simples, mas não resolve o problema de release cycle.
- **Web views em telas nativas:** resolve o release cycle, mas perde a experiência nativa.
- **React Native / Flutter com code push:** plataformas de terceiros com restrições das lojas e risco de policy.
