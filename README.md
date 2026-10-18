# BTM Business — centro de conocimiento

Micrositio estático con el **conocimiento de negocio** de la familia de productos de bádminton:

- **BTM Pro** — suite de gestión de torneos (a construir, F0–F3)
- **SBS / Courtside Scoreboard** — marcador en vivo (en producción)
- **BCM** — gestión de club (idea pre-validada, semilla técnica en `app_bdm`)

Se publica en **GitHub Pages** y funciona como hub: enlaza el pitch, el plan técnico, la investigación de mercado y los repos de la familia.

## Estructura

```
BTM-business/
├── index.html      ← hub / centro de conocimiento (home de Pages)
└── pitch/
    ├── index.html  ← pitch de la familia (deck navegable, exportable a PDF con tecla P)
    └── logo.svg    ← logo de la familia (volante)
```

Los documentos fuente viven en sus repos (fuente de verdad, sin duplicar):

| Documento | Ubicación |
|---|---|
| Plan técnico F0–F3 | `courtside-scoreboard/docs/PLAN-GESTION-TORNEOS.md` |
| Brief comercial | `courtside-scoreboard/docs/COMERCIALIZACION.md` |
| Investigación de mercado (3 docs) | `courtside-scoreboard/docs/research/` |
| Semilla BCM | repo `app_bdm` |

## Publicar (GitHub Pages)

El repo se sirve con Pages desde la rama `main`, raíz `/`:

```bash
# 1. crear repo en GitHub y conectar (reemplaza OWNER por tu cuenta/org)
gh repo create OWNER/btm-business --public --source=. --push

# 2. activar Pages (una sola vez)
gh api repos/OWNER/btm-business/pages -X POST -f "source[branch]=main" -f "source[path]=/"
```

El sitio queda en `https://OWNER.github.io/btm-business/`.

## Actualizar contenido

- **Pitch:** editar `pitch/index.html` y push a `main` (deploy automático de Pages).
- **Documentos enlazados:** se editan en su repo de origen — el hub siempre apunta al `HEAD` de la rama, nunca se desincroniza.
