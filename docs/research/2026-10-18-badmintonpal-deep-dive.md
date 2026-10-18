# Deep dive: BadmintonPal (badmintonpal.com) — archivo de investigación

**Fecha:** 18-oct-2026 · **Procedencia:** informe del agente de investigación (acceso directo al bundle de la web, fichas de Google Play, AppBrain, Wayback Machine, búsquedas). Síntesis decisorio: `docs/PLAN-GESTION-TORNEOS.md` §3.

---

## 0. Resumen ejecutivo

**BadmintonPal** ("Badminton Tournament Manager") **no es una suite de torneos SaaS completa: es la extensión web del ecosistema de una app Android de arbitraje muy popular** — *Badminton Umpire Score Keeper* (100K+ instalaciones, Play desde feb-2014; Pro ≈ **US$1.99**). Detrás, con casi certeza: **Lahiru Chandima / Flexi Technologies Lanka** (Sri Lanka, la misma empresa de SpicePOS — inferencia de alta confianza: el bundle web referencia la "Badminton Umpire android app" y la ficha Play de la Pro referencia "la web app de gestión de torneos"; no declarado oficialmente).

- Web = SPA Angular sobre **Firebase** (proyecto `badminton-manager`: Auth Google/FB/email, Firestore + RTDB, FCM, App Check). Gratuito, sin pagos integrados, sin tiers. Último deploy verificado: **14-oct-2026** (activo, no abandonado).
- Monetización solo en Android: ads + compra única (feed en vivo gratis 30 días en la versión free).

## 1. Alcance funcional (verificado en el bundle)

- **Torneos → categorías** (singles/dobles) → **fase de grupos + knockout** (# de grupos, winners-per-group, `acrossGroupMatches`, división inter-grupo, niveles de KO). **Sin seeding por ranking, sin double-elim, consolation, Swiss** ni evidencia.
- **Cancha/árbitro/dispositivos**: asignación por partido + calendarios (`court-calendar`, `umpire-calendar`, `device-calendar`) + **clash detection** ("This match clashes with following…" — `app-match-clash-details`) + `averageMatchDuration` para estimación.
- **Import:** CSV drag&drop con samples descargables (`player-upload.csv`, `pair-upload.csv`, `scoring-device-upload.csv`, `sample_quick_matches.csv`). **Cero mención de Tournament Software/BWF import** en todo el bundle.
- **Export:** PDF de scoresheets por partido (vía la app árbitro + Firebase Storage). No hay export Excel/HTML para publicar en web federativa.
- **Scoring devices:** "Device ID can be found in Settings → Score feed device ID in Badminton Umpire android app" → **el punto entra desde el teléfono del árbitro por device-ID** (web `/livescores`, `/livescores/device/:id`, `scoreboard` fullscreen). Privacy del feed: "cualquiera que conozca el device id puede verlo" (política oficial del dev: docs.google.com/document/d/1k9ryhWG4dDubpRPGU7cZuCmcn_FxMo_UTg1KTcUC86k).
- **Pantallas:** (a) web global Firebase; (b) **`BadmintonScoreDisplay.exe` Windows por LAN/IP** (descarga desde Google Drive, 1 partido) — espejo informal de la TV×cancha de SBS, no es producto.
- **Quick Matches** fuera de torneo (sesiones de club).
- Páginas públicas: `tournament-info/:key`, `tournament-category-info/:key/:category` (componentes `app-public-*`).

## 2. Que no hace (brechas de BTM Pro)
Registro online con pagos (no), seeding/ranking (no), formatos avanzados BWF (no), import/export Tournament Planner (no), API (no), streaming (no), rankings/perfiles como concepto (no). Data Safety de la app: **"Data can't be deleted"**. Sin About/empresa/blog/precios; soporte por Gmail personal; mara web sin SEO (el nombre "Badminton Tournament Manager" lo ocupan otros 2 productos: app eliminada de Play en 2024 com.spellthem.swisssystemmanager y sport-software.de).

## 3. Quejas de usuarios (Play/AppBrain)
- TTS tras paywall ("auto play announcement should be enabled but it is included in premium" — 2026).
- Confusión Pro/gratis ("I already paid for the Pro version but nothing changed" — 2019; "no recibí instrucciones" — 2024).
- "Can't even create a tournament or share results" (2023) — la web nació, en el fondo, por esta queja.

## 4. Implicación estratégica
Confirma que "árbitro con teléfono + live feed" tiene demanda orgánica y que el segmento club es defensible con $0. Pero **no compite en el mercado federado/multi-cancha** (formatos, seeds, import TP, streaming, SLA). Sus fortalezas (10 años de árbitro BWF en Android, doble vía de pantalla) y sus límites definen la tabla comparativa de §2.3 del doc primario.

## Fuentes principales
- https://badmintonpal.com (+ bundle `main.205f39680feefe64.js` — Firebase `badminton-manager`, rutas, textos)
- Play free: https://play.google.com/store/apps/details?id=com.lahiruchandima.badmintonumpire · Pro: …/com.lahiruchandima.badmintonumpire.pro
- AppBrain (métricas/precios): https://www.appbrain.com/app/badminton-umpire-score-keeper/com.lahiruchandima.badmintonumpire
- APKPure (versiones v16→v64): https://apkpure.com/badminton-umpire-score-keeper/com.lahiruchandima.badmintonumpire/versions
- Política de privacidad del feed (device ID público): https://docs.google.com/document/d/1k9ryhWG4dDubpRPGU7cZuCmcn_FxMo_UTg1KTcUC86k
- Wayback: http://web.archive.org/web/20211215154629/https://badmintonpal.com/ (web SPAs al menos desde dic-2021)
- Reddit (bench del nicho, menciona a otros: Whatsscore, SmashLive, Smashduck, Playinga): https://www.reddit.com/r/badminton/comments/1gpgvyw/ y /comments/1foypx4/
