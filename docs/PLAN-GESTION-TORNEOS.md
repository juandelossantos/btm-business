# BTM Pro — Gestión de Torneos: investigación de mercado, arquitectura y plan de trabajo

**Versión:** 0.3 · **Estado:** investigación cerrada; plan propuesto para aprobación del equipo · **Fecha:** 2026-10-18
**Audiencia:** equipo BTM Pro / SBS.

> ⚠️ **Documento vivo.** Investigación web verificada con fuentes (octubre 2026). Los informes completos se persisten en `docs/research/` (F0.2).

### Convención de nombres (decisión del equipo)

- **SBS — Score Broadcasting System**: el producto actual (`courtside-scoreboard`). Módulo de difusión de marcadores en vivo + operación de cancha, standalone, LAN-first.
- **BTM Pro — Badminton Tournament Manager Pro**: el **nuevo** producto, suite completa de gestión de torneos. Es el **padre** de la familia.
- **BCM — Badminton Club Manager**: tercer miembro de la familia: gestión de **club/escuela** de badminton — miembros, jugadores, padres de familia, pagos de membresía, entrenadores, pases, calendario. Primera aproximación real: el repo `app_bdm/` (SaaS multi-tenant Next.js 14 + Prisma/PostgreSQL, ya con Club/roles/familias/pagos/eventos). Nombre de trabajo histórico: "app_bdm".
- Nota: `COMERCIALIZACION.md` v0.4 llama "BTM PRO" al producto actual SBS. En adelante reservamos **"BTM Pro"** para el nuevo suite; los documentos comerciales se actualizan en F0.1.

---

## 0. El encuadre decisorio (definición del equipo)

Este documento corrige la tesis de `COMERCIALIZACION.md` v0.4, que se evaluó con la mirada de "complementario a Tournament Software, nunca competir en draws". Esa estrategia **sigue siendo válida para SBS** (colaborador, ya importa sus exports) pero **la estrategia de BTM Pro es la opuesta**: competir frontalmente. Tenemos un plan comercial para SBS; **no tenemos aún uno para BTM Pro** — este documento es el paso 1 para cerrar ese hueco.

| | **SBS** (producto actual) | **BTM Pro** (nuevo, padre) | **BCM** (app_bdm) |
|---|---|---|---|
| Naturaleza | Módulo de **difusión de marcadores en vivo** + operación de cancha | **Suite completa de gestión de torneos**: registro → pagos → draws → seeding → scheduling → arbitraje → resultados en vivo → publicación → rankings → federación | **Gestión de club/escuela**: miembros, jugadores, **padres de familia**, pagos de membresía, entrenadores, pases, calendario |
| Relación con Tournament Software | **Colaborador, no competidor.** "Trabaja en equipo" | **Competidor directo y frontal** | n/a (no compite en torneos) |
| Modelo comercial | Independiente, vendible solo | Propio (parent): debe sostenerse solo | Independiente, vendible solo |
| Papel en la familia | **Hijo/módulo**: el torneo vive en BTM Pro, el marcador vive en SBS | **Padre**: origen de verdad del torneo | **Hijo/módulo**: el club (sus jugadores/miembros) es la cantera de los torneos; los resultados vuelven al historial del jugador |

---

## 1. Objetivo

> Convertir a BTM Pro en un competidor real y directo de los mejores del mercado en gestión de torneos de badminton: **locales → nacionales → internacionales**, en su totalidad.

Sub-objetivos:
1. **Inteligencia de mercado** verificada: BadmintonPal (§3), Tournament Software (§4, en curso), barrido completo del resto (§5).
2. **Respuesta crítica a "¿SBS instalable o web?"** (§7).
3. **Compatibilidad con Tournament Software**: que un usuario de Tournament Planner migre a BTM Pro rápida y fácilmente (§6) — con puente de salida también.
4. **Plan inicial de trabajo** con fases, dependencias y gates (§9).

---

## 2. Arquitectura: la familia BTM Pro → SBS + BCM

### 2.1 Tres productos, un sistema (BTM Pro = padre y hub; SBS y BCM = líneas de negocio vendibles por separado)

```
              ┌──────────────────────────────────────────────────────────┐
              │        BTM Pro — Badminton Tournament Manager Pro          │
              │              (padre y hub de la familia)                   │
              │ registro ↦ pagos ↦ draws ↦ seeding ↦ scheduling ↦          │
              │ árbitros ↦ resultados ↦ publicación ↦ rankings ↦ federación │
              └────────┬──────────────────────────┬───────────────┬────────┘
                       │ API v1                   │ API v1        │ export/import
                       │ push automática          │ push automática│ XML/CSV (coexistencia)
                       ▼                          ▼               ▼
 ┌──────────────────────────────┐  ┌──────────────────────────┐  ┌────────────────────────────┐
 │  SBS — Score Broadcasting    │  │  BCM — Badminton Club    │  │  El resto del mundo        │
 │  marcador en vivo, LAN-first │  │  Manager: club/escuela   │  │  Tournament Planner (BWF), │
 │  standalone, instalable      │  │  SaaS multi-tenant       │  │  Cup2000, Excel, otros...  │
 │  (hoy: TV×cancha + árbitro + │  │  (arranque: repo         │  │  Sigue consumiendo SBS     │
 │  TV global + offline)        │  │   app_bdm: membresía,    │  │  standalone y sigue siendo │
 │                              │  │   padres/pagos/entren.)  │  │  fuente de imports para él │
 └──────────────────────────────┘  └──────────────────────────┘  └────────────────────────────┘
   ▲ standalone: corre sin          ▲ standalone: corre sin          ▲ standalone: los imports
   │ BTM Pro (imports TS hoy)       │ BTM Pro (MVP app_bdm hoy)      │ de TS actuales se mantienen
   └─────── ninguna línea de negocio DEPENDE de BTM Pro ────────────┘
```

**Reglas de la familia:**
- **R1 — Cada hijo es vendible solo.** Nada en SBS ni en BCM exige BTM Pro. Los imports de Tournament Software de SBS (CSV/XLSX de roster + partidos) se mantienen y se mejoran (§6); BCM corre con el MVP de `app_bdm` hoy.
- **R2 — BTM Pro es el padre y el hub.** Cuando un organizador usa la familia, BTM Pro es el origen de verdad del torneo; BTM Pro↔SBS y BTM Pro↔BCM van por **API** con push automática (nada de export/import manual entre hermanos). **SBS y BCM no se conectan entre sí directamente:** se articulan solo vía BTM Pro.
- **R3 — Reutilización de ingeniería, separación de identidad:** los tres comparten la lib `packages/shared` (dominio, scoring, parsers) cuando el motor lo pide. Son **despliegues, licencias, planes de precios y bases de usuarios separados**.
- **R4 — Las puertas de salida no se bloquean:** BTM Pro exporta XML/CSV en los shapes del mundo federativo (§6.2); ningún producto atrapa datos.
- **R5 — Identidad de usuario:** v1 con cuentas separadas por producto; el SSO único de la familia es F3+ (§9). La union narrativa ("el club que también organiza y transmite") es argumento de venta del bundle, no requisito técnico de v1.

### 2.2 Alcance honesto del BTM Pro v1

Competir frontalmente exige el motor de draws (el hueco hoy ausente que el equipo **ordena reabrir**). V1:

- **a. Registro e inscripción online**: categorías (MS/MD/WS/WD/XD, sub-7…sub-19), precios por categoría, cupos, deadlines, página pública por torneo, **pagos** (pasarela Colombia + internacional: Wompi / MercadoPago / Stripe — decisión pendiente, §10.3), confirmaciones.
- **b. Motor de draws/brackets**: single elimination (byes, W.O., feed-in/consolation opcional), round robin, fase de grupos → knockout. **Editable en caliente** (replace/W.O./reseed sin invalidar lo ya jugado) — es la queja recurrente del incumbente (§5.4).
- **c. Seeding**: manual + semi-automático por criterio (orden de inscripción, ranking local, seeds a mano). Sin ranking BWF propietario en v1.
- **d. Scheduling**: canchas, árbitros, sesiones, **detección de conflictos** (es tabla básica: BadmintonPal ya la tiene, §3) + estimación de duración por tipo de partido + impresión de programa.
- **e. Resultados en vivo y publicación**: aquí BTM Pro **nace con ventaja** porque su hijo SBS ya lo hace mejor que cualquiera del mercado cloud (§2.3).
- **f. Federación v0**: páginas multi-torneo por organizador, export XML/CSV (§6); módulo federativo completo es Fase 3 (§9).

Exclusiones explícitas de v1 (para no diluir): ranking BWF propio, membresías federativas, apps móviles para jugadores, white-label completo.

### 2.3 Posicionamiento: ninguneado por ambos extremos

La investigación (§3, §5) muestra que **ningún competidor une el mundo cloud con el mundo LAN-real**:

| | Tourneysoft | Playinga | BadmintonPal | Tournament Planner | **BTM Pro + SBS** |
|---|---|---|---|---|---|
| Registro online + pagos | ✔ | ✔ (free tier) | ✗ | ✔ (federativo) | ✔ (v1) |
| Motor de draws | básico | básico | básico (grupos→KO) | ✔ (fuerte) | ✔ (v1) |
| Scheduling + conflictos | parcial | parcial | ✔ (básico) | ✔ (fuerte) | ✔ (v1) |
| **TV por cancha profesional multi-cancha (LAN)** | ✗ | ✗ | ~ (exe de Drive, 1 partido) | ✗ | ✔ **hoy, ya desplegado** |
| **Offline / LAN-first el día del partido** | ✗ | ✗ | parcial | ✔ (desktop) | ✔ **hoy** |
| Árbitro en teléfono, BWF completo | ✗ | ✗ | ✔ (el mejor del nicho) | ✗ | ✔ **hoy** |
| Live score punto-a-punto sub-segundo | ✗ | parcial | ✔ (device-ID) | ✗ ("can take hours" — comunidad, §5.4) | ✔ **hoy** |
| Migrable desde Tournament Planner | ✗ | ✗ | ✗ | n/a | ✔ **(§6 — único con puente)** |

**La posición no es "copiar a Tournament Software": es "Tournament Software + el día del partido que nadie tiene".**

### 2.4 La familia conectada: flujos entre líneas y modos de independencia

#### a) Cómo se conectan (cuando la familia está unida)

```
   ┌────────────────┐                          ┌───────────────────────────┐
   │  BCM (club)    │ ──────roster (API)─────▶ │  BTM Pro                  │
   │  miembros,     │  1-click "inscribir mi   │  torneo, categorías,      │
   │  padres/pagos, │  club a este torneo"     │  draws, seeding, schedule │
   │  entrenadores  │                          │  pagos de inscripción     │
   └───────┬────────┘                          └───────┬──────────┬────────┘
           │                                          │          │
           │  historial: los resultados               │ API v1   │ export XML/CSV
           │  "vuelven a casa" al perfil              ▼          ▼ (la FA sigue
           │  del jugador en BCM (F3)          ┌──────────────┐   en TP si opera)
           │                                   │  SBS         │   ┌────────────────┐
           │                                   │  (máquinas + │ ◀─│ Tournament     │
           └─ roster de BCM también entra ▶    │  TVs + árbitro│  │ Planner etc.   │
              como CSV/XLSX manual             └──────────────┘   └────────────────┘
              si BTM Pro no está
```

**Contratos de integración (API v1, diseño en F1/F2):**

| Flujo | Producto origen → destino | Qué viaja | Disparo |
|---|---|---|---|
| Roster del club | BCM → BTM Pro | miembros/athletes (nombre, género, DOB, grupo/edad) | 1-click "inscribir mi club" en BTM Pro (o CSV si BCM no está) |
| Torneo publicado | BTM Pro → SBS | categorías, draws programados, canchas, árbitros asignados | push automática al "adjuntar" el evento (SBS consume una API key por instalación) |
| Marcador en vivo | SBS → BTM Pro | progreso de partido (estado, sets, ganador) | streaming por socket (el pipeline que SBS ya usa para TV global) |
| Resultado final | SBS → BTM Pro | match result authoritative (event-sourced) | push al finalizar |
| Historia del jugador | BTM Pro → BCM | resultados/posiciones por playerId | pull/sincronización diferida (F3) |
| Pagos de inscripción | BTM Pro → pasarela (→ BCM opcional) | recibo por jugador si el torneo lo facturó al club | webhook (F3+) |

#### b) Cómo funciona cada uno en TOTAL independence

| | SBS hoy | BCM hoy | BTM Pro v1 |
|---|---|---|---|
| Origen del torneo/club | Tournament Planner, Cup2000, Excel, CSV/XLSX manual (imports ya implementados) | MVP `app_bdm` (club, roles, familias, pagos, eventos) — sin torneo formal | El torneo se crea y vive aquí (suite completa §2.2) |
| Marcador | SBS completo (TV×cancha, TV global, árbitro teléfono, offline, WebRTC opcional) | — | Web pública de BTM Pro con **score entry manual en web** (fallback) o SBS conectado (mejor) |
| Publicación | TV + web pública local + Firebase público opcional | portal interno del club | página pública por torneo + live links |
| Lo que GANA si se conecta | el torneo entero entra solo (sin import ni digitación) | sus jugadores compiten y su historia vuelve automática | el día de partido real (SBS) y la cantera (BCM) |

**Regla de oro de independencia (del equipo):** cada producto debe saber convivir con "usted no me tiene instalado": SBS con Tournament Planner/Excel; BCM solo; BTM Pro con import manual + score entry web. La familia conectada es un **upsell que simplifica la vida**, nunca una dependencia estructural (de ahí R1–R5).

#### c) Punto crítico de diseño (por qué el hub es BTM Pro y no un "backend común" compartido)

Es tentador crear "una tabla de jugadores global compartida por los tres". Es una **trampa**: obliga a que las tres bases vivan en el mismo despliegue/servicio y fusiona dos dominios que hoy son distintos (el Member del club vs el Player del torneo: mismas personas, datos ligeramente distintos, autorizaciones de privacidad distintas — sobre todo con menores, §8.5). La integración correcta: **mapeo de identidades por API en el momento del "roster push"** (BCM responde "estos son mis IDs y estos los datos que comparto para tu torneo"), con los datos del torneo viviendo en BTM Pro y los del club en BCM, y sincronización diferida consentida. Es la única opción que respeta R1 (independencia) y las exigencias legales de datos de menores.

---

## 3. Competidor: BadmintonPal (badmintonpal.com) — deep dive

*(Resumen del informe completo de investigación — fuentes verificadas en el informe original en conversación; a persistir en `docs/research/` durante F0.2.)*

**Qué es:** ecosistema de un desarrollador indie (casi seguro Lahiru Chandima / Flexi Technologies Lanka, Sri Lanka — el mismo de SpicePOS): **app Android "Badminton Umpire Score Keeper" desde 2014 (100.000+ instalaciones; Pro ≈ US$1.99)** + **web Angular/Firebase gratuita** (badmintonpal.com, SPA; último deploy verificado: 14-oct-2026). La web se llama "Badminton Tournament Manager".

**Fortalezas:**
- **El árbitro Android más maduro del nicho** (10 años; BWF: anuncia lo que el árbitro debe decir, saques, deshacer, timers; scoresheet PDF).
- **Live scores reales** vía device-ID (app árbitro → Firebase RTDB → web pública / scoreboard fullscreen).
- **Doble vía de pantalla:** web global + **`BadmintonScoreDisplay.exe` LAN (misma Wi-Fi, fullscreen)** — espejo de SBS, pero de 1 partido y descargado de Drive (no es producto).
- **Motor básico completo:** grupos (conteo, winners-per-group, partidos inter-grupo, división) → knockout; canchas/árbitros/dispositivos; **clash detection** ("This match clashes with following…"); import CSV multi-tabla con samples; páginas públicas; PDF por partido.
- Precio: **$0** en web (no hay pagos integrados; monetiza en Android solo).

**Debilidades explotables (brechas que BTM Pro ataca):**
1. **Cero marca/empresa/docs:** sin About, sin precios, sin blog, soporte por Gmail; SEO confundido (el nombre "Badminton Tournament Manager" lo ocupan otros 2 productos).
2. **No compite en el segmento federativo/multi-cancha:** sin seeding por ranking, sin consolation/Swiss/formatos BWF, sin import desde Tournament Planner, sin API.
3. **Registro con pagos: inexistente** (el organizador carga equipos a mano).
4. **Streaming: ausente.**
5. **One-dev + Firebase lock-in:** sin SLA (asusta en torneos federados); Data Safety: "Data can't be deleted".
6. Quejas crónicas de su suite (paywall TTS, confusión Pro/gratis, "Can't even share results" 2023 — la web nació, en el fondo, por esa queja).

**Lección de BadmintonPal:** "árbitro con teléfono + web live" tiene demanda **orgánica** (100K installs sin marketing). Y confirma que nadie en badminton-cloud cubre LAN multi-cancha + offline + árbitro profesional + streaming. **Ese hueco ya es BTM Pro.**

---

## 4. Competidor: Tournament Software / Tournament Planner — deep dive

*(Informe completo en `docs/research/2026-10-18-tournament-software-deep-dive.md`, con todas las URLs. Aquí, lo decisorio.)*

### 4.1 Quién es y qué vende

- **Visual Reality B.V.** (Alkmaar, Países Bajos; KvK 39067526). ~40 empleados, stack **C#/.NET + React**, 30+ años de historia, sin publicidad: "200+ federations in 160 countries, 40,000 tournaments every year". Términos: ley neerlandesa, soporte **solo L-V 9–17**, productos "AS IS… no liable for any data losses" (T&C arts. 8–9).
- **Familia:** Tournament Planner (desktop **Windows** por deporte), Tournament TV (pantallas de sala tipo PowerPoint — antecesor de negocio de SBS), Visual LiveScore, **TP Network** (match control en LAN cableada: segundo exe con IP+password contra el PC de TP, solo arrastra partidos/pagos/check-in/resultados), League Planner, box ladders, rankings, y un **Online Tournament Planner SaaS "coming soon"** que lleva años en "Something new is coming!!" (€299/año provisional). Plataforma federativa white-label con módulos cotizados (CMS, SSO, pagos, **"TS Services API"** — API privada, solo bajo contrato con federación).
- **Pricing 2026:** Tournament Planner desktop **€106/año por deporte** ("includes free updates and usage of tournamentsoftware.com until Jan 1st next year"); ladders €107; Online €299 provisional ("final price has not been set" — la página lleva años así). White-label federativo: a propuesta.

### 4.2 El moat real (y por qué NO es el software)

El entrenchedness no es el exe: es el **ecosistema de datos y personas** —
1. **Distribución a presión federativa:** el manual de **BWF Referees Level 2** tilda TP de "ubiquitous" y exige competencia en TP a los referees internacionales; los circuitos de grado 2/3 europeos **obligan** a usar la "BWF-license" de la Member Association (€500/torneo en el Badminton Europe Elite Circuit). **Badminton Oceania entrega el archivo TP ya configurado** al organizador y controla el "master TP file". India/B Maharashtra: TP obligatorio para campeonatos estatales.
2. **Mapa federativo activo (verificado hoy):** `bwf`, `bwfpara`, `fir`, `be` (Badminton England, obligatorio), `ffbad` (Francia), `badmintondenmark` (Dinamarca — migró desde Cup2000), `badmintonscotland`, `bam` (Malasia), `ba` (Australia), `bnz` (NZ, con manual propio), `badmintoncanada`+`badca`, `badmintonfinland`, `sba` (Singapur), `bat` (Tailandia), `competitions.be` (Bélgica white-label con dominio propio).
3. **Base de datos de jugadores + ranking por federación + árbitros entrenados + 30 años de archivos** — migrable federación por federación, no en bloque.

### 4.3 Formatos de datos — la sección crítica para migrar

- **Nativo: nada de XML/JSON público.** El "master" es el archivo **`.tp` binario propietario** (un archivo por torneo; migrarlo entre PC implica exportar/desinstalar/importar; las versiones vía email "come easily out of sync"). No hay DTD/XML documentado. La única API estructurada es la "TS Services API" **privada de federación**.
- **Importar INTO TP (superficie documentada — la que ofrecen ellos mismos):**
  - **Excel de base de jugadores**: `Player > Import to Player Database` — columnas fijas definidas por TP: *Member ID* (código federativo, obligatorio), nombres, género, DOB, y tabla Events por categoría. (Manuals NZ/BNZ quick-start, FIR, BWF L2).
  - **Excel de ranking**: `Player > Import Ranking` con *Publication Date*.
  - **Entradas online**: `Internet > Entries` (el wizard descarga los registros web).
  - **Resultados online**: `Internet > Download Results`.
  - Menú con `Import Results…` / `Import Schedule…` existentes (formato no documentado — a muestrear).
- **Extraer AWAY desde TP (migración hacia BTM Pro):**
  1. **Los XLSX que TP define** (players/ranking) — **el formato más estable disponible, porque TP y las FAs lo usan para alimentar a TP**. Si BTM Pro sabe leer esos shapes, hereda la aduana de entrada de todo el ecosistema.
  2. **Páginas públicas del torneo, con URLs GUID estables**: `/{fed}.tournamentsoftware.com/sport/tournament.aspx?id={GUID}` con subpáginas `draws.aspx`, `players.aspx`, `seeds.aspx`, `clubs.aspx`, `matches`, `legacymatches.aspx?d=YYYYMMDD` por jornada (HTML server-rendered; los perma-links de torneos viejos siguen vivos). Import s**olo con el link aportado por el dueño del torneo** — ver riesgo legal.
  3. **Reports/imprimento**: TP emite poule standings, matches per day, match cards, players, entries per event, draw list, court schedule, winners, payments — vía PDF/print; CSV parcial.
- **⚠️ Riesgo legal explícito:** el T&C de Visual Reality (**art. 10.2(h)**) prohíbe usar el servicio "to develop, train or benchmark a competing product", además del 10.2(b) (no reverse engineering). **El website/find del portal NO se scrapea jamás.** La integración se limita: (a) XLSX que los propios organizadores exportan de TP, (b) el link concreto del torneo del propio usuario. Sin robots, sin find, sin harvest masivo.

### 4.4 El flujo de trabajo que hay que batir (y dónde sangra)

La cadena real (manuales FIR/BWF L2/Oceania 2026): federación crea el "TP file" plantilla (eventos, tarifas, pistas) → organizeds online con cuentas federativas en `bwf.tournamentsoftware.com` → `Internet > Entries` baja los registros → Player Validation (¡función nativa porque "wrong nationality, wrong player id, different version of player name" es dolor frecuente!) → import de ranking para **seeding** (separación por país, draws Monrad, qualifiers) → **scheduling** — y aquí el x-ray documental propio del furo manual FIR: *"easily the most difficult and time-consuming single job… TP is not able to do everything automatically… You might find it easier to schedule matches using an Excel and transfer to TP afterwards"* → match day con **TP Network** (LAN) + emitir los "LiveScore" intraday (visualizado en el portal; el publish continúa adrenalizzato: auto-publish results desde 2018.1, **solo resultados**, draws se publican a mano) → cierre: final positions + **envío del .tp a la federación** (que lo exige; Oceania/FIR con protocolo).

**Live score de TP:** mediado a puño del árbitro/PC: la comunidad lo describe como *"at the mercy of individuals posting scores… can take hours"* (BadmintonCentral) y reddit enero-2025: *"they rarely get the match results posted on time"*. TP Network es LAN **cableada**, sin match control cloud ni mobile. **Contra SBS, ese día de partido es de otra era.**

### 4.5 Debilidades y ventana estratégica

- **Windows-only desktop en 2026** (req. Win 8/10/11); nada Mac/iPad/Linux. El "Online TP" es vaporware con precio abierto. **Changelog 2018–2025: bugfixes anuales** (la novedad de 2025.1: soporte OAuth de Gmail).
- **Sincronización frágil por archivo** (.tp por email; múltiples versiones desincronizadas — patrón documentado por manuales); soporte L-V y "no liable for data loss" — brutal para el día del torneo.
- **La ventana:** (1) BWF **cerró** las webs públicas de TP (7-ene-2025) y llevó la experiencia de fans a bwfbadminton.com + app B4U — el "estándar para el fan" ya se decidió lejos de Visual Reality; (2) **KNLTB (tenis/pádel NL) fuerza el Online TP desde enero 2026** = la primera migración desktop→SaaS real del ecosistema; (3) el roadmap corporativo confiesa: "Our roadmap **through 2028** focuses on building a future-proof SaaS platform… reducing technical debt" — hasta 2028 van a estar ocupados en su deuda. **BTM Pro tiene 2–3 años de cielo abierto en el segmento organizador.**
- **Lección comercial:** TP cobra €106/año **por deporte** (un organizador de tenis+badminton paga 2×). BTM Pro nace badminton-only con free tier + precio anual organizador competidor (COP-friendly). Y cuando el día llegue a multi-deporte, esa lección ya está aprendida.

---

## 5. Barrido del resto del mercado

*(Síntesis del barrido completo — el informe original con todas las fuentes se persiste en F0.2.)*

### 5.1 La pirámide del mercado

1. **Federativo internacional:** `bwf.tournamentsoftware.com` (Visual Reality) — backend BWF oficial (registro de jugadores, sanción de torneos, inscripción a torneos sancionados). La web pública de resultados se **cerró el 7-ene-2025** (la experiencia de fans migró a bwfbadminton.com + app B4U) → la capa de *experiencia* ya se decidió fuera de Visual Reality.
2. **Federativo nacional:** Tournament Planner licenciado por FAs (España vía Todostorneos.es — **el mismo producto** versión ES), Cup2000 (Noruega; Dinamarca migró Cup2000→TP), BadmintonPlayer.dk, Club Locker (USA Badminton: membresías desacopladas de los torneos = inconsistencia), procesos semi-papel (RFEB: PDFs por email; Perú: voucher por Instagram/WhatsApp; Colombia: TP bajo el estándar BWF SH-6 de la federación).
3. **Club/local:** el caos productivo — Excel + Google Forms + WhatsApp; genéricos (Challonge, Toornament…) "requieren manual setup de draws y seeding de badminton" (gitnux 2026). **Y la señal grande: Badminton Europe tiene una licitación abierta** buscando una empresa que desarrolle software propio de membresía + sanción + torneos (Facebook de BE, enero 2026). Badminton Ireland ya movió datos/membresía a **Sport:80**.

### 5.2 Mapa de amenazas

**Real:** ❗ Tourneysoft (SaaS badminton EN+ES, el más parecido a BTM Pro), ❗ Playinga (español + tier gratuito + LATAM; SEO "torneos badminton Medellín"), ❗ Cup2000 (en declive pero mide el estándar), ❗ Brakto (clon temprano del posicionamiento de BTM Pro), ❗ Sport:80 (si escala de membresía a torneos).
**De nicho (vigilar):** Rackonnect (India, 50K+ jugadores), SliceWin (grass-roots "solo el admin necesita app"), Smashduck (iOS novata), STADIUM/TournMate/Rapid Brackets (piloto USA 2026).
**No amenaza:** KeepTheScore, Ultimate Streamer, Whatsscore, Tournify, Competize, Challonge puro.

### 5.3 Benchmark "best in class" a emular

- **Smoothcomp** (deportes de combate): scheduling drag&drop con estimación en vivo, brackets multi-formato, check-in QR, scoring tablet, livestream con overlays generados del scoreboard, plataforma federativa white-label. *La hermana directa del roadmap de BTM Pro.*
- **Scoreholio** (pickleball/cornhole): tablet por pista + kiosk, dashboard en cualquier TV, scoring desde teléfonos con QR, cobros en app, "announce mode", offline degradado elegantemente ("tablets lost connection = no problem"). La prueba de que **1 solo organizador** puede correr 7 canchas en 3.5h.
- **USTA Serve Tennis + Match! App (VTD):** plataforma federativa + el **"Virtual Tournament Desk"** (reporte de scores, notificaciones de asignación de cancha, match status) — oro estándar del flujo organizador–jugador–espectador.
- **Sport:80:** 80+ federaciones nacionales; caso badminton: Badminton Ireland.
- **Playtomic:** el torneo como actividad social del club (Americano/Mexicano/Eliminación con pareo automático) — volumen recurrente entre torneos serios.
- **Toornament:** planes por tamaño; **full data ownership** y API white-label en el tier alto — modelo del programa de federaciones.

### 5.4 Pain points de organizadores (con evidencia)

1. **"Spreadsheet headache"**: round robin de 8 jugadores = 28 matches a mano; rotación de dobles = pesadilla matemática (SliceWin; hilo mod-approved en r/BadmintonIndia: "without the spreadsheet headache").
2. **El live score de facto no existe**: sobre Tournament Software: *"at the mercy of individuals posting scores… can take hours… even then it could just be the final scores of the day"* (BadmintonCentral). Pregunta recurre desde 2019: "Does badminton have ANY live score app?" — sin respuesta seria fuera del pro. **Sin API pública de Visual Reality.**
3. **La barrera de adopción gana a las features**: "the biggest fear of organizers is making older players download an app" (SliceWin).
4. **Cambios de última hora rompen torneos**: "bracket changes during play can feel operationally rigid" (gitnux 2026). El draw editable en caliente es un ganador, no un detalle.
5. **Cobros desde la app** es tabla obligatoria (Scoreholio/Smoothcomp/USTA/Playtomic/Sport:80); en badminton cloud casi nadie.
6. **Los genéricos obligan a workarounds de badminton**: "TeamSnap lacks bracket creation, seeding logic, automated scheduling"; SportsEngine "scoring requires workarounds".

### 5.5 Top 10 must-have para "best in market" (derivado de evidencia)

1. **Import/export completo de Tournament Planner** — jugadores + inscripción + **draw + resultados + seeds** (§6; nadie de los rivales cloud lo tiene: es la barrera que nos hace tomables en cuenta).
2. **Live score punto-a-punto real** — ya vía SBS: sub-segundo desde el árbitro vs "voluntarios que tardan horas".
3. **Offline/LAN-first del día del partido** — ya vía SBS, con degradación documentada.
4. **TV por pista + dashboard central + announce mode** — ya vía SBS (falta el announce).
5. **Scheduling drag&drop con conflictos + estimación de duración** (Cup2000/Smoothcomp core; BadmintonPal ya esboza).
6. **Draw editable en caliente**: replace/W.O./bye/reseed sin invalidar lo jugado.
7. **Inscripción con pagos** (pasarela CO nativa; Stripe internacional).
8. **Web del torneo sin app para espectadores**: draws, timetable, standings, live stream + notificaciones (Match VTD-style).
9. **One-operator experience**: 1 dispositivo por pista (teléfono/tableta árbitro), nada obligatorio para jugadores.
10. **Programa de organizadores/federación**: subdominio, rankings por temporada, export XML/CSV/API, white-label.

**#11 (bonus):** formato social de club — Americano/Mexicano/rotación de parejas con pareo automático (Playtomic) → volumen entre torneos federados.

### 5.6 Evidencia de switching (el incumbente es vulnerable)

- BWF **cerró** la web pública de Tournament Software → la capa de experiencia ya se "decidió" lejos de Visual Reality.
- Dinamarca reemplazó Cup2000 → Tournament Planner **con plan manual de apoyo a clubes** → las FAs cambian de software cuando un vendor les ofrece más; Visual Reality no es invencible.
- **Badminton Europe licitando** suite propia; Ireland con Sport:80; USA desacoplado.
- **Nadie** ha desplazado a TP en torneos federativos sancionados (rosters/draws/results sanitizados). El flanco de entrada real: live score real, mobile-first, operación de día D — y export XML para que la federación no pierda nada mientras BTM Pro corre el torneo.

---

## 6. Estrategia de compatibilidad y migración desde Tournament Software (basada en §4.3)

**Premisa del equipo:** que un usuario de Tournament Planner migre a BTM Pro rápido y fácil — y que mientras tanto SBS siga "trabajando en equipo" con TP (R1).

### 6.1 Las 4 rutas de entrada (por orden de costo/valor)

1. **Ruta 1 — Excel nativo de TP (la aduana principal):** parser de los XLSX **definidos por el propio TP**: base de jugadores (Member ID + nombres + género + DOB + eventos) y ranking (puntos + Publication Date). Ventaja estructural: es el formato que **TP, las FAs y los referees ya usan entre sí** — si BTM Pro lo habla, hereda la integración con el ecosistema federativo sin pedir permiso a nadie. Se reutiliza el pipeline de `players.ts` (mapping, validación defensiva, errores por fila, no fail-all) — la disciplina que el repo ya aplica con `xlsx.test.ts` y un export real como fixture.
2. **Ruta 2 — Link público del torneo del usuario:** el organizador pega la URL de **su** torneo (`/{fed}.tournamentsoftware.com/sport/tournament.aspx?id={GUID}`) y BTM Pro extrae el estado publicado: players, seeds, draws, matches por jornada (HTML server-rendered con estructura estable y URLs GUID fijas). **Consentimiento del dueño + cero scraping del portal** (sin find, sin robots, sin harvest — riesgo T&C art. 10.2(h), ver §4.3). Tests de fixture como alerta temprana ante cambios de DOM.
3. **Ruta 3 — Reports de cierre (v2):** CSV/print de reports de fin de torneo (winners, final positions, payments) para historial/ranking.
4. **Ruta 4 — Archivo `.tp`:** **no legible** (binario propietario — decirlo honesto en la documentación). La migración de datos pasa por las rutas 1–3; el `.tp` puede seguir alimentando a la FA mientras nuestra export (6.2) cubra sus exigencias.

### 6.2 Ruta de salida — la condición para poder correr un torneo federado

La sostenibilidad federativa exige que, aunque la FA siga en TP, el torneo corra en BTM Pro: **BTM Pro exporta en los shapes que TP y las FAs importan** — Excel de jugadores con Member ID, Excel de ranking con Publication Date, y los reports que consumen (draw list, court schedule, winners, entries per event). Es la ruta 1 invertida: si TP solo come Excel en sus shapes, producimos ese Excel. Sin XML/JSON oficial (no existe), sin scraping, sin licitación concedida: **coexistencia operativa desde el día 1** — el organizador corre el torneo en BTM Pro y hace la entrega federativa con lo que BTM Pro exporta.

### 6.3 UX del wizard de migración

"Arrastra tu Excel de TP / pega el link de tu torneo" → preview + validación fila a fila + elige qué entra (roster / ranking / draws / resultados) → import. Reuse de los patrones y del pipeline del import actual de roster en el admin de SBS. Y una **"migración demo" con datos reales** en la landing ("Migra de Tournament Planner en 5 minutos").

### 6.4 Guardrails

- **Nunca** leer/atacar el archivo `.tp` (reverse engineering — prohibido por sus T&C e inútil por la fragilidad del binario).
- **Nunca** scrapear el portal (find público ni harvester) — solo el link concreto aportado por el usuario dueño del torneo.
- Fixtures de test con **exports reales** persistidos en repo (como ya hizo el roster).
- Posicionamiento público = **coexistencia** ("Compatible con Tournament Planner: importa tu roster, ranking y cruces"), como SBS lo hace hoy — no confrontación legal.

---

## 7. Decisión crítica: "¿SBS instalable o web?"

**Respuesta corta: las dos cosas, con roles distintas y sin hibridar.**

1. **SBS sigue siendo instalable desktop/LAN (standalone):** su ventaja estructural es exacta "una instalación = una máquina" y sin internet obligatorio. **No se webifica** — webificarlo es perder offline, que es lo que ninguna web competidora tiene.
2. **Cuidado que es también real:** "SBS standalone comercial independiente" significa dos funnels, dos licencias, dos manuales, dos soportes. Para que no sea una sábana de costos: SBS standalone se divide en tier Free + pago per-evento; **el bundle BTM Pro + SBS regala el tier completo de SBS** — si el standalone SBS añade "marko per-evento", excelente; si no, su papel estructural es que **BTM Pro nunca pierda el día del partido**.
3. **BTM Pro nace cloud/web** (registro + pagos + publicación no funcionan sin internet, y contra Tourneysoft/Playinga ahí es donde va la guerra). **PERO** el corazón del torneo (draws/scheduling/resultados) se diseña PWA/offline-cache para el organizador sin señal en el gym; el offline real del marcador lo tiene SBS.
   - ⚠️ **No replicar la instalación LAN de SBS dentro de BTM Pro** — es duplicar inversión y fragmentar la familia. BTM Pro asume internet en la sala de control; si no, se cubre con SBS standalone (que ya lo llevamos).
   - ✔ **Local fallback mode:** siempre existe la salida "backup local + XML" de BTM Pro para poder correr el torneo con SBS standalone + printables si la sala pierde internet en puro dead-obligado.
4. **Packaging SBS:** el instalable con licencia offline firmada (COMER §10) es **también el packaging del módulo ScoreBroadcast dentro de BTM Pro** — un Electron que consume la API. Compilar el mismo SBS con dos orígenes de datos: imports manuales (hoy) / API de BTM Pro (mañana).
5. **Criterio de honestidad si la infraestructura de cloud favery Required:** si el equipo no alcanza para dos productos en simultáneo, BTM Pro **managed-first** (acompañado por el servicio del equipo, COMER §7) vende mientras se construye — propuesta de arranque dual en §9.

---

## 8. Eslabón comercial: cómo se paga la familia

### 8.1 Tres modelos, oficialmente separados (familia de facturas independientes)

- **SBS** (v0.4 sigue vigente con correcciones del brief crítico §8.5): licencia offline + servicio de operador (2M COP/evento, +1M/día adicional) + comisión de enganche + add-ons (streaming, resultados públicos, muro patrocinadores) + formalización DIAN.
- **BTM Pro** (plan comercial nuevo — v0.0 a escribir tras resolver §2/§4/§6): pricing por organizador, registro con pagos (fee de la pasarela claro en la economics), add-ons (API, white-label), **el bundle "incluye SBS completo"** como anchor, canal federativo por licitación.
- **BCM** (primera línea de renta recurrente de la familia): SaaS por club — free 1 club pequeño + pago por tamaño/miembros, con la **transacción de pagos de membresía como monetización natural **(pasarela CO; % o fee plano por cobro)**. El `app_bdm` MVP ya tiene el modelo de datos (Club/roles/familias/PaymentConcept/PaymentRecord) y la arquitectura soporta lo comercial (README). Su doc comercial propio: `COMERCIALIZACION-BCM.md` (F0.3).

### 8.2 Pricing probe contra el mercado (hipótesis, a validar)

| Linea | Rivales observados | Propuesta v0 (a validar) |
|---|---|---|
| BTM Pro per-torneo (registro+draws) | Tourneysoft; Playinga free tier; TS desktop real: **€106/año por deporte** (≈ COP 424k), Online provisional €299 (§4) | Free hasta 32 jugadores (neutralizar el free de Playinga); COP 250k–400k/torneo grande |
| BTM Pro club/academia anual | SBS v0.4 "plan estrella" 1.5M–4M/año | Plan familia unificado: BTM Pro + SBS como bundle anual (⚠ canibalización controlada del "plan estrella", ver brief) |
| BTM Pro federación/white-label | Sport:80, Smoothcomp federation, Toornament Arena | Cotizado; API + subdominio + export XML; targets: licitación BE, FAs LatAm vía Panam |
| BCM club SaaS | Playtomic/club; TeamSnap; SliceWin "solo el admin paga" | Copiar la lección de adopción: **el club paga, los jugadores no instalan nada**; free 1 club + tiers por nº miembros; % de pasarela de membresías |
| Bundles | (nadie lo ofrece en badminton) | "El club y sus torneos en una cuenta" (BTM Pro + BCM), "el club + torneos + pantallas" (3 líneas) — con descuento de familí y un único apoyo comercial |

### 8.3 El bundle como arma motora

El "ambiente montado" que ningún competidor tiene: **el día del partido en LAN** (TV×pista + offline + árbitro + streaming). Se negocia en un solo precio: BTM Pro vende "Tus torneos, de punta a kappa — y el mejor día de partido del mercado" en una factura (subscription o per-evento), y SBS completo va dentro.

### 8.4 Canal federativo — "operación beta institucional"

Playbook, del caso empíricos del barrido: ((1)) BTM Pro light + SBS en 1–2 organizaciones **locales** de LatAm para la vitrina con datos reales; ((2)) oportunidades de licitación (Badminton Europe, FAs de Panam) con "**import-compatible Tournament Planner**" como preágrama y "co-exist con TP hasta la transición"; ((3)) caso de estudio con números.

### 8.5 Rectificaciones del brief comercial que ya son acción (del brief local, no se repiten: se agendan en §9)

Unit economics cero → F0.3; IVA 19% en la lista final de precios → F0.3; "plan estrella" sin evidencia → F0.3/F1; firma de instaladores (bloqueante, #33) → gates transversales §9; código fuente TS distribuido en el .asar (crown jewel copiable, security fase1) → gates transversales; datos de menores PROPHIBIDO hoy en public → legal #103 + gate; **marca:** ShuttleSync vs BTM Pro activo → **resuelta: BTM Pro** (F0.1 ahora formaliza, no decide).

---

## 9. PLAN INICIAL DE TRABAJO (propuesta para aprobación del equipo)

**Formato:** fases con gates. Estimaciones optimistas (P50). Detalles de dev (repositorio, hosting) en §10.

### F0 — Fundaciones (~1–2 semanas) · prioridad máxima
- **F0.1 Formalización de nombre BTM Pro** (ya decidido por el equipo; queda ejecutarlo): actualizar `COMERCIALIZACION.md` (que llama "BTM PRO" al actual), EULA, i18n/brand-tokens, **limpiar residuo ShuttleSync** (artefactos stale de public-viewer, doc fase1 "executableName"), y formalizar en SIC la marca al tiempo que se cierra el gap legal. ✔ acción por escrito + limpieza de residuo.
- **F0.2 Cierre de persistencia de la investigación (formateada):** informes completos de BadmintonPal, barrido de mercado y Tournament Software ya cerrados → **persistir en `docs/research/2026-10-18-*.md`** + **documentar el plan técnico de migración** con los formatos de §4.3/§6 (Excel players/ranking de TP, link de torneo del usuario, shapes de export) y **fixtures reales** de export de TP en repo. ✔ doc + fixtures listos para F1.5.
- **F0.3 Plan comercial en tres archivos**: `COMERCIALIZACION-SBS.md` (refinado del actual, modelo standalone) + **`COMERCIALIZACION-BTM-PRO.md` (nuevo)** + **`COMERCIALIZACION-BCM.md` (nuevo, ligero)** — con IVA explicito, unit economics mínima del evento, pricing probe de §8.2 (incluye líneas BCM y bundles) y el pitch de una página por producto. ✔ aprobación del esquema de precios.
- **F0.4 Auditoría de arrancador BCM (`app_bdm`):** inventario de lo que ya existe/funciona (multi-tenant, roles, familias, pagos, eventos, TournamentMatch), huecos vs el alcance v1 deseado (pases, entrenadores, pagos piloto real), calidad/seguridad y decisión de motor: evolucionar `app_bdm` o re-base.产出: `docs/PLAN-BCM.md` (plan propio de la línea, con su propio calendario — no bloquea BTM Pro; puede correr en paralelo con otro dev o después de F1).

### F1 — BTM Pro v1 "gestión mínima total" (duración por estimación del motor)
- **F1.1 Ingeniería:** paquete `packages/tournament` en el monorepo (dominio de BTM Pro: torneo, categoría, inscripción, draw, schedule, results) con `shared` reutilizable. ✔ dominio typecheck + tests.
- **F1.2 **Motor de draws** (en `shared`, standalone-testable): SE con byes/W.O., round robin, grupos→knockout; **editable en caliente** (replace/W.O./reseed). ✔ spec + kit de tests (incl. edge cases del arbitraje que SBS ya maneja).
- **F1.3 API de BTM Pro** (hosting cloud, modelstride del server de SBS como base Express/Socket.io/Prisma): CRUD torneos, inscripciones con **pagos** (Wompi/MercadoPago/Stripe), auth equivalente a SBS. ✔ e2e register→pagar→draw→publish.
- **F1.4 Web pública del torneo** (página pública): draws, timetable, standings, live links a SBS; notificaciones v0 (email). ✔ link sin app exigida.
- **F1.5 Import XML/CSV de Tournament Planner** (módulo BTM Pro, reutilizable para SBS). Se diseña cuando §4 cierre. ✔ wizard con fixtures reales, incl. migración end-to-end demo.
- **SBS lane (R1):** push API de BTM Pro → SBS (recompa server socket existente). Marketing standalone SBS: "works with Tournament Planner, Cup2000, Excel…" ✔ página de integraciones.

### F2 — El diferencial: día de partido integrado (~2–3 semanas)
- **F2.1 SBS↔BTM Pro**: BTM Pro publica `match:scheduled` / `score:live` / `match:done`; SBS consume. ✔ demo end-to-end de un día 1 real.
- **F2.2 Announce mode + kiosk/TV central** (Scoreholio-parity): "court 3: player A vs B" a la sala de control. ✔ A11y.
- **F2.3 Bracket editing caliente** en UI con el mismo motor de F1.2. ✔ flujo real de change durante tournament.

### F3 — Escala federativa + comercialización en acción
- **F3.1 Export XML/CSV (targets federados) + result feed.**
- **F3.2 Programa de organizadores:** multi-torneo, subdominio, rankings por temporada.
- **F3.3 Sales:** eventos reales de pago como pilotos, 1 FA Latam pilot (proponer a través de Panam y la licitación BE), streaming add-on lanzado (según matriz COMER: M13 condicional / M12 después).
- **F3.4** Beta de datos públicos (agenda federativa si la licitación esta cerrada): modo "BTM Pro runs el torneo, TP recibe los datos" para organizaciones que temen tolería.

### Gates transversales (en el camino, no se posponen)
- **Firma de código** (#33, branch `installer-signing-phase0` en marcha) — bloqueante de TODA venta SBS hoy.
- **Licenciamiento offline Ed25519** (COMER §10) — decidir proveedor vs propio (fue pendiente en COMER §17).
- **Hardening security fase según plan** (crown jewel en el .asar, arranques by utilityProcess, etc.).
- **Legal #103:** contratos + **datos de menores gate** (bloqueante fuerteLEGAL hoy).
- **i18n ES completo en BTM Pro** (tabla real del producto que se vende en LatAm).

### Orden si hay capacidad limitada (1–2 devs)
`F0 → F1.2+F1.3 (crítico) → F2.1 (diferencial real) → resto`. La fase "everything-else" en paralelo solo si se puede. **Multi-day resilience (PLAN-UIUX Fase 10) sigue siendo requisito operacional** del día de torneo real; no se abandona.

**Línea BCM (F0.4 → PLAN-BCM.md):** independiente del calendario BTM Pro. Si hay un segundo dev, BCM puede avanzar en paralelo desde `app_bdm`; si no, entra después de F2. La conexión BCM→BTM Pro ("inscribir mi club" + roster push) es un gate de F3, así que ambos órdenes valen.

---

## 10. Riesgos y preguntas abiertas (para el equipo)

1. **Orden: BTM Pro SaaS vs "BTM Pro managed"** — solo-cloud exige pasarelas+hosting+cargos antes de su punta de curves; managed-first vende mientras se construye. **Propuesta: v1 managed-first, cloud detrás para el segundo trimestre tras el primer pago real.**
2. **Capacidad de banda:** con 1–2 devs, no se puede pelear Tourneysoft + Playinga + Visual Reality simultáneamente. Solución: F0+F1.2+F1.3+F2.1 FIRST. Es lo que este plan propone.
3. **Pagos en CO:** ¿Wompi (CO fácil), MercadoPago (LatAm), o Stripe internacional — o abstracto con 2 proveedores detrás? A milestone F1.3.
4. **Repositorio:** mismo monorepo (packages/tournament + shared) o repo nuevo aparte para BTM Pro? Propuesta: **shared sí, deployment separado** (independencia de release, identidad legal, dominio).
5. **Residuo ShuttleSync:** F0.1 lo lleva a 0 (artefactos stale, doc fase1, manifest, sw.js).
6. **Riesgo legal T&C de Visual Reality** (art. 10.2(h)): cualquier integración con su portal se limita a los XLSX que el organizador exporta y al link del torneo del propio usuario (§6.4). Revisar con abogado el copy de marketing "compatible con Tournament Planner" antes de publicarlo.
7. **`.tp` como candado federativo:** mientras la FA exija el archivo .tp, la fuga del organizador requiere el export de §6.2 en los shapes que TP importa — es el ítem que decide si un torneo federado puede correr 100% en BTM Pro. Prioridad de F1.5/F3.1.
8. **Lección multi-deporte:** TP cobra por deporte (un organizador tenis+badminton paga 2×). BTM Pro v1 es badminton puro por decisión, pero la puerta a multi-deporte queda señalada como móvil de pricing futuro.

---

## Anexo A — Informes completos recibidos
1. Deep dive **BadmintonPal** (bundle Angular/Firebase, app Android, fuentes de precios/quejas).
2. **Barrido de mercado mundial** (Tourneysoft, Playinga, Cup2000, Brakto…; benchmark y pain points con fuentes).
3. Deep dive **Tournament Software / Tournament Planner (Visual Reality)** — cerrado: familia de productos, pricing real (€106/año por deporte; Online €299 provisional), muralla federativa (BWF/England/Oceania BWF-license), **formatos de datos** (Excel nativo de players/ranking, `.tp` binario, páginas GUID públicas, "TS Services API" privada, riesgo T&C art. 10.2(h)), flujo de trabajo end-to-end y ventana estratégica (cierre web pública BWF 2025; KNLTB→Online TP 2026; roadmap SaaS "through 2028"). → §4, §6.
4. **Doc comercial local crítico** (COMERCIALIZACION v0.4, SOLID FACTS con números exactos).

*(Los informes se persisten en `docs/research/2026-10-18-*.md` durante F0.2.)*
