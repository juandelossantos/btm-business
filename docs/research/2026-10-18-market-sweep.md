# Barrido del mercado mundial — software de gestión de torneos de badminton (2026)

**Fecha:** 18-oct-2026 · **Procedencia:** informe del agente de investigación (fuentes verificadas por URL). Síntesis decisorio: `docs/PLAN-GESTION-TORNEOS.md` §5. informes companions de la familia: docs/research/2026-10-18-{tournament-software,badmintonpal}-deep-dive.md

---

## 1. La arquitectura real del mercado (pirámide de 3 capas)

1. **Federativo internacional:** `bwf.tournamentsoftware.com` (Visual Reality) — backend oficial BWF para registro de jugadores, sanción e inscripción. **La web pública de resultados se cerró el 7-ene-2025** (fan-facing migró a bwfbadminton.com + app Badminton4U): https://corporate.bwfbadminton.com/news-single/2024/10/21/closure-of-tournament-software-websites-from-7-january-2025
2. **Federativo nacional:** Tournament Planner licenciado por FA (España: Todostorneos.es es el mismo producto en ES), Cup2000 (Noruega; Dinamarca migró Cup2000→TP), BadmintonPlayer.dk, Club Locker (USA Badminton: membresías desacopladas del torneo), procesos semi-papel (RFEB: PDFs por email; Perú: voucher por Instagram/WhatsApp; Colombia: TP bajo el estándar BWF SH-6 de Fecolbadge).
3. **Club/local:** Excel + Google Forms + WhatsApp = el stack por defecto; ~30 micro-apps sin ganador claro. **La cap mejor definida para ganar deporte por deporte.**

## 2. Competidores badminton-específicos (más allá de BadmintonPal y TS)

| Herramienta | Perfil | Amenaza |
|---|---|---|
| **Cup2000** (cup2000.com, DK) | Desktop Windows desde 2000; oficial de Dinamarca (→ migró a TP) y Noruega (+DGI); draws auto con seeds, scheduling **con detección de conflictos**, PDFs, facturación, live score propio | Media-alta **en declive** — mide el estándar de features |
| **BadmintonPlayer.dk** | Portal nacional de Badminton Denmark: torneos, inscripciones, liga de equipos, ranking, "superbruger" presencial | Alta dentro de DK; no exporta global |
| **Tourneysoft** (tourneysoft.com) | SaaS badminton EN/ES: inscripciones con pagos, draws automáticos por formato, comunicación a participantes | **Media — el rival más directo por posicionamiento** (sin offline/LAN) |
| **Playinga** (playinga.com) | SaaS multi-deporte ES/EN con tier free: inscripciones, brackets, live score, standings; indexa "torneos badminton Medellín" | **Media-alta en LATAM** (idioma + gratis) |
| **Brakto** (brakto.com) | Web nueva: "software profesional de torneos, BWF scoring, live results, brackets gratis" — clon del posicionamiento BTM Pro | Media (temprana; vigilar SEO) |
| **Whatsscore** | Gestión torneos online (draws auto MD/WD/MX, live scores) | Baja-media |
| **Smashduck** | App iOS gestion de torneos de club (recién lanzada) | Baja-media |
| **Rackonnect** (India) | App de jugadores + hub de torneos; 50K+ jugadores del norte de India; software para organizadores | Media (geolocalizada; mercado India masivo) |
| **SliceWin** (slicewin.com) | App de club: round robin, Mexicano/rotación, rankings Elo; **"solo el admin necesita la app"**; gratis | Media en grassroots ("spreadsheet headache" es su marketing) |
| **TournMate / RACKETAPP / STADIUM** | Micro-tools de brackets/live score para club (multi-raqueta) | Bajas individualmente; ruido de fondo |
| **Rapid Brackets** (C1 Badminton, USA) | Nuevo bracket engine buscando "10 tournament directors" para piloto 2026 | Baja hoy, potencial medio plazo |
| **Competize** | App generalista ES torneos/ligas cualquier deporte | Baja-media (clubes ES) |
| **Todostorneos.es** | "Clon español" de Visual Reality (misma descarga/v de Gestor de Torneos de Bádminton v2026) | Media-alta en España (es el mismo producto) |

## 3. Soluciones federativas por país (sistema real en uso)
- **España (RFEB):** badminton.es con buscador propio + "Impresos FESBA" en PDF por email; herramienta estándar = Tournament Planner (Todostorneos).
- **Colombia (Fecolbane):** stack BWF (`bwf.tournamentsoftware.com`); PDF oficial "SH-6 — Badminton Tournament Planner (BWF)": https://badmintoncolombia.org/wp-content/uploads/2025/03/SH-6.pdf
- **Perú (FDPB):** web propia + redes; inscripción con voucher por Instagram/WhatsApp; torneos internacionales bajo sanción BWF/Panam. **Vacío de sistema propio.**
- **Ecuador/México (FEMEBA):** YouTube para transmisión; inscripciones fuera de plataforma.
- **Dinamarca:** fluctuó Cup2000→TP ya consumado; Noruega sigue en Cup2000.
- **Inglaterra (BE):** TP **obligatorio** para el circuito BE + membresía integrada (be.tournamentsoftware.com).
- **USA (USAB):** inscripciones junior/adultio en TournamentSoftware + membresías en **Club Locker** (dos sistemas inconexos = hueco).
- **Badminton Europe:** usa TS para el circuito junior/elite **y está licitando software propio** de membresía + sanción + torneos (enero 2026): https://www.facebook.com/badmintoneurope/posts/710731697765236 — ventana estratégica explícita.
- **Badminton Ireland:** migró datos/membría a **Sport:80** ("from fragmented data to strategic intelligence").

## 4. Benchmark "best in class" (deportes adyacentes)
- **Smoothcomp** (combates): drag&drop scheduling con estimación en vivo; brackets multi-formato; check-in QR; scoring tablet/TV; **livestream con overlays generados del scoreboard**; plataforma federativa white-label; pricing por atleta. — *hermana directa del roadmap BTM Pro.*
- **Scoreholio** (pickleball/cornhole): tablet por pista + **kiosk**; dashboards en cualquier TV; scoring desde teléfonos con QR; **cobros in-app**; cambios de formato en caliente; "announce mode"; offline degradado ("tablets lost connection = no problem"). La prueba de **1 organizador solo con 7 canchas**.
- **USTA Serve Tennis + Match! App (VTD):** plataforma federativa gratuita USTA + **Virtual Tournament Desk** (reporte de scores, notificación de asignación de cancha, match status, anuncios) — el oro del flujo organizador–jugador–espectador.
- **Sport:80:** 80+ federaciones nacionales; caso Badminton Ireland. Modelo "vender a la federación".
- **Playtomic:** torneos como actividad social de club (Americano/Mexicano/King of Court con pareo automático) — volumen recurrente.
- **Toornament:** planes por tamaño; **full data ownership** + API white-label en tiers altos.

## 5. Pain points de organizadores (evidencia)
1. **"Spreadsheet headache"** — RR 8 jugadores = 28 partidos a mano; rotación de dobles = "total nightmare" (SliceWin; r/BadmintonIndia mod-approved "without the spreadsheet headache").
2. **Live score de facto no existe** — sobre Tournament Software: *"at the mercy of individuals posting scores… can take hours"* (BadmintonCentral); reddit pregunta repetidamente *"does badminton have any live score app?"* sin respuesta seria fuera del pro. **Sin API pública de Visual Reality.**
3. **La barrera de adopción gana a las features** — *"the biggest fear of organizers is making older players download an app"* (SliceWin).
4. **Cambios de última hora rompen torneos** — *"bracket changes during play can feel operationally rigid"* (gitnux 2026) → el draw editable en caliente es un ganador.
5. **Cobros desde la app** es tabla obligatoria (Scoreholio/Smoothcomp/USTA/Playtomic/Sport:80); el badminton cloud casi nadie.
6. **Genéricos = workarounds de bádminton** — TeamSnap sin brackets/seeding; SportsEngine "scoring requires workarounds" (gitnux 2026).

## 6. Evidencia de switching (incumbente vulnerable)
- BWF cerró la web pública de TS (2025) → la experiencia de fans ya se decidió lejos de Visual Reality.
- Dinamarca Cambió Cup2000→TP con plan manual de apoyo a clubes → **las FAs cambian de software cuando un vendor les ofrece más.**
- Badminton Europe **licitando**; Ireland con Sport:80; USA desacoplado.
- Nadie ha desplazado a TP en torneos federativos sancionados (rosters/draws/results sanctioned). El flanco de entrada: **live score real, mobile-first, operación de día D** — con export XML para la FI federativa mientras dura la coexistencia.

## 7. Top 10 must-have para "best in market" (derivados del barrido)
1. Import/export completo de Tournament Planner (players/inscripción/draw/results/seeds).
2. Live score punto-a-punto real vía API/árbitro (¡ya vía SBS!).
3. Offline/LAN-first del día del partido (ya vía SBS) + degradación elegant padre.
4. TV por pista + dashboard central + "announce mode" (ya vía SBS; falta el announce).
5. Scheduling drag&drop con conflictos + estimación de duración.
6. Draw editable en caliente (replace/W.O./bye/reseed).
7. Inscripción/regalos/entry fees **con cobros en app**.
8. Web pública del torneo sin app (draws, timetable, standings, live TV) + notificaciones.
9. One-operator experience (1 dispositivo por pista; nada obligatorio para jugadores mayores).
10. Programa de organizadores/federación: subdominio, rankings por temporada, export XML/CSV, white-label.
**#11 (bonus):** formato social de club (Americano/Mexicano/rotación con pareo automático) — el volumen Secular entre torneos federados.
