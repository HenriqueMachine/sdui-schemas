# sdui-schemas

JSON Schema contract for the **SDUI fintech** project. This is the single source of truth shared between [`sdui-backend`](https://github.com/HenriqueMachine/sdui-backend) and [`sdui-app`](https://github.com/HenriqueMachine/sdui-app).

Both repositories reference this repo as a **git submodule** pinned to a version tag.

## Schemas

```
v1/
├── screen.schema.json      # Root payload: GET /screens/{name} response
├── component.schema.json   # Tree node: type, props, children, actions, fallback
└── action.schema.json      # Action catalog: NAVIGATE, SUBMIT, LOCAL_TOGGLE, SHOW_TOAST, CLOSE, OPEN_MODAL
```

## Versioning convention

| Version | Directory | Status |
|---|---|---|
| v1 | `v1/` | Current |

- **Non-breaking changes** (new optional fields, new component types): increment minor/patch on the tag, update both submodules.
- **Breaking changes** (removed required fields, type changes): create a new `v2/` directory, tag `v2.0.0`, migrate both repos explicitly.

Tags follow `v{MAJOR}.{MINOR}.{PATCH}` (e.g. `v1.0.0`).

## Using as a submodule

```bash
# Add to your repo
git submodule add https://github.com/HenriqueMachine/sdui-schemas.git schemas
git submodule update --init

# Pin to a specific version
cd schemas && git checkout v1.0.0 && cd ..
git add schemas && git commit -m "chore: pin sdui-schemas to v1.0.0"
```

## Architecture Decision Records

- [ADR-001 — Adotar SDUI](docs/adr/ADR-001-adotar-server-driven-ui.md)
- [ADR-002 — Contrato JSON Schema versionado](docs/adr/ADR-002-contrato-sdui.md)
- [ADR-003 — Navegação por rota nomeada](docs/adr/ADR-003-navegacao-sdui.md)
