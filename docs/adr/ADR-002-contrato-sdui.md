# ADR-002 — Contrato SDUI: JSON Schema versionado + fallback híbrido

**Status:** Aceito  
**Data:** 2026-06-26

## Contexto

Para que backend e app possam evoluir de forma independente sem quebrar um ao outro, precisamos de um contrato explícito, versionado e validado nos dois lados.

## Decisão

### Transporte

REST. Endpoint por tela: `GET /screens/{nome}`.  
Header de versão enviado pelo cliente em toda chamada: `X-App-Version: <semver>`.

### Formato

JSON descrevendo uma árvore de componentes. Cada nó da árvore segue o shape:

```json
{
  "type": "string",
  "props": {},
  "children": [],
  "actions": [],
  "fallback": {}
}
```

### Schemas

Os schemas JSON (em `v1/`) são a **fonte única de verdade** do contrato:

- `screen.schema.json` — estrutura raiz de uma tela
- `component.schema.json` — nó da árvore de componentes
- `action.schema.json` — catálogo de ações disponíveis

Backend valida toda resposta contra `screen.schema.json` antes de enviá-la. Resposta inválida retorna 500 em ambiente de desenvolvimento.

### Versionamento

- Schemas vivem no repositório `sdui-schemas`, versionado com tags semânticas (`v1.0.0`).
- Backend e app referenciam o schema via **git submodule** fixado em uma tag.
- Mudanças breaking exigem nova versão major (`v2/`).

### Evolução segura

- Para componentes **críticos** (marcados no schema): o backend respeita o header `X-App-Version` e não envia componentes que a versão do app não suporta.
- Para componentes **não-críticos**: o payload pode trazer um campo `fallback` com uma representação alternativa compatível com versões mais antigas.
- O cliente **ignora silenciosamente** componentes desconhecidos sem fallback e registra telemetria (`unknown_component` + `type`).

### Catálogo de ações

Ações são nomeadas com catálogo **fechado** — não há DSL Turing-completa:

`NAVIGATE`, `SUBMIT`, `OPEN_MODAL`, `CLOSE`, `LOCAL_TOGGLE`, `SHOW_TOAST`

## Consequências

### Positivas
- Contrato explícito detecta divergências em tempo de desenvolvimento, não em produção.
- Fallback híbrido permite deploy gradual sem forçar todos os usuários a atualizar.
- Schema como fonte única elimina cópias divergentes entre backend e app.

### Negativas
- Adicionar um componente novo exige: atualizar o schema, tagear, atualizar o submodule nos dois repos, implementar no backend e no app.
- O overhead de validação no backend aumenta latência marginalmente (aceitável em dev; pode ser desligado em produção com feature flag).
