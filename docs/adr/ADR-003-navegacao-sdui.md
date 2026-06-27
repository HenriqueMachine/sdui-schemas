# ADR-003 — Navegação SDUI por rota nomeada

**Status:** Aceito  
**Data:** 2026-06-26

## Contexto

Em SDUI, a navegação entre telas pode ser controlada pelo servidor (rota, parâmetros, comportamento de pilha) ou pelo cliente (navegação local imediata). Precisamos decidir o modelo de controle e o catálogo de rotas.

## Decisão

### Catálogo de rotas fechado no cliente

O servidor envia o **nome** da rota, não uma URL. O cliente possui um registry de rotas fixo implementado com `go_router`. Rotas desconhecidas exibem snackbar e mantêm a tela atual sem crash.

**Catálogo inicial de rotas:**

| Nome | Descrição |
|---|---|
| `home` | Tela principal com saldo e atalhos |
| `login` | Tela de login (reservada, sem auth real nessa fase) |
| `transfer` | Seleção de contato e valor |
| `transfer_confirm` | Revisão antes de confirmar |
| `transfer_success` | Recibo após transferência bem-sucedida |
| `extract` | Histórico de transações |

### Pilha de navegação

A pilha vive no Flutter Navigator (cliente). O pop é **local, instantâneo e offline-friendly** — não depende de chamada ao servidor.

### `backBehavior` declarativo

A raiz de cada tela pode declarar o comportamento do botão voltar:

```json
{ "backBehavior": "DEFAULT" }
{ "backBehavior": "CONFIRM_DISCARD" }
```

O campo é uma string enum aberta no schema (aceita valores futuros sem breaking change).

### Ações transacionais — Estratégia B (resposta inline)

`POST /transfers` retorna `{ result, nextScreen }` com a árvore SDUI da próxima tela já embutida na resposta. Após sucesso, o cliente usa `pushReplacement` (substituindo a tela de confirmação pela de sucesso na pilha, sem permitir voltar para confirmação).

### Deeplinks

Deeplinks externos reutilizam o mesmo `RouteRegistry` — não há caminho paralelo de navegação.

## Consequências

### Positivas
- Pop é instantâneo e offline — não depende do servidor.
- Catálogo fechado previne navegação para rotas arbitrárias injetadas via payload.
- Estratégia B elimina um round-trip: a tela de sucesso chega embutida na resposta da transferência.

### Negativas
- Adicionar uma nova rota exige atualização do app (nova versão nas lojas para novas rotas).
- O cliente não pode navegar para uma URL arbitrária — comportamento intencional por segurança.

## Alternativas consideradas

- **Servidor envia URL completa:** mais flexível, mas abre vetor de open redirect e dificulta rastreamento de navegação.
- **Estratégia A (redirecionar após POST):** cliente faz `GET /screens/transfer_success` após POST bem-sucedido. Adiciona um round-trip desnecessário.
