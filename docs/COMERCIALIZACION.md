# COMERCIALIZACIÓN — BTM PRO

**Versión:** 0.4 · **Estado:** documento de evaluación (sin plan de acción) · **Fecha:** 2026-10-15
**Ámbito:** modelo de negocio, planes, servicio gestionado, comisión de enganche, licenciamiento, distribución, control de copia y **marco tributario/contractual (Colombia)**.
**Audiencia:** equipo BTM PRO.

> ⚠️ **Aviso:** las secciones tributaria (§12) y contractual son **orientativas**, no asesoría legal ni contable. Deben validarse con un **contador público** y un **abogado** antes de facturar. La normativa colombiana 2026 es volátil (el Decreto 572 de 2025 fue suspendido y reactivado dentro del mismo año).

---

## Cómo leer este documento

- Es un **documento de evaluación**, no un plan de acción. El plan se define **después** de evaluar las ideas (§14) con la matriz (§15).
- Los precios de **software** (§6) y del **servicio** (§7) son **hipótesis a validar**. Dos decisiones ya están tomadas por el equipo y este documento las **analiza y cuestiona**:
  - **El operador se paga COP 2.000.000 por evento**, y ese valor se **suma** al valor del producto/servicio.
  - **Hay una comisión para el "enganche"** (club o entrenador que recomienda y ayuda a vender).
- Convenciones:
  - **COP** = peso colombiano. Se asume **~4.000 COP/USD** y **UVT 2026 = $52.374** (§16).
  - Esfuerzo: **S** (bajo) · **M** (medio) · **L** (alto).
  - Impacto: **★** bajo · **★★** medio · **★★★** alto.
- Regla de oro:
  > **El software offline no se protege, se empaqueta.** El ingreso no viene de impedir la copia; viene de vender lo que no se puede copiar (servicio, soporte, pauta, hardware, relación institucional).

---

## 1. Resumen ejecutivo

BTM PRO es hoy una **capa de cancha**: marcador en vivo por TV, árbitro desde el celular, y todo corriendo en **una máquina sobre la LAN, sin internet obligatorio**. El código además **importa roster y partidos desde exports de Tournament Software**: está diseñado para **complementar** un motor de torneo, no para reemplazarlo.

Estrategia:

1. **No competir** con Tournament Software / Brakto en registro y draws. **Enchufarse** a ellos.
2. **Vender el día del partido**: confiabilidad, TV por cancha, árbitro, y un **operador humano**.
3. **Monetizar en tres ejes** (licencia / servicio / add-ons) **+ dos flujos que se suman**: el **pago del operador** (2M) y la **comisión de enganche**.
4. **Control de copia realista**: legal + activación offline firmada + degradación elegante. **Nunca** bloqueo online.
5. **Cumplir la ley** (IVA, retención, facturación, habeas data) — que **sube el precio final** al cliente y **no se puede ignorar**.

**Conclusión preliminar:** el activo más defendible no es el software, es el **servicio + la relación + la pauta en pantalla**.

---

## 2. El producto (contexto técnico que condiciona el negocio)

| Aspecto | Hecho | Implicación comercial |
|---|---|---|
| Despliegue | Electron macOS/Windows, **una instalación = una máquina** | Licencia natural por instalación (node-locked) |
| Red | LAN + Socket.io; **sin internet obligatorio** | El cobro online no puede ser requisito el día del partido |
| Datos | **SQLite embebido** | Fácil de respaldar; difícil de multi-tenant en la nube |
| Estado | **Event sourcing**; backup firmado por instalación (HMAC) | Confiabilidad como propuesta de valor |
| TVs | Pantallas por cancha + **TV Global** + Lite (ES5) | **Superficie publicitaria** (patrocinadores) |
| Interop | Importa roster **y** partidos de **Tournament Software** | **Complementario**, no competidor |
| Idioma | Admin ES/EN; TV/árbitro/viewer en inglés | Deuda de i18n para ligas hispanas |
| Licencia legal | EULA propietaria (licencia, no venta) | La base legal ya existe |
| Firma de código | **Instaladores NO firmados** | **Bloqueante** para vender a terceros |
| **Datos de menores** | Roster con fecha de nacimiento de jugadores Sub-7 a Sub-19 | **Restricción legal fuerte** (ver §12.6) |

**Ausencia importante:** **no hay motor de draws/brackets propio**.

---

## 3. Posicionamiento

> **"La capa de cancha que no depende de internet, con el marcador en cada TV y alguien que lo hace funcionar."**

- **Hueco:** los competidores son web-first y dependen de la red del venue. Nadie en bádminton LatAm ofrece *TV + árbitro móvil + offline + operador*.
- **No competimos:** registration, pagos, draws/brackets, ranking.
- **Sí competimos:** marcador en vivo, árbitro, display, resultados, pauta.
- **Activo diferencial:** el **muro de patrocinadores** convierte cada TV en **inventario publicitario**.

---

## 4. Mercado de referencia (anclas de precio)

| Jugador | Categoría | Modelo | Precio |
|---|---|---|---|
| Tournament Software | Motor de torneo | SaaS web | ~€300/año; planes ~$299–999/año |
| Brakto | Motor de torneo (racquet) | SaaS | Free ≤10 · Evento **$25 + $2/jug** · Club **$99/mes** o ~$999–1.188/año |
| Score7 / Challonge | Genérico / esports | SaaS | Free / ~$15/mes |
| Playinga / Tourneysoft | Venue + torneo | SaaS | Planes por club |
| WeighBook (pesca) | Servicio de resultados | Per-evento | **$2/angler/torneo** |
| ProTSR / ChronoTrack / Seiko | **Servicio gestionado** | Cotización por evento | Miles de USD/evento |

**Lecturas:** el deporte se cobra **per-evento o per-jugador**; el **servicio gestionado** es un negocio probado; el ancla de software está en cientos de USD/año y la de servicio en miles de USD/evento.

---

## 5. Arquitectura de ingresos y flujo del dinero

### 5.1 Los tres ejes

| Eje | Qué se cobra | Recurrencia | ¿Copiable? |
|---|---|---|---|
| **A. Licencia de software** | Instalación / temporada / torneo | Anual o por evento | Sí |
| **B. Servicio BTM PRO** | Setup + router + PC + coordinación + soporte remoto | Por evento | No |
| **C. Add-ons / pauta** | Streaming, patrocinadores, resultados, SLA | Por evento o anual | No |

### 5.2 Los dos flujos que se **suman**

- **Operador:** **COP 2.000.000** por evento (pago del humano). Se **suma**.
- **Comisión de enganche:** **%** para quien recomienda y ayuda a vender (§8).

### 5.3 Flujo del dinero

```
CLUB PAGA
├── Licencia de software ............... eje A
├── Servicio BTM PRO ............... eje B  ← nuestro margen
├── Operador en sitio .................. COP 2.000.000  ← pago del operador
├── Add-ons ............................ eje C
├── Viáticos ........................... se reembolsan
└── IVA 19% (si somos responsables) .... se traslada a la DIAN

NOSOTROS PAGAMOS
├── Operador ........................... pass-through
├── Comisión de enganche (X%) .......... CAC (sale del margen)
├── Equipo, logística, impuestos (ICA, renta)
└── Retenciones que nos practican ...... anticipo de renta (se descuenta al declarar)

MARGEN = (A + B + C) − comisión − costos − impuestos
```

**Regla contable:** la **factura va discriminada por líneas**. Si el club aprende que "todo = un número", nunca se vende el plan anual ni se justifica el operador.

---

## 6. Eje A — Planes de software (hipótesis a validar)

| Plan | Alcance | Precio idea (COP) | Objetivo |
|---|---|---|---|
| **Free / Club Base** | 1 instalación, ≤2 canchas, torneos internos, marca visible | $0 | Adopción, demo |
| **Torneo (pay-per-event)** | 1 torneo (≤4 canchas / ≤2 días), código | 150.000–400.000 | Organizador ocasional |
| **Club / Liga — Torneos ilimitados** | Anual, N instalaciones, ilimitado, sin marca | 1.500.000–4.000.000/año | **Plan estrella** |
| **Federación / Enterprise** | Ilimitado, white-label, SLA | A medida | FCB, ligas |
| Add-on: resultados públicos | Página sin marca + dominio | 300.000–800.000/año | Recurrencia |
| Add-on: muro patrocinadores | Pauta en TV/Global + marquee | Rev-share o 200.000–500.000/evento | Incremental |
| Add-on: streaming | WebRTC + viewers | 200.000–600.000/evento | Premium |

**Análisis:** "torneos ilimitados" encaja mejor con offline; el per-torneo tiene más roce el día del partido; riesgo de canibalización entre ambos.

---

## 7. Eje B — Servicio BTM PRO + operador

### 7.1 Modelo decidido por el equipo

| Elemento | Provee | Naturaleza |
|---|---|---|
| **TVs** | El club | — |
| **Servicio BTM PRO** | Nosotros | Ingreso/margen |
| **Operador en sitio** | Nosotros (fundador o certificado) | **COP 2.000.000** (se suma) |
| **Transporte/alimentación/hospedaje** | El club | Reembolso |

### 7.2 Composición del precio que ve el club

```
VALOR DEL PRODUCTO/SERVICIO
+ COP 2.000.000 (operador)
+ comisión de enganche (recomendado: sale del margen, no del cliente)
+ viáticos
+ IVA 19% (si aplica)
= TOTAL QUE PAGA EL CLUB
```

### 7.3 Evaluación crítica

**Bien:** TVs por el club (cero capex); router+PC por nosotros (margen + foso antícopia); el servicio es lo único no copiable; separar el operador permite vender con y sin él.

**Cuestionamientos:**

1. **Operador a 2M fijos:** generoso para 1 día, insuficiente para 4 días. Recomendado: base **2M hasta 2 días**, **+1M por día adicional**.
2. **Punto único de falla:** falta **operador de respaldo + soporte remoto**.
3. **"El club cubre transporte/alimentación/hospedaje" es ambiguo** → viáticos como **línea fija** y **nosotros reservamos**.
4. **"Gestión de todo el torneo" es alcance abierto** → definir por escrito (§7.6/§7.7).
5. **El piso sube ~2M**: el servicio gestionado queda en ~3M+ por evento; hace falta el **tier de autoservicio** (§7.9).

### 7.4 Rejilla de precios

**Servicio BTM PRO (sin operador)** — nuestro margen:

| Concepto | COP |
|---|---|
| Base: ≤4 canchas, ≤2 días | 1.000.000 |
| Día adicional | +500.000 |
| Bloque extra de canchas (por cada 4) | +250.000 |
| Soporte remoto en vivo | incluido |

**Operador (se suma):**

| Concepto | COP |
|---|---|
| Base: ≤2 días | 2.000.000 |
| Día adicional | +1.000.000 |

### 7.5 Ejemplos (sin viáticos ni IVA)

| Escenario | Servicio | Operador | Licencia | Total aprox. |
|---|---|---|---|---|
| **Autoservicio** 1 día, 3 canchas | 1.000.000 | — | anual o per-torneo | **1.000.000 + licencia** |
| **Gestionado** 1 día, 3 canchas | 1.000.000 | 2.000.000 | 300.000 | **~3.300.000** |
| **Gestionado** 2 días, 4 canchas | 1.000.000 | 2.000.000 | anual | **~3.000.000 + anual** |
| **Regional** 3 días, 8 canchas + streaming | 1.500.000 | 3.000.000 | anual | **~5.000.000+** |
| **Nacional** 4 días, 12 canchas + streaming + resultados | 2.000.000 | 4.000.000 | enterprise | **cotización (7M+)** |

### 7.6 Alcance que SÍ incluye

Levantamiento del venue · instalación y prueba de router+PC+software (día previo) · carga/verificación de roster y programación · configuración de TVs y TV Global · **operación en vivo** · backup firmado y entrega de resultados · soporte al árbitro y al organizador.

### 7.7 Fuera de alcance

Registro/inscripciones y cobro · **draws/brackets** · árbitros · decisión de programación y apelaciones · TVs, soportes, cables, dispositivos de cast y energía · internet del venue.

### 7.8 Requisitos que el club debe cumplir

**TVs con navegador o dispositivo de cast por TV** · energía por TV y puesto de operación · espacio para el operador · acceso el día previo · **contacto único** con decisión · internet si se quieren resultados/streaming.

### 7.9 Variantes (modularidad del operador)

| Variante | Escalabilidad | Precio |
|---|---|---|
| **Autoservicio** | Alta | Sin operador |
| **Llave en mano** | Baja | + 2M |
| **Operador remoto** | Media | intermedio |
| **Certificación de operadores** | **Alta** | operador tercero |
| **Retainer para ligas** | Media | anual |

### 7.10 Condiciones comerciales

50% al reservar / 50% el día del evento · penalidad por cancelación tardía (~25%) · **viáticos como línea separada** · equipo en **comodato** · datos del club · responsabilidad definida en contrato.

---

## 8. Programa de enganche (comisión de referidos)

### 8.1 Objetivo

Convertir la experiencia y credibilidad de un **club, entrenador u organizador** en un canal de ventas con incentivo.

### 8.2 Roles

| Rol | Qué hace |
|---|---|
| **Embajador** | Recomienda / pasa el contacto |
| **Socio comercial** | Acompaña y **cierra** |
| **Operador certificado** | Opera (no necesariamente vende) |

### 8.3 Estructura de comisión (hipótesis)

| Modalidad | Base | Comisión |
|---|---|---|
| **Embajador** | Primera anualidad del software o 1 evento | **8–10%** |
| **Socio comercial** | Valor facturado del primer contrato | **12–18%** |
| **Recurrente** | Anualidades siguientes | **3–5%** |

### 8.4 Reglas de oro

1. **Se paga contra cobro efectivo**, no contra la venta.
2. **Sale del margen, no se suma al cliente** (es CAC).
3. **No hay comisión** sobre leads ya activos ni sobre descuentos.
4. **Tope de sanidad:** ≤15% del margen.
5. **Conflicto de interés:** si el referidor también decide la compra, **revelarlo**.
6. **Pago por factura o bonificación**; revisar tratamiento fiscal (§12).
7. **Puede ser en especie** (crédito en su licencia) para clubes.

### 8.5 Crítica / riesgos

Encarece si el comisionista presiona descuentos (→ comisión sobre **precio de lista**); colisión con el operador (roles distintos); proceso de pago lento ahuyenta referidores; riesgo de canal (club-embajador vs venta directa a su liga).

---

## 9. Eje C — Control de copia y uso

**Realidad:** con software instalado y offline, **el 100% de protección no existe**.

### 9.1 Capas

| # | Capa | Efectividad | Costo |
|---|---|---|---|
| 1 | **Legal** (EULA) | Alta con clientes formales | Cero |
| 2 | **Activación offline firmada (Ed25519)** | Alta | Medio |
| 3 | **Node-locking** | Media | Bajo |
| 4 | **Códigos por torneo** | Media | Medio |
| 5 | **Detección de reloj** | Baja | Bajo |
| 6 | **Degradación elegante** | UX, no seguridad | Bajo |
| 7 | **Lo que no se copia** (servicio, soporte, pauta, hardware, relación) | **La mayor** | — |
| 8 | **Appliance** (mini-PC/router) | Media + margen | Medio |
| 9 | **Telemetría opt-in** | Baja | Bajo |

### 9.2 Anti-patrones

❌ Bloqueo online obligatorio · ❌ DRM agresivo · ❌ sobre-invertir en anti-crack · ❌ confundir protección con cobro.

### 9.3 Filosofía

> La licencia **gatea features y soporte, nunca deshabilita la app**. La revocación muerde del lado servidor; el check local es UX.

---

## 10. Licenciamiento offline — diseño técnico (referencia)

### 10.1 Flujo de activación

1. **Compra** → *license key* (`SS-XXXX-XXXX-XXXX`).
2. **Primer arranque** → **fingerprint estable** (machine-id + placa/CPU).
3. **Solicitud offline** → `activation-request.json` por WhatsApp/email.
4. **Emisión** → `license.lic` **firmado con Ed25519**.
5. **Verificación** → clave pública embebida, **sin red**.

### 10.2 Estructura de `license.lic`

```json
{
  "v": 1,
  "licenseKey": "SS-XXXX-XXXX-XXXX",
  "customer": "Club Deportivo ...",
  "plan": "club-annual",
  "entitlements": {
    "installations": 2,
    "tournaments": "unlimited",
    "maxCourts": 12,
    "publicResults": true,
    "sponsors": true,
    "streaming": true
  },
  "issuedAt": "2026-10-15T12:00:00Z",
  "notAfter": "2027-10-15T12:00:00Z",
  "machine": { "fingerprint": "sha256:..." },
  "signature": "ed25519:..."
}
```

### 10.3 Código por torneo

`tournamentCode = HMAC-SHA256(tournamentId, licenseKey)`, canjeable **offline**; consumo registrado en `LicenseUse`; reconciliación diferida.

### 10.4 Estados y degradación (nunca brick)

`licensed` · `trial` · `grace` · `limited` (marca de agua, límite de canchas, sin resultados/patrocinadores) · `clock_untrusted`.

> **La licencia jamás interrumpe el día del partido.**

---

## 11. Distribución y canales

1. **Venta directa founder-led** (Chía como caso; cierre por WhatsApp).
2. **Federaciones y ligas como canal** (FCB, ligas, Indeportes, IMRD, Juegos Nacionales/Intercolegiados).
3. **Clubes y academias** → plan anual ilimitado.
4. **Revendedores** (timing/scoring) → white-label.
5. **Programa de enganche** (§8).
6. **Firma de código** → prerequisito.
7. **Ecosistema** (app_bdm + web) → upsell.

---

## 12. Aspectos tributarios y contractuales (Colombia) — NUEVO

> Orientativo. Validar con contador público y abogado. UVT 2026 = **$52.374**.

### 12.1 Impuestos que afectan el precio

| Tributo | Tarifa / Regla | Impacto en el precio final |
|---|---|---|
| **IVA** | **19%** (tarifa general, art. 468 ET). El software estándar es **gravado**. | **Sube 19%** lo que paga el club (si somos responsables de IVA). Se traslada a la DIAN, no es ingreso nuestro |
| **Exclusión IVA** — hosting/cloud (art. 476 num. 21-22) | Excluido | **No aplica** a nuestra licencia on-premise |
| **Exclusión IVA** — licencias de software para *desarrollo comercial de contenidos digitales* (art. 476 num. 20) | Excluido | **No aplica**: BTM PRO no es herramienta de creación de contenido. **Nuestra licencia es gravada al 19%** |
| **Exportación de servicios** (art. 481 lit. c ET) | **0% exento** (con derecho a devolución) | Solo si el servicio se usa **exclusivamente en el exterior** por quien **no tiene negocios en Colombia**. Requiere registro y soportes |
| **Renta** | Según régimen | Se paga sobre la utilidad, no sobre el precio |
| **ICA** (municipal) | ~**4–10 por mil** sobre ingresos brutos (varía por municipio y actividad) | Costo real; verificar **Chía/Cundinamarca** y dónde se registra la actividad |
| **Régimen Simple (RST)** | Opcional: consolida renta + ICA. Tarifas **1,8%–14,5%** sobre ingresos brutos (Grupo 4, servicios profesionales: **4,9%–14,5%**) | Puede reducir la carga total; **el IVA NO está incluido**. Requiere RUT resp. 47 y estar al día |

### 12.2 Retención en la fuente (el club nos retiene al pagar)

| Concepto | Tarifa 2026 | Base mínima |
|---|---|---|
| **Licenciamiento o derecho de uso de software** | **3,5%** | $1 |
| **Servicios informáticos** (análisis, diseño, desarrollo, implementación, mantenimiento, ajustes, pruebas, documentación, web, consultoría informática) | **3,5%** | $1 |
| **Servicios generales** | **4%** (declarante) / **6%** (no declarante) | **2 UVT ($104.748)** desde 1-jul-2026 |
| **Honorarios y comisiones** | **11%** (persona jurídica) / **10–11%** (persona natural) | $1 |
| **ReteIVA** (sobre el IVA facturado en servicios) | **15% del IVA** | 2 UVT |

**Efectos:**
- **No es un costo adicional**: es un **anticipo del impuesto de renta** que se nos retiene; se descuenta al declarar. Pero **afecta el flujo de caja** y la **negociación**.
- ⚠️ **La retención se calcula según el concepto correcto de la factura.** Si facturamos como "servicios generales" (4%) en vez de "licenciamiento de software" (3,5%) o viceversa, quedamos con saldos a favor/favor en contra. **Definir el concepto con el contador.**

### 12.3 Facturación electrónica

- **Obligatoria** para personas jurídicas, para **RST**, y para personas naturales con ingresos **> 3.500 UVT** (o con varias sedes, que exploten intangibles, usuarios aduaneros, etc.).
- Requiere **habilitación DIAN** + **firma electrónica**. La DIAN ofrece **facturación gratuita** y certificado gratis.
- **El club casi siempre pedirá factura** para deducir costos → sin factura electrónica, la venta se dificulta o se informaliza.

### 12.4 Régimen y forma jurídica

| Opción | Pro | Contra |
|---|---|---|
| **Persona natural (no responsable de IVA)** | Simple; sin IVA en factura (al inicio) | Tope 3.500 UVT; no escala; el club no descuenta IVA; poca credibilidad institucional |
| **Persona natural en RST** | Carga consolidada; factura electrónica | IVA aparte; topes |
| **SAS** | Credibilidad con federaciones/clubes; limita responsabilidad; puede exportar y descontar IVA | Costos de constitución y contabilidad |

**Recomendación provisional:** al empezar la venta a terceros, **formalizar** (RST o SAS) — un club o federación que firma un contrato grande va a querer factura electrónica y un NIT.

### 12.5 El operador: contrato de prestación de servicios vs. contrato realidad

- Si el operador es **independiente** (autonomía, puede delegar, sin horario impuesto, sin subordinación) → **contrato de prestación de servicios** válido. Se le practica **retención (4% servicios, o 10–11% honorarios)** y él asume su seguridad social.
- ⚠️ **Riesgo de "contrato realidad":** si en la práctica hay **subordinación**, **horario fijo** y **prestación personal indelegable**, un juez puede declarar **contrato de trabajo** → **prestaciones sociales, seguridad social y aportes retroactivos**. La denominación del contrato **no** importa; importa la realidad.
- **Implicación en el precio:** el **2M del operador como contratista** no incluye prestaciones. Si se vuelve empleado, el costo real sube ~**40–50%** (cesantías, prima, vacaciones, salud, pensión, ARL, parafiscales). **El 2M debe ser un honorario de contratista con autonomía real** — o hay que presupuestar la carga prestacional.
- **Recomendación contractual:** contrato de **servicios de operación** por evento (no continuo), con derecho de sustitución, sin horario, con entregables y un **contrato de referidos** separado (§12.7).

### 12.6 Protección de datos personales (Ley 1581 de 2012 y decretos)

- **Autorización previa, expresa e informada** del titular para tratar sus datos.
- **Menores de edad (art. 7 Ley 1581 + Decreto 1074 art. 2.2.2.25.2.9):** el tratamiento **está PROHIBIDO** salvo **datos de naturaleza pública** y siempre que respete el interés superior del menor; se requiere **autorización del representante legal**, previo **derecho del menor a ser escuchado**. Y **los datos de menores deben protegerse de manera prevalente**.
  - **BTM PRO almacena fecha de nacimiento y datos de jugadores Sub-7 a Sub-19** → esto es **material**. Hay que definir quién es el **responsable** (probablemente el club/organizador, no BTM PRO) y usar **contrato de encargado del tratamiento**.
  - **Riesgo alto:** la **página pública de resultados** con nombres de menores. Mitigación: pedir autorización, o publicar solo **iniciales/apellido**, o aceptar que el organizador asume el rol de responsable con su propia política.
- **Registro Nacional de Bases de Datos (RNBD):** obligatorio solo para **sociedades y ESAL con activos > 100.000 UVT** y entidades públicas. Una startup pequeña **no** está obligada — pero **sí** debe tener **política de tratamiento** y **autorizaciones**.
- **Recomendación:** (a) **política de tratamiento** propia; (b) **contrato de encargado** con cada club (el club es responsable, nosotros encargados); (c) **cláusula en el EULA** sobre datos; (d) cuidado con la página pública y menores.

### 12.7 Propiedad intelectual

- El software está protegido por **derecho de autor** desde su creación (Ley 23 de 1982, Ley 44 de 1993, **Decreto 1360 de 1989**).
- **Registro ante la DNDA es gratuito, declarativo y no obligatorio**, pero es **medio de prueba idóneo** de la titularidad → **registrar BTM PRO** (adjuntar código fuente o descripción + manuales; ~15 días hábiles).
- **Registrar la marca "BTM PRO"** ante la **SIC** (Superintendencia de Industria y Comercio).
- El **EULA** existente es válido y coherente; reforzarlo con: concepto de licencia, límite de instalaciones, prohibición de uso competitivo, ley aplicable y **tratamiento de datos**.

### 12.8 Contratos que hacen falta (checklist)

| Documento | Para qué | Prioridad |
|---|---|---|
| **Contrato marco de licencia** (además del EULA click-wrap) | Club/federación; número de instalaciones, período, soporte | Alta |
| **Orden de servicio por evento** (anexo) | Alcance, fecha, canchas, operador, viáticos, precio | Alta |
| **Contrato de prestación de servicios del operador** | Evitar contrato realidad | Alta |
| **Contrato de referidos / comisión** | Base, %, pago contra cobro, conflicto de interés | Alta |
| **Contrato de encargado del tratamiento** (datos) | Cumplir Ley 1581 | Alta |
| **Acta de comodato** (router + computador) | Propiedad y responsabilidad por daño/pérdida | Media |
| **Política de tratamiento de datos** | Ley 1581 | Media |
| **Contrato de confidencialidad (NDA)** con socios/operadores | Código fuente y clientes | Media |

### 12.9 Impacto neto en el precio final (ejemplo)

Evento gestionado 1 día, 3 canchas: Servicio 1.000.000 + Operador 2.000.000 + Licencia 300.000 = **3.300.000**.

| Concepto | Valor | Nota |
|---|---|---|
| Subtotal | 3.300.000 | |
| **IVA 19%** | **+627.000** | Si somos responsables de IVA → 3.927.000 |
| Retención renta (3,5–4%) | −115.500 a −132.000 | La retiene el club; es anticipo, no costo |
| ReteIVA (15% del IVA) | −94.050 | La retiene el club |
| **Recibe el cliente (valor factura)** | **3.927.000** | |
| **Flujo de caja inmediato** | ~3.700.000 | Neto de retenciones |
| **Si el operador es contratista (2M)** | ~2.000.000 pasa al operador | Sin prestaciones |
| **Si el operador fuera empleado** | ~2.800.000–3.000.000 | Por la carga prestacional |

**Conclusión tributaria:** el **IVA 19%** es el factor que más mueve el precio final y la **disposición a pagar** del club. La retención no es costo pero **hay que manejarla en el flujo**. La forma jurídica (informal vs RST vs SAS) cambia radicalmente la factura que el club acepta.

---

## 13. Riesgos y críticas honestas

| Riesgo | Severidad | Mitigación |
|---|---|---|
| No hay draws/brackets propios | Alta | No prometer "gestión completa"; importar o construir |
| Sin firma de código | Alta | Firmar instaladores |
| Sin licenciamiento | Alta | Activación offline firmada (§10) |
| **IVA 19% encarece la venta** | Alta | Cotizar "más IVA"; evaluar régimen |
| **Datos de menores** | Alta | Autorización del representante legal; contrato de encargado; cuidar la página pública |
| **Contrato realidad del operador** | Alta | Contrato de servicios con autonomía real |
| Mercado pequeño → precios bajos | Alta | Volumen por canal |
| Punto único de falla (un operador) | Alta | Respaldo + soporte remoto |
| Comisión mal diseñada | Media | Reglas §8.4 + contrato |
| "Ilimitado" canibaliza per-torneo | Media | Definir límites |
| i18n incompleto | Media | Traducir para ligas |
| Scope creep en "gestión total" | Media | §7.6 / §7.7 |
| **Retención mal conceptuada** | Media | Definir concepto de factura con contador |
| **Normativa tributaria volátil** | Media | Revisar tablas cada trimestre |

---

## 14. Banco de ideas (evaluable)

### 14.1 Monetización

| ID | Idea | Eje | Esf. | Impacto | Depende de | Evidencia / riesgo |
|---|---|---|---|---|---|---|
| M1 | Plan anual **"torneos ilimitados"** | A | S | ★★★ | Licenciamiento | Mejor encaje offline; canibaliza per-torneo |
| M2 | **Licencia per-torneo** | A | M | ★★ | Licenciamiento | Más roce el día del partido |
| M3 | **Free tier** club base | A | S | ★★ | Licenciamiento | Adopción vs. canibalización |
| M4 | **Enterprise / federación** white-label | A | M | ★★★ | Firma + licenciamiento | Venta larga |
| M5 | **Servicio BTM PRO** | B | M | ★★★ | Contrato, logística | Escala con certificación |
| M6 | **Operador presencial** (+2M) | B | S | ★★★ | Red de operadores | Punto único de falla |
| M7 | **Operador remoto** | B | M | ★★ | Red en venue | Menor costo/control |
| M8 | **Certificación de operadores** | B | L | ★★★ | Currículo, red | Palanca de escala |
| M9 | **Retainer anual** para ligas | B | S | ★★ | Relación institucional | Recurrencia |
| M10 | **Programa de enganche** (§8) | B | S | ★★★ | Contrato de referidos | Canal barato; conflicto |
| M11 | **Muro de patrocinadores** | C | M | ★★★ | Ya implementado | Inventario publicitario |
| M12 | **Resultados públicos** | C | M | ★★ | Public viewer | Recurrencia; menores (§12.6) |
| M13 | **Streaming** | C | M | ★★ | WebRTC, internet | Infra; viewers limitados |
| M14 | **Appliance** ("Box") | C | M | ★★ | Hardware, logística | Margen + foso |
| M15 | **SLA / soporte prioritario** | C | S | ★★ | Proceso de soporte | Bajo costo |
| M16 | **Bundle con app_bdm + web** | C | M | ★★ | Integración | Upsell |
| M17 | **Datos/estadísticas** | C | L | ★★ | Producto, privacidad | Alto esfuerzo |

### 14.2 Control de copia

`P1` EULA · `P2` Activación offline firmada · `P3` Node-locking · `P4` Códigos por torneo · `P5` Detección de reloj · `P6` Degradación elegante · `P7` Appliance · `P8` Telemetría opt-in.

### 14.3 Distribución

`D1` Venta directa · `D2` Federaciones/ligas · `D3` Clubes/academias · `D4` Revendedores · `D5` Red de operadores · `D6` Firma de código · `D7` Contenido/embajadores · `D8` Programa de referidos.

---

## 15. Evaluación: criterios y matriz de puntajes

### 15.1 Criterios (1 = malo · 5 = bueno)

| Sigla | Criterio | 5 significa |
|---|---|---|
| **DP** | Disposición a pagar | El cliente ya gasta en esto |
| **E** | Esfuerzo | Bajo esfuerzo |
| **Esc** | Escalabilidad | No depende de nómina lineal |
| **Df** | Defensibilidad | No copiable |
| **EO** | Encaje offline | No rompe la promesa LAN |
| **RR** | Riesgo reputacional | Bajo riesgo |
| **TTC** | Time-to-cash | Cobra rápido |
| **Dep** | Dependencias | Sin bloqueantes |

> Puntajes **borrador (v0.4)**, a discutir con el equipo. Escala simple (suma), sin ponderar; el peso lo pone el equipo en la evaluación.

### 15.2 Matriz — Monetización

| ID | DP | E | Esc | Df | EO | RR | TTC | Dep | **Total** | Recomendación |
|---|---|---|---|---|---|---|---|---|---|---|
| M1 | 4 | 3 | 5 | 2 | 5 | 5 | 3 | 2 | **29** | Ahora (con licenciamiento) |
| M2 | 4 | 3 | 4 | 2 | 2 | 3 | 3 | 2 | **23** | Después |
| M3 | 1 | 4 | 5 | 1 | 5 | 5 | 1 | 2 | **24** | Ahora (adopción) |
| M4 | 5 | 2 | 4 | 3 | 4 | 4 | 2 | 1 | **25** | Después (prerequisitos) |
| M5 | 4 | 3 | 3 | 5 | 5 | 4 | 4 | 3 | **31** | **Ahora** |
| M6 | 4 | 4 | 1 | 5 | 5 | 3 | 5 | 3 | **30** | **Ahora** |
| M7 | 3 | 3 | 3 | 4 | 3 | 3 | 3 | 2 | **24** | Después |
| M8 | 3 | 2 | 5 | 5 | 5 | 4 | 2 | 2 | **28** | Después (palanca) |
| M9 | 4 | 4 | 4 | 4 | 5 | 4 | 3 | 2 | **30** | Ahora |
| M10 | 3 | 5 | 5 | 3 | 5 | 3 | 5 | 4 | **33** | **Ahora** |
| M11 | 3 | 3 | 4 | 5 | 5 | 4 | 3 | 4 | **31** | **Ahora** |
| M12 | 3 | 3 | 4 | 4 | 3 | 4 | 3 | 3 | **27** | Después |
| M13 | 3 | 2 | 3 | 3 | 2 | 3 | 3 | 2 | **21** | Condicional |
| M14 | 4 | 2 | 3 | 5 | 5 | 3 | 2 | 2 | **26** | Después |
| M15 | 4 | 4 | 4 | 4 | 5 | 4 | 3 | 3 | **31** | **Ahora** |
| M16 | 3 | 2 | 4 | 4 | 4 | 3 | 2 | 1 | **23** | Después |
| M17 | 2 | 1 | 4 | 3 | 3 | 2 | 1 | 1 | **17** | Descartar (por ahora) |

### 15.3 Matriz — Control de copia

Criterios: **Ef** efectividad · **E** esfuerzo (5 = bajo) · **Co** costo (5 = bajo) · **RR** riesgo (5 = bajo).

| ID | Ef | E | Co | RR | **Total** | Recomendación |
|---|---|---|---|---|---|---|
| P1 EULA | 4 | 5 | 5 | 5 | **19** | **Ya hecho** |
| P2 Offline firmada | 5 | 2 | 3 | 5 | **15** | Hacer (con licenciamiento) |
| P3 Node-locking | 3 | 4 | 4 | 4 | **15** | Hacer |
| P4 Códigos torneo | 3 | 2 | 3 | 4 | **12** | Solo si per-torneo |
| P5 Reloj | 2 | 4 | 4 | 4 | **14** | Opcional |
| P6 Degradación elegante | 2 | 4 | 4 | 5 | **15** | **Hacer (crítico reputacional)** |
| P7 Appliance | 4 | 2 | 2 | 4 | **12** | Ver M14 |
| P8 Telemetría | 2 | 4 | 4 | 3 | **13** | Opcional |

### 15.4 Matriz — Distribución

Criterios: **DP** disposición · **E** esfuerzo (5 = bajo) · **Esc** escalabilidad · **TTC** time-to-cash · **Dep** dependencias.

| ID | DP | E | Esc | TTC | Dep | **Total** | Recomendación |
|---|---|---|---|---|---|---|---|
| D1 Venta directa | 5 | 5 | 2 | 5 | 4 | **21** | **Ahora** |
| D2 Federaciones/ligas | 5 | 3 | 5 | 2 | 3 | **18** | Después (alto impacto) |
| D3 Clubes/academias | 4 | 5 | 3 | 4 | 4 | **20** | **Ahora** |
| D4 Revendedores | 3 | 3 | 5 | 3 | 3 | **17** | Después |
| D5 Red de operadores | 4 | 2 | 5 | 2 | 3 | **16** | Después |
| D6 Firma de código | 3 | 3 | 5 | 4 | 1 | **16** | **Prerequisito** |
| D7 Contenido/embajadores | 3 | 5 | 4 | 5 | 4 | **21** | **Ahora** |
| D8 Programa referidos | 3 | 5 | 5 | 5 | 4 | **22** | **Ahora** |

### 15.5 Lectura de la matriz

- **Ganancia rápida (hacer ya):** M5 (servicio), M6 (operador), M10/D8 (enganche), M11 (patrocinadores), D1/D3/D7 (venta directa y contenido).
- **Palancas de escala (mediano plazo):** M8 (certificación de operadores), D2 (federaciones/ligas), M1 (anual ilimitado), M9 (retainer).
- **Prerequisitos bloqueantes:** D6 (firma de código), M1/M2/M4 (licenciamiento), P2/P6.
- **Alto esfuerzo / baja certeza:** M17 (datos), M13 (streaming a escala).
- **Trampas:** M2 sin código anticipado; M11 sin modelo de pauta; comisión sumada al cliente.
- **Lo que la matriz NO captura:** el **IVA 19%** y la **ley de datos** encarecen/limitan M1–M4 y M12. Van como **riesgo transversal**, no como puntaje.

---

## 16. Supuestos

- **FX:** ~4.000 COP/USD. **UVT 2026:** $52.374.
- **Mercado:** clubes y ligas de bádminton en Colombia/LatAm, sensibles al precio.
- **Cliente típico:** sin equipo técnico para montar LAN/TVs.
- **Hardware:** el club tiene TVs; los dispositivos de cast son del club o add-on.
- **Internet:** puede no existir en el venue.
- **Operador:** COP 2.000.000/evento (≤2 días); contratista independiente.
- **Comisión:** sale del margen; no se suma al cliente.
- **Tributario:** el equipo aún **no está formalizado** (persona natural / sin IVA). Cambiará al vender.
- **Precios:** hipótesis, no cotizaciones. Todos **"más IVA"** si aplica.

---

## 17. Decisiones abiertas

- [ ] ¿Se fijan los planes de software (§6) o se validan primero con clubes?
- [ ] ¿Se adopta la rejilla §7.4 (Servicio + Operador separados)?
- [ ] ¿El operador escala por día (+1M) o se mantiene plano en 2M?
- [ ] ¿Existe el **tier de autoservicio sin operador** (§7.9)?
- [ ] ¿Viáticos como línea fija o "el club cubre"?
- [ ] **Comisión:** ¿8–10% / 12–18% / 3–5%? ¿sobre qué base? ¿dinero o crédito?
- [ ] ¿El operador puede cobrar comisión si además consigue el cliente?
- [ ] **Régimen tributario:** ¿persona natural, RST o SAS? ¿Cuándo?
- [ ] **Concepto de factura:** ¿"licenciamiento de software" (3,5%) o "servicios generales" (4%)?
- [ ] **Datos de menores:** ¿quién es responsable? ¿se publican nombres en resultados públicos?
- [ ] ¿Operador como contratista (autonomía) o empleado (prestaciones)?
- [ ] ¿Se registra el software en la DNDA y la marca en la SIC?
- [ ] ¿Quién firma los instaladores y con qué cuenta?
- [ ] ¿Licenciamiento propio (Ed25519) o proveedor (Keygen/Cryptlex/LicenseSpring)?
- [ ] ¿Operador de respaldo sí/no?
- [ ] ¿Quién vende la pauta de patrocinadores (nosotros o el club)?

---

## 18. Glosario

| Término | Definición |
|---|---|
| **Eje A / B / C** | Licencia de software / servicio BTM PRO / add-ons y pauta |
| **Servicio BTM PRO** | Setup + equipo + coordinación + soporte remoto (nuestro margen) |
| **Operador** | Persona en sitio; **2M** que se **suman** |
| **Autoservicio** | El club opera solo; sin operador |
| **Enganche / embajador** | Quien recomienda y acerca la venta, con comisión |
| **Node-locking** | Licencia atada a una máquina |
| **Degradación elegante** | La app pierde funciones, nunca se bloquea |
| **IVA** | Impuesto sobre las ventas (19% general) |
| **Retención en la fuente** | Anticipo de renta que el pagador descuenta |
| **ReteIVA** | Retención del 15% sobre el IVA facturado en servicios |
| **RST / SIMPLE** | Régimen Simple: consolida renta + ICA |
| **ICA** | Impuesto de industria y comercio (municipal) |
| **UVT** | Unidad de Valor Tributario ($52.374 en 2026) |
| **DNDA** | Dirección Nacional de Derecho de Autor (registro de software) |
| **SIC** | Superintendencia de Industria y Comercio (marcas, datos) |
| **RNBD** | Registro Nacional de Bases de Datos |
| **Contrato realidad** | Relación laboral encubierta bajo prestación de servicios |
| **CAC** | Costo de adquisición de cliente (aquí, la comisión) |

---

## 19. Fuentes

**Mercado:** Tournament Software · Brakto · Score7 · WeighBook · ProTSR · ChronoTrack · Seiko Sports Technologies.

**Licenciamiento offline:** Keygen (keygen.sh y docs de offline licensing) · Cryptlex / Licentric / LicenseSpring · ADR "offline licenses" de SurfSense · "How offline license activation actually works" (dev.to).

**Tributario Colombia:**
- IVA: art. 468 ET (19% general) · art. 476 ET (servicios excluidos: hosting/cloud num. 21-22; licencias para contenido digital num. 20 — no aplicable) · art. 481 ET (exportación de servicios, 0%).
- Retención en la fuente 2026: tabla con UVT $52.374 · licenciamiento de software **3,5%** · servicios informáticos **3,5%** · servicios generales **4%/6%** (base 2 UVT desde 1-jul-2026) · honorarios y comisiones **11%/10%** · ReteIVA **15% del IVA**. Decreto 572 de 2025 (suspendido y reactivado en 2026).
- RST: DIAN (dian.gov.co/impuestos/Regimen-Simple-de-Tributacion-RST) · tarifas 1,8%–14,5% · Grupo 4 servicios profesionales 4,9%–14,5%.
- Facturación electrónica: art. 615 ET · DIAN "Ser facturador electrónico".
- ICA: municipal; verificar acuerdo vigente de **Chía/Cundinamarca**.

**Propiedad intelectual:** Ley 23 de 1982 · Ley 44 de 1993 · **Decreto 1360 de 1989** · DNDA (registro de software, gratuito y declarativo) · marca ante la SIC.

**Protección de datos:** **Ley 1581 de 2012** · Decreto 1377 de 2013 · **Decreto 1074 de 2015 art. 2.2.2.25.2.9** (menores) · RNBD (SIC, obligatorio > 100.000 UVT de activos).

**Laboral:** CST art. 23 · Corte Constitucional C-154/1997 · T-366/2023 · Consejo de Estado (contrato realidad).

---

## 20. Bitácora de cambios

| Versión | Fecha | Cambios |
|---|---|---|
| 0.1 | 2026-10-15 | Versión inicial: posicionamiento, mercado, planes, servicio, licenciamiento, distribución. |
| 0.2 | 2026-10-15 | Documento de evaluación: contexto técnico, crítica económica, banco de ideas, marco de evaluación, supuestos, glosario, fuentes. |
| 0.3 | 2026-10-15 | Se separa **Servicio BTM PRO** del **Operador** (2M que se suman). Se añade el **programa de enganche** (§8). Se recomponen rejilla y ejemplos; se agrega el tier de autoservicio. |
| 0.4 | 2026-10-15 | **Nueva §12: aspectos tributarios y contractuales (Colombia)** — IVA, retención, RST, ICA, facturación, DNDA, Ley 1581 (menores), contrato realidad, checklist contractual e impacto en el precio final. **Nueva §15: matriz de puntajes** de las 33 ideas. |
