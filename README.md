# BTM Business — centro de conocimiento y negocio

Micrositio estático (GitHub Pages) que concentra el **conocimiento de negocio** de la familia de productos de bádminton:

- **BTM Pro** — suite de gestión de torneos (a construir, F0–F3)
- **SBS / Courtside Scoreboard** — marcador en vivo (en producción, repo `courtside-scoreboard`)
- **BCM** — gestión de club (idea pre-validada, semilla técnica en `app_bdm`)

**Reparto de responsabilidades:** este repo es el centro de conocimiento y negocio (pitch, planes, investigación, comercial). Los repos de producto (`courtside-scoreboard`, `app_bdm`, `club-team-chia-web`) guardan únicamente lo relativo a cada unidad de negocio.

## Estructura

```
BTM-business/
├── index.html                          ← hub (home de Pages)
├── doc.html                            ← visor Markdown (renderiza cualquier .md de docs/)
├── docs/
│   ├── PLAN-GESTION-TORNEOS.md         ← plan técnico F0–F3, arquitectura familia, riesgos
│   ├── COMERCIALIZACION.md             ← brief comercial (3 líneas de negocio, precios, IVA)
│   └── research/
│       ├── 2026-10-18-tournament-software-deep-dive.md
│       ├── 2026-10-18-badmintonpal-deep-dive.md
│       └── 2026-10-18-market-sweep.md
└── pitch/
    ├── index.html                      ← pitch de la familia (deck navegable, PDF con tecla P)
    └── logo.svg                        ← logo de la familia (volante)
```

## Publicar un documento nuevo (sin tocar HTML)

1. Guarda el `.md` en `docs/` (o una subcarpeta).
2. Enlázalo donde corresponda con el visor: `doc.html?f=docs/mi-documento.md`.
3. Push a `main` → live.

El visor renderiza GFM (tablas, código, citas), toma el título del `h1` y **navega enlaces `.md` internos dentro del visor** (los enlaces relativos entre documentos funcionan sin recargar). Requiere servirse por HTTP (Pages o `python3 -m http.server`); en `file://` el fetch de Markdown está bloqueado por CORS.

## Publicar el sitio (GitHub Pages)

```bash
# 1. crear repo en GitHub y conectar (reemplaza OWNER por tu cuenta/org)
gh repo create OWNER/btm-business --public --source=. --push

# 2. activar Pages (una sola vez)
gh api repos/OWNER/btm-business/pages -X POST -f "source[branch]=main" -f "source[path]=/"
```

El sitio queda en `https://OWNER.github.io/btm-business/` con deploy automático en cada push a `main`.

## Dónde vive cada cosa

| Contenido | Repo | Ubicación |
|---|---|---|
| Pitch, plan técnico, comercial, investigación | **este repo** | `pitch/`, `docs/` |
| Producto SBS (código + docs de producto) | `courtsideOrganizationTeam/courtside-scoreboard` | repo propio |
| Semilla BCM (código) | `santijbdm1-sudo/app_bdm` | repo propio |
| Web del club | `juandelossantos/club-team-chia-web` | repo propio |
