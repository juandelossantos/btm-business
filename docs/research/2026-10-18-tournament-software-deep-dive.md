# Deep dive: tournamentsoftware.com / BWF Tournament Planner (Visual Reality)

**Investigación web del 18-oct-2026 · paso de persistencia F0.2 (fuentes: informe del agente de investigación, verificado por acceso directo al sitio, manuales en PDF y búsquedas en la web). Doc primario de síntesis: `docs/PLAN-GESTION-TORNEOS.md` (§4/§6).**

---

## 0. Resumen ejecutivo

1. **Titular del año:** el BWF **cerró** los sitios públicos de resultados (`bwf.tournamentsoftware.com`, `bwfpara.tournamentsoftware.com`) el **7-ene-2025** y trasladó el fan-facing a `bwfbadminton.com` + app **Badminton4U**, **pero mantuvo Tournament Planner como backend** de sancionamiento, registro de jugadores y entradas de las Member Associations.
   Fuente: https://corporate.bwfbadminton.com/news-single/2024/10/21/closure-of-tournament-software-websites-from-7-january-2025
2. **No existe versión online en producción para badminton:** la página "Online Tournament Planner" lleva años en "Something new is coming!!" con precio provisional de **299 €/año**. El cliente Windows-only sigue siendo el producto principal (versión 2026; requisitos Win 8/10/11).
3. **Intercambio de datos real:** (a) archivos **Excel** de importación que la propia herramienta define, (b) archivo binario propietario **`.tp`**, (c) sync propietaria con el sitio (Internet > Publish / Entries / Download Results). **No hay API pública ni DTD/XML documentados**; la "TS Services API" solo existe como módulo privado de la plataforma de federaciones.
4. **El moat real no es software:** es la base de datos de jugadores + ranking por federación + árbitros entrenados + 30 años de archivos. Migrable federación por federación, no en bloque.

---

## 1. La empresa y la familia de productos

- **Entidad:** Visual Reality B.V., Comeniusstraat 5, 1817 MS Alkmaar (NL), KvK 39067526. Derecho neerlandés, tribunales de Ámsterdam, soporte **solo L-V 9:00–17:00**, productos "AS IS… no liable for any data losses" (T&C arts. 8–9): https://www.tournamentsoftware.com/support/terms-and-conditions
- **Historia/escala:** 30+ años ("two college developer friends…"), cero publicidad, "200+ federations in 160 countries", "40,000 tournaments every year", ~40 empleados (About: https://www.tournamentsoftware-company.com/). Vacante activa C#/.NET + React → stack confirmado.
- **Catálogo (en vivo):**

| Producto | Qué es | URL |
|---|---|---|
| Tournament Planner (TP) | App **Windows** por deporte (tenis, pádel, badminton, squash, TT, darts, pickleball, judo) | https://www.tournamentsoftware.com/product/page.aspx?id=3&s=2 |
| Tournament TV | App tipo PowerPoint para pantallas de sala | id=17 |
| Visual LiveScore | Live score punto-a-punto sobre internet | idem |
| TP Network | Segundo exe LAN para Match Control (IP+password al PC con TP; arrastra partidos/pagos/check-in/resultados; no publica ni recibe entradas) | article EC92CDBE-… |
| League Planner | Ligas de equipos | catálogo |
| Flex/Box Ladders | Ladders con ascensos/descensos autofinanciados | id=112 |
| Rankings | Rankings online a partir de torneos publicados | id=17 |
| **Online Tournament Planner** | SaaS "coming soon", €299/año provisional | id=117 |
| turnier.de | Instancia alemana | https://www.turnier.de |
| Plataforma federativa | White-label por federación: CMS, SSO, pagos, subdominio, **TS Services API (privada)** | https://www.tournamentsoftware-company.com/for-federations/ |

- **Conector:** el TP de escritorio **sube** al multi-tenant `*.tournamentsoftware.com` (temas por federación: themes `badmintonengland`, `bwf`, `ba`… en `static.tournamentsoftware.com/content/styles/css/themes/<fed>/styles.css`). Publish manual ("Internet > Publish") + auto-publish de **resultados solamente** desde v2018.1 (changelog: https://www.tournamentsoftware.com/product/article.aspx?s=2&id=E0524F9C-E95F-4A82-93C9-316BB47FA6C6).

---

## 2. Entrenchment federativo

### 2.1 BWF
- Manual **BWF Referees Level 2**: "This software has become **ubiquitous** for badminton tournaments throughout the world" + competencia obligatoria en TP (pdf pp.35–37): https://badmintonoceania.org/wp-content/uploads/2020/07/BWF-Referees-Manual-Level-2-English_Digital-compressed.pdf
- **Draw Management Guide** (v5.10 feb-2020) es material oficial de referees BWF, sacada del propio TP.
- Los draws de Grado 1 los hace el Cuerpo de Referees BWF en TP; India (BAI) hace sus sorteos de ranking "through the Tournament Planner software": https://www.badmintonindia.org/download/regulations/FINAL%20REVISED%20COMPETITION%20REGULATION.pdf
- **Backend intacto:** las entradas de las MAs se hacen en `bwf.tournamentsoftware.com/sport/tournament.aspx?id=…` (verificado hoy en el calendario BWF): https://corporate.bwfbadminton.com/events/calendar/

### 2.2 Mapa de federaciones (subdominios activos verificados)
`bwf`, `bwfpara`, `fir` (racketlon — manual propio), **`be` (Badminton England — obligatorio para su circuito; incluso distribuye la descarga de TP: https://www.badmintonengland.co.uk/competition/adult-competition/tournament-organisers)**, `ffbad` (Francia), `badmintondenmark` (Dinamarca — migró desde Cup2000), `badmintonscotland`, `bam` (Malasia), `ba` (Australia), `bnz` (NZ — con manual oficial propio), `badmintoncanada` + `badca`, `badmintonfinland` (con ranking público), `sba` (Singapur), `bat` (Tailandia), `competitions.be` (Bélgica white-label con dominio propio).

### 2.3 Barrera a fondo
- Circuito europeo grado 2/3: **"It is mandatory… to use the Tournament Planner. Organisers must use the 'BWF-license' belonging to its Member Association"** (€500/torneo en el Elite Circuit 2026): https://badmintoneurope.com/documents/d/corporate/elite-circuit-regulations-valid-1-january-2026-?download=true
- **Badminton Oceania setea el TP file por el organizador** y controla el master: https://badmintonoceania.org/wp-content/uploads/2026/05/Oceania-Grade-3-Host-Organiser-Guidelines-Updated-May-2026.pdf
- Maharashtra (India): TP obligatorio para campeonatos estatales: https://www.scribd.com/document/854893772/UPDATED-mba-tournament-guidelines-2022-2023-11-08-2022
- **El feed de TV/overlays no es feature de TP:** es contrato con data providers comerciales (Sportradar, Data Sports Group…), alimentado por la cadena TP→BWF→bwfbadminton. No hay feed público documentado.

---

## 3. Modelo comercial (quién paga, cuánto)

| Producto | Precio 2026 | Comentarios |
|---|---|---|
| TP desktop (por deporte) | **106 €/año** | "includes free updates and usage of tournamentsoftware.com until January 1st of the next year" — pago directo PayPal/Visa/MS en la web: https://www.tournamentsoftware.com/product/order.aspx?s=2 |
| Flex/Box Ladder | **107 €/año** | por deporte |
| Online TP | **299 €/año provisional** | "Final price has not been set yet" |
| LiveScore / TournamentTV / CMS web / API / SMS / pagos / white-label | **no público** | cotización por propuesta ("Platform proposal") |

- Quien paga: el **organizador o la federación**. BWF/Oceania proveen la licencia BWF preconfigurada (el organizador no paga la licencia en torneos de ese circuito).
- KNLTB (tenis/pádel NL) fuerza el **Online Toernooiplanner** desde el 1-ene-2026: primera migración desktop→SaaS real del ecosistema.
- **Lección por deporte:** licencia per-sport — un organizador de tenis+badminton paga 2×.

---

## 4. Formatos de datos (crítico para migración)

### 4.1 Nativo: `.tp` binario (un archivo por torneo)
- Ruta local `C:\Users\…\Documents\Tournaments\<name>.tp`; migración entre PC = exportar TP file / desinstalar / instalar / importar. "TP is an offline system… needs to be regularly synchronised"; versiones por email "come easily out of sync" (manual FIR: https://www.racketlon.net/wp-content/uploads/2017/06/FIR-Tournament-Software-Tournament-Planner-Handbook-30.4.2017-1.pdf).
- Las confederaciones distribuyen plantillas ya configuradas (Tournament Template Defaults).
- **No hay DTD/XML export documentado.**

### 4.2 Importar INTO TP (superficie documentada)
- **Excel base de jugadores**: `Player > Player Database > Import to Player Database` — descargado de la federación ("Download Players for TP as at…"), campo **Member ID obligatorio** + nombres, género, DOB, tabla Events con checkbox por categoría (manual NZ/BNZ quick-start: https://tennis.kiwi/assets/Tournament-Planner-quick-start-guide.pdf).
- **Excel de ranking**: `Player > Import Ranking` con "Publication Date" + Reset.
- **Entradas web**: `Internet > Entries` (requiere configurar antes `Internet > Publish > Online Entry = On`, entry date, Member ID required, etc.).
- **Resultados en línea**: `Internet > Download Results` (artículo oficial: https://www.tournamentsoftware.com/product/article.aspx?s=2&id=8D361D7F-EC19-45A8-85C5-FDBBEF3200A8).
- Existencia de menú `Import Results…` / `Import Schedule…` (formato no documentado — a muestrear).

### 4.3 Extraer AWAY de TP (migración hacia BTM Pro)
1. **Los XLSX de TP** (players/ranking) — formato estable porque TP y las FAs los usan entre sí. La aduana de entrada del ecosistema.
2. **Páginas públicas con URLs GUID estables:** `/{fed}.tournamentsoftware.com/sport/tournament.aspx?id={GUID}` con subpáginas `draws.aspx`, `events.aspx`, `players.aspx`, `seeds.aspx`, `clubs.aspx`, `matches`, `legacymatches.aspx?id=…&d=YYYYMMDD` por jornada; URL nueva `/{fed}.tournamentsoftware.com/sport/tournament?id={GUID}` (redirección). HTML server-rendered.
3. **Reports/printables:** poule standings, matches per day, match cards, players, entries per event, draw list, first matches, payments, court schedule, winners.
4. **No hay API pública:** el buscador del portal es POST `/find/tournament/DoSearch` server-rendered (prohibido integrar — ver guardrails); solo endpoint JSON público hallado: `/api/UserListAPI/ToggleUserListItem` (favoritos).
5. **TS Services API** solo por contrato de federación.

### 4.4 ⚠️ Riesgo legal
T&C **art. 10.2(h)**: prohibido usar el servicio "to develop, train or benchmark a competing product or service"; 10.2(b) no reverse engineering. → **No scrapear el portal ni su búsqueda;** solo XLSX que el organizador exporta + el link del torneo del propio usuario. Full: https://www.tournamentsoftware.com/support/terms-and-conditions

### 4.5 Live scoring
- Visual LiveScore (web, server-side, sin spec pública de websocket);暴露 el estado del día pero mediado por personas: comunidad — *"at the mercy of individuals posting scores… can take hours"*. TP Network hace Match Control LAN (cableada, IP+password).

---

## 5. Flujo end-to-end estándar

1. Federación/confederación crea el **TP file plantilla** (eventos/categorías, tarifas, pistas, Regulations, currency) — a veces centralizado (Oceania se lo manda al organizador).
2. Inscripción **online** (cuentas federativas en `bwf.tournamentsoftware.com`); "Players do not register directly into TP… they cannot see their registration until it has been first processed by TP" (manual FIR).
3. `Internet > Entries` baja los registros → **Tournament/Player Validation** (nacionalidades erróneas, IDs, variantes de nombre — dolor frecuente y función nativa).
4. Import de ranking (Excel, publication date) → seeding automático + separación por país + draws **Monrad** + qualifiers (manual FIR "Making Draws"; idéntico en BWF L2).
5. **Scheduling** — el confeso punto de dolor: "easily the most difficult and time-consuming single job… **TP is not able to do everything automatically**… You might find it easier to schedule matches using an Excel and transfer to TP afterwards" (manual FIR).
6. Reports de organización exigidos por BWF (shuttle consumption, durations, officials' duties).
7. Match day: match cards, check-in, **TP Network** (Match Control LAN), LiveScore a la web, email de first matches (Badminton England RTO doc 2026).
8. **Publish continuo** tras cada resultado (auto-publish solo resultados desde 2018.1; draws a mano). El referee es responsable de enviar el TP file actualizado a BWF/CC/MA **cada día** (BWF L2).
9. Cierre: Final Positions, export para ranking, backup del .tp, envío del archivo a la federación (Oceania/FIR exigen protocolo).

---

## 6. Debilidades y ventana de ataque

- **Windows-only desktop en 2026**; nada Mac/iPad/Linux; el Online es vaporware ("coming soon", precio sin fijar).
- **Changelog 2018–2025:** sustancialmente "small bugfixes" por año (2025.1: OAuth Gmail; "2025.3: only updates in the Dutch planners").
- **Sync por archivo frágil** (.tp por email; desincronización multi-versión documentada; Oceania prohíbe múltiples accesos al master).
- **Data hygiene dolorosa** (Player Validation como parche a datos sucios de FAs).
- **Soporte L-V 9–17 + "AS IS / no liable for data loss"** — pésimo para el día del torneo.
- Comunidad: reddit ene-2025 "Alternative for tournamentsoftware.com" (*"they rarely get the match results posted on time"*) y may-2025 "Tournament software alternatives" (clubes buscando algo ligero).
- **Ventana:** (1) BWF ya separó el fan-facing (2025); (2) KNLTB migra a Online TP (2026); (3) roadmap corporativo "through 2028" reduciendo tech debt. **2–3 años de cielo abierto en el segmento organizador si alguien llega antes con SaaS + día de partido real.**

---

## 7. Ecosistema adyacente
- Genéricos listados junto a TP en "mejor software BWF": Challonge, SportEasy, Tourney Machine (matchboxsoftware.com/blog/5-best-bwf-tournament-software-in-2024/).
- Tourneysoft (SaaS badminton EN), MatchBox, Tournament Planner "homónimo" de Ardent Sport (Android: com.ardentsport.tournamentplanner — ojo por colisión de nombre/SEO), Smoothcomp, Sport80, y el patrón real de club pequeño = **hoja de cálculo + Challonge** (cola larga completamente servida a mano hoy).

## Anexo — materiales de referencia extraídos durante la investigación
`/tmp/opencode/tp/` (temporal): `fir.txt` (manual FIR completo), `qs.txt` (quick-start NZ), `ref2.txt` (BWF Referees L2), `oc3.txt` (Oceania Grade 3 2026) + HTML de páginas producto/order/terms. A copiar a `docs/research/assets/` o re-descargar si se necesitan los textos completos.
