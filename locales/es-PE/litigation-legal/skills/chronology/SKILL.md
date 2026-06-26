---
name: chronology
description: Construye o actualiza una cronología a partir de fuentes documentales declaradas y archivos subidos — eventos fechados extraídos, desduplicados y etiquetados por relevancia según la teoría del caso. Úsalo cuando el usuario pida construir una cronología o línea de tiempo desde un expediente o carpeta del caso, diga "cron del expediente" o "qué pasó cuándo", o necesite una cronología de trabajo, de hechos, o específica por testigo.
argument-hint: "[slug] [--format=working|sof|witness-[nombre]]"
---

# /chronology

1. Cargar `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` → teoría del caso, hecho pivote, hechos clave.
2. Cargar `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` → fuentes de almacenamiento documental, patrón de carpeta del caso por defecto.
3. Seguir el flujo de trabajo y referencia abajo.
4. Identificar fuentes en orden: rutas proporcionadas por el usuario en esta sesión, carpeta del caso por defecto, fuentes declaradas en `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md`.
5. Para fuentes legibles: extraer eventos fechados. Para fuentes inaccesibles: anotar en Vacíos.
6. Desduplicar, consolidar con lista de fuentes por evento.
7. Etiquetar relevancia (🔴/🟡/⚪) según teoría del caso.
8. Escribir `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/chronology.md` (o variante de formato según flag).
9. Si existe versión previa: incrementar número de versión, presentar resumen de diferencias al usuario.
10. Confirmar antes de finalizar: "Esto es lo que construí. Revisá las entradas 🔴 — ¿hay algo que haya calificado mal?"

---

# Cronología

## Restricciones de uso de documentos obtenidos por exhibición

Antes de trabajar con un conjunto de documentos litigiosos, preguntá: "¿Alguno de estos documentos fue obtenido mediante exhibición o discovery en un proceso legal?" Si la respuesta es sí:

<!-- ADAPT-PE: El discovery anglosajón (disclosure/discovery) no existe en el proceso civil peruano. En Perú, la obtención de documentos de la contraparte opera mediante "exhibición de documentos" (Art. 257-261 CPEP 2025 o Arts. 241-245 CPC) a pedido de parte y con autorización judicial, o mediante "informes" (Art. 262 CPEP 2025). El alcance es sustancialmente más limitado que el discovery anglosajón. Se requiere mapear estos conceptos al régimen probatorio peruano. -->

- **Inglaterra y Gales (CPR 31.22):** Los documentos obtenidos mediante disclosure están sujetos a la obligación implícita (implied undertaking) — solo pueden usarse para la finalidad del proceso en que fueron exhibidos, salvo que el tribunal otorgue permiso, la parte que los exhibió consienta, o el documento haya sido leído en audiencia pública. Usarlos para otro caso, otra pretensión o un fin comercial sin permiso constituye desacato. <!-- ADAPT-PE: La regla CPR 31.22 es una norma procesal civil inglesa sin equivalente directo en Perú. En el proceso civil peruano, los documentos obtenidos mediante exhibición están sujetos al principio de finalidad probatoria del proceso donde se actuaron. El "desacato" (contempt of court) anglosajón no tiene equivalente exacto en el sistema peruano, donde las sanciones por uso indebido se canalizan por vía de responsabilidad civil o disciplinaria. Requiere análisis con un procesalista peruano para determinar el régimen de restricciones aplicable. -->
- **EE.UU.:** Las órdenes de protección (protective orders) y la Regla 26(c) pueden imponer restricciones similares. Revisar la orden aplicable. <!-- ADAPT-PE: Las "protective orders" y la Regla 26(c) de las Federal Rules of Civil Procedure son instituciones del discovery estadounidense sin equivalente directo en Perú. En el proceso civil peruano, las limitaciones al uso de prueba obtenida en un proceso provienen de la garantía constitucional del debido proceso y del principio de licitud probatoria (Art. IX del Título Preliminar del CPC, Art. 159 CPEP 2025). Se requiere definir qué mecanismo peruano cumpliría la función de una protective order. -->
- **Otras jurisdicciones:** Restricciones similares suelen aplicar. Revisar la norma local.

Confirmar: "Este uso está dentro del proceso en que los documentos fueron exhibidos, o tengo permiso / consentimiento, o los documentos ya son públicos." Si no se confirma, señalarlo: "⚠️ Los documentos exhibidos pueden tener restricciones de uso. Confirmá que este uso está permitido antes de continuar."

## Propósito

Los hechos ocurren en orden. La cronología es la columna vertebral de la que cuelga cada narrativa — el escrito de hechos en un alegato, los memorandos de reserva, los informes de conciliación, la preparación de testigos. <!-- ADAPT-PE: Las "depositions" (deposiciones) no existen en el proceso civil peruano. En Perú, la prueba testimonial se actúa en audiencia oral ante el juez (Arts. 222-233 CPC, Arts. 193-210 CPEP 2025). La preparación de testigos (witness prep en la práctica anglosajona) no está regulada de la misma forma; se requeriría adaptar esta sección a la práctica forense peruana donde la preparación de testigos es una actividad permitida pero con restricciones éticas (Código de Ética del Abogado). --> Construir una cronología a mano es lento; la IA es buena en extracción estructurada. La trampa: si entra basura, sale basura. Esta skill extrae de las fuentes que la configuración declara y de lo que el usuario suba.

## Modos

Esta skill sirve a dos contextos de práctica. Elegí un modo por defecto según el `## Rol` del usuario en el CLAUDE.md de configuración del plugin; el usuario puede sobrescribirlo por ejecución con un flag.

- **Modo `--matter` (por defecto para abogado interno de empresa).** Enfocado en la historia del caso. Lee la teoría del caso y los hechos clave de `matter.md`, extrae de las fuentes de almacenamiento documental declaradas (Google Drive, SharePoint, Gmail, iManage, CLM — lo que declare la sección `## Panorama` de CLAUDE.md), y trata `history.md` como el registro interno corriente (decisiones, holds, memorandos de reserva — intencionalmente fuera de la cronología). La salida es centrada en el caso: qué pasó a lo largo de la controversia, etiquetado para uso en la estrategia procesal. <!-- ADAPT-PE: iManage y CLM son plataformas de gestión documental legal usadas principalmente en firmas y departamentos legales anglosajones. En Perú, los equivalentes funcionales serían sistemas de gestión documental como SID (Sistema de Información Documental), gestores documentales corporativos, o simplemente carpetas compartidas en red. La referencia debería adaptarse a las plataformas efectivamente usadas en el entorno legal peruano. -->
- **Modo `--documents` (por defecto para asociado de estudio / asistente legal).** Enfocado en el expediente documental. Lee la teoría del caso de la configuración, luego extrae de una exportación de eDiscovery, un conjunto de archivos por custodio o una producción numerada Bates. <!-- ADAPT-PE: El sistema de numeración Bates es un estándar de identificación documental del discovery anglosajón sin equivalente en Perú. En el proceso peruano, los documentos se identifican por su ubicación en el expediente judicial (fojas, cuaderno, tomo) o por denominación de la parte que los presenta. "eDiscovery" como proceso y plataformas (Everlaw, Relativity, DISCO, Aurora) son ajenos a la práctica peruana. Se requiere adaptar a sistemas de gestión de expedientes electrónicos como el EJE (Expediente Judicial Electrónico) o la Mesa de Partes Electrónica del Poder Judicial peruano. --> La salida es centrada en la producción documental: qué muestran los documentos, con citas de identificación, etiquetados según la teoría del caso.

Ambos modos convergen en la misma estructura de salida (línea de tiempo, etiquetas de relevancia 🔴/🟡/⚪, vacíos, variante de hechos). La diferencia está en el perfil de fuentes y el marco de relevancia.

Si `## Rol` es `solo` u `other`, usar `--matter` por defecto pero mencionar ambos modos en la primera ejecución y dejar que el usuario elija.

## Encuadre según posición procesal (etiquetas de relevancia)

El mismo evento es relevante de manera distinta según si el abogado está probando una pretensión o desvirtuándola. Leer `## Posición` en el perfil de práctica (y la postura por caso si el caso sobrescribe el valor por defecto):

- **Demandante (encuadre ofensivo)** — 🔴 marca eventos que *establecen* elementos de la pretensión (responsabilidad, nexo causal, daños, notificación), *cierran* vacíos que la defensa intentará abrir, o *inician* plazos de prescripción a favor del demandante. <!-- ADAPT-PE: La doctrina de "statute of limitations" anglosajona tiene su equivalente funcional en la "prescripción extintiva" peruana (Art. 1989-2007 CC). Sin embargo, las reglas de cómputo, suspensión e interrupción difieren. "Notice" (notificación como elemento de la pretensión) en el common law tiene un alcance distinto al régimen de notificaciones peruano. Se requiere validación con un civilista peruano. --> 🟡 marca eventos que respaldan la pretensión pero son susceptibles de impugnación. ⚪ es contexto de fondo.
- **Demandado (encuadre defensivo)** — 🔴 marca eventos que *rompen* elementos de la pretensión (falta de nexo causal, falta de notificación, falta de legitimación), *abren* defensas de prescripción o incompetencia, o *respaldan* defensas afirmativas (renuncia, asunción de riesgo, culpa comparativa). <!-- ADAPT-PE: Las "affirmative defenses" del common law (release, waiver, assumption of risk, comparative fault) tienen equivalentes parciales en el derecho civil peruano: la renuncia de derechos (Art. V del Título Preliminar del CC), la asunción de riesgo (no regulada explícitamente como defensa autónoma pero reconocida en doctrina de responsabilidad civil), y la culpa comparativa / concausa (Art. 1973 CC — "concausa" en responsabilidad extracontractual). Se requiere mapeo preciso con un civilista peruano. --> 🟡 marca eventos que debilitan la narrativa del demandante. ⚪ es contexto.
- **Ambos / varía** — preguntar al usuario para cada cronología qué encuadre aplicar para las etiquetas de relevancia. La línea de tiempo subyacente es neutral; solo cambia la lectura de relevancia.

Anotar el encuadre aplicado al inicio de la salida: `Etiquetas de relevancia aplicadas desde la perspectiva [demandante / demandado].` Al producir una variante de Relación de Hechos, usar la posición por defecto salvo que el usuario indique lo contrario. <!-- ADAPT-PE: "Statement of Facts" (SoF) es una sección del brief anglosajón que no tiene equivalente estructural exacto en los escritos judiciales peruanos. En Perú, los hechos se exponen en los "fundamentos de hecho" de la demanda (Art. 424 CPC, Art. 108 CPEP 2025) o en los "antecedentes" de ciertos recursos. La función es similar pero el formato y las convenciones difieren. Se requiere adaptar la salida a los formatos procesales peruanos. -->

## Cargar contexto

Común:
- CLAUDE.md de configuración del plugin → contexto de teoría del caso (abogado interno: `## Panorama` para fuentes documentales; asociado de estudio: `## Teoría del caso` y `## Revisión documental` para plataforma + custodios), `## Salidas` para el encabezado del producto de trabajo, `## Postura de decisión` para la regla de señalización de privilegio. <!-- ADAPT-PE: "Work-product header" y "privilege-flagging rule" son conceptos de la doctrina de privilegios anglosajona. En Perú, el secreto profesional del abogado (Art. 30 de la Ley 30215 y Código de Ética del Abogado Peruano) y la reserva de la defensa técnica tienen un alcance distinto al attorney-client privilege y work-product doctrine. Se requiere definir qué protección corresponde a los documentos de trabajo del abogado bajo el régimen peruano. -->
- `chronology.md` previo para este caso, si existe.
- Cualquier archivo que el usuario suba o ruta que proporcione en sesión.

Modo `--matter` también lee:
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` → teoría del caso, hechos clave, hecho pivote (para etiquetado de relevancia), fechas clave.
- Patrón de carpeta del caso por defecto de CLAUDE.md → dónde están los documentos de este slug.

Modo `--documents` también lee:
- Metadatos de plataforma eDiscovery si hay un conector disponible (Everlaw, Relativity, DISCO, Aurora) — por custodio + rango de fechas. <!-- ADAPT-PE: Ninguna de estas plataformas (Everlaw, Relativity, DISCO, Aurora) opera en el mercado legal peruano. Se requiere identificar los equivalentes funcionales peruanos: el Expediente Judicial Electrónico (EJE), la Mesa de Partes Electrónica (MPE), el Sistema de Notificaciones Electrónicas (SINOE), y sistemas de gestión documental internos de estudios jurídicos peruanos. -->
- Manifiesto de producción o índice de documentos si el usuario lo señala.

**Barrera de conflictos — ineludible (modo `--matter`).** Antes de construir la cronología, verificar `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml` para el slug del caso. Si el caso no está en `_log.yaml`, rechazar y redirigir:

> "No encuentro [slug del caso] en el registro de casos. Ejecutá `/litigation-legal:matter-intake` primero para que corra la verificación de conflictos y se configure el espacio de trabajo del caso. No voy a construir una cronología sobre un caso que no ha pasado por intake — la verificación de conflictos es la barrera de entrada."

No continuar sobre un caso sin intake. El intake es lo que ejecuta la verificación de conflictos y escribe la fila en `_log.yaml` que esta skill lee. El modo `--documents` (ejecutándose contra un conjunto documental ad hoc sin slug de caso) está exento de esta barrera, pero sus salidas deben tratarse como investigación pre-caso y no archivarse como producto de trabajo del caso.

## Flujo de trabajo

### Paso 0: Barrera de privilegio (se ejecuta primero, siempre)

El trabajo de cronología extrae de documentos. Los documentos suelen estar amparados por privilegio (abogado-cliente, producto de trabajo, interés común, defensa conjunta) — los archivos de casos de abogados internos suelen estarlo por defecto; las producciones de eDiscovery, especialmente producciones continuas o de interés común, suelen contener material privilegiado o no revisado. Extraer contenido de un documento privilegiado a una cronología que luego se comparte puede *arriesgar* la renuncia al privilegio, según quién la reciba y bajo qué doctrina (las protecciones de interés común, defensa conjunta, Kovel y producto de trabajo pueden aplicar). <!-- ADAPT-PE: Todo este párrafo se basa en doctrinas de privilegio anglosajonas sin equivalente directo en Perú: "attorney-client privilege" (privilegio abogado-cliente), "work-product doctrine" (doctrina de producto de trabajo), "common interest" (interés común), "joint defense" (defensa conjunta), "Kovel" (extensión del privilegio a terceros asesores). En Perú, la protección de las comunicaciones abogado-cliente se fundamenta en el secreto profesional (Art. 30 Ley 30215, Código de Ética del Abogado Peruano, Art. 165 CPP), que tiene un alcance distinto. La doctrina de producto de trabajo no está reconocida como tal. La renuncia (waiver) al privilegio es un concepto del common law sin desarrollo equivalente en la dogmática peruana. Se requiere un análisis completo con un especialista en ética profesional y derecho probatorio peruano para determinar qué protecciones existen y cómo opera su posible pérdida. --> El análisis de renuncia al privilegio depende de los hechos — obtener visto bueno del abogado a cargo antes de distribuir.

La skill no extraerá hasta que el usuario elija una postura de privilegio:

> Antes de extraer: ¿cómo se ha revisado el privilegio de las fuentes?
>
> - **A. Todas las fuentes están revisadas** — ya hiciste la revisión de privilegio. Extraigo sin señalizaciones de privilegio. La salida está en postura lista para discovery; igualmente marcada como producto de trabajo.
>
> - **B. Mixto o aún no revisado** — Extraigo y etiqueto cada entrada con un flag `priv`: `ok` (proviene de material claramente no privilegiado), `flag` (proviene de material potencialmente privilegiado — abogado-cliente, producto de trabajo, interés común), o `review` (origen incierto). Las entradas señalizadas se marcan visualmente en la salida, y la variante de Relación de Hechos las filtra por defecto.
>
> - **C. Abortar — revisar primero** — pausar la skill. Revisar las fuentes. Volver y re-ejecutar.

Registrar la elección en el encabezado de la cronología como `postura_privilegio: A-revisado | B-mixto | C-abortado`. Si es B o C, registrar brevemente la justificación.

**Por qué una barrera y no solo una advertencia:** una advertencia se lee una vez y se olvida. Una barrera fuerza que la decisión sobre la postura quede registrada, lo que significa que cada archivo de cronología lleva su propia trazabilidad — cualquiera que lo lea después sabe si las entradas se derivaron de material con privilegio revisado.

### Paso 1: Identificar fuentes documentales

**Modo `--matter`:**

1. **Rutas proporcionadas por el usuario** — cualquier cosa que haya entregado en esta sesión (rutas de archivos, enlaces de drive, exportaciones de correo).
2. **Carpeta del caso por defecto** — del patrón de almacenamiento documental de CLAUDE.md, expandido para este slug (ej., `G:/Legal/Casos/acme-vs-nosotros-2026`).
3. **Fuentes declaradas** — la tabla `Almacenamiento documental` en CLAUDE.md, filtrada a las que este caso podría tocar (ej., archivo Gmail para comunicaciones del lado emisor, carpeta SharePoint del área legal).
4. **Preguntar** — si las fuentes parecen escasas, preguntar: "Puedo construir con lo que tengo, pero la cronología quedará incompleta. ¿Algo más que deba revisar? ¿Correos clave, contratos, memorandos internos, cartas notariales?"

**Modo `--documents`:**

1. **Exportación de producción / conjunto Bates** — el usuario señala el directorio de producción o un manifiesto; la skill lee por rango de identificación + fecha. <!-- ADAPT-PE: Ver flag ADAPT-PE anterior sobre numeración Bates. -->
2. **Conector eDiscovery** — si hay un conector MCP disponible (Everlaw, Relativity, DISCO, Aurora), extraer por custodio + rango de fechas. <!-- ADAPT-PE: Ver flag ADAPT-PE anterior sobre plataformas eDiscovery. -->
3. **Archivos por custodio** — si el usuario proporciona buzones de correo o exportaciones de disco por custodio, leerlos también.
4. **Preguntar** — si la cobertura parece escasa para un custodio o rango de fechas clave, preguntar.

### Paso 2: Extraer + leer

Para cada fuente con archivos legibles:

- **PDFs, correos (.eml), .docx, .txt** — leer directamente.
- **Archivos de correo (Gmail, Outlook)** — si hay un conector MCP autenticado, consultar por rango de fechas + contraparte / términos clave; de lo contrario, el usuario exporta los hilos relevantes a una carpeta.
- **Plataformas eDiscovery (Everlaw, Relativity, DISCO, Aurora)** — si hay conector disponible, extraer por custodio + rango de fechas; de lo contrario, el usuario proporciona una exportación. <!-- ADAPT-PE: Ver flag ADAPT-PE anterior sobre plataformas eDiscovery. -->

Si la skill no puede acceder a una fuente declarada, nombrarla explícitamente en la sección de Vacíos de la salida en lugar de continuar en silencio.

**No suplementar en silencio.** Si la cobertura de fuentes para una época del caso es escasa — menos documentos de los esperados para un período, un custodio cuyo buzón no es accesible, una producción que no ha llegado — reportar lo encontrado y detenerse. NO llenar vacíos con búsqueda web, búsqueda de registros públicos, o conocimiento del modelo sobre el caso sin preguntar. Decir: "Las fuentes devolvieron [N] eventos para [período / custodio]. La cobertura parece escasa. Opciones: (1) señalame fuentes adicionales (identificación de documento, carpeta, buzón), (2) intentar un conector MCP distinto si está configurado, (3) buscar en la web eventos de registro público en este período — los resultados se etiquetarán `[búsqueda web — verificar]` y deben contrastarse con una fuente primaria antes de usarlos, o (4) detenerme aquí y anotar el vacío. ¿Cuál preferís?" El abogado decide si acepta fuentes de menor confianza; la skill no decide por él.

**Atribución de fuente.** Etiquetar cada entrada de la cronología con el origen del evento: la ruta del archivo, número de identificación del documento, conector MCP o fuente de almacenamiento documental declarada para eventos extraídos de documentos recuperados (ya capturados en la columna Fuentes). Para cualquier evento o fecha que no pueda trazarse a un documento recuperado — ej., un hecho recordado de datos de entrenamiento del modelo, un evento de registro público encontrado por búsqueda web — etiquetarlo en línea: `[búsqueda web — verificar]`, `[conocimiento del modelo — verificar]`, o `[proporcionado por el usuario]` cuando el usuario declaró el hecho en sesión. Las entradas etiquetadas `verificar` conllevan mayor riesgo de fabricación que las entradas con fuente documental y deben revisarse primero. Nunca eliminar ni colapsar las etiquetas — son la señal más rápida para el abogado sobre qué entradas verificar antes de incorporarlas a un escrito.

**El etiquetado alcanza cada sección que afirme una conclusión jurídica, plazo o fecha calculada — no solo las entradas de la línea de tiempo.** La línea de tiempo proviene de documentos. La sección de Vacíos, la sección de Eventos clave, las líneas de vínculo con la teoría, y cualquier afirmación sobre prescripción, acto interruptivo, plazo de caducidad, fecha límite de presentación, cierre probatorio o determinación de privilegio son análisis jurídico que la skill escribe desde el conocimiento del modelo salvo que tengan fuente. <!-- ADAPT-PE: Los conceptos de "statute of limitations", "tolling event", "filing deadline", "discovery cutoff" provienen del proceso anglosajón. Los equivalentes peruanos son: prescripción extintiva (Art. 1989 CC), caducidad (Art. 2003 CC), suspensión e interrupción de la prescripción (Arts. 1994-1996 CC), plazos procesales peruanos (CPC, CPEP 2025), y cierre probatorio / saneamiento procesal (Arts. 465-468 CPC, Art. 159-161 CPEP 2025). Se requiere mapeo con un procesalista peruano para determinar los conceptos equivalentes exactos. --> Cada afirmación de este tipo lleva una etiqueta de procedencia: `[calculado de: <norma citada con etiqueta>]`, `[conocimiento del modelo — verificar]`, `[proporcionado por el usuario]`, o una etiqueta de conector de investigación si fue obtenida en esta sesión. Un plazo de prescripción sin etiqueta se presume `[conocimiento del modelo — verificar]`. Una línea de "evento clave" que caracteriza la relevancia jurídica de un hecho es análisis y necesita la etiqueta. La regla es simple: si es una afirmación sobre el derecho, no una afirmación sobre lo que dice un documento, debe llevar la misma etiqueta de procedencia que las entradas de la línea de tiempo. Cuando no hay un conector de investigación accesible y la skill está calculando plazos o citando normas, registrarlo en la línea **Fuentes:** de la nota del revisor (ver CLAUDE.md del plugin `## Salidas`) — no emitir un banner independiente.

### Paso 3: Extraer eventos

Para cada documento, identificar eventos fechados:

- **Correo:** `[fecha] [remitente] comunicó a [destinatario] [asunto/contenido]`
- **Reunión:** `[fecha] [asistentes] se reunieron sobre [tema]` (según registro de calendario o notas)
- **Decisión:** `[fecha] [quien decide] decidió [qué]` (según documento que la registra)
- **Presentación / escrito:** `[fecha] [parte] presentó [recurso/demanda/contestación]` <!-- ADAPT-PE: Las categorías procesales anglosajonas "motion/complaint/response" tienen equivalentes en el CPC peruano: los actos postulatorios (demanda, contestación, reconvención), los medios impugnatorios (recursos de reposición, apelación, casación, queja), y los escritos en general. Se requiere mapeo terminológico completo con un procesalista peruano para todas las categorías de escritos y actuaciones procesales. -->
- **Evento externo:** `[fecha] [algo ocurrió]` (contrato firmado, producto lanzado, regulador actuó, evento cruzó un umbral)

Generalmente un evento por documento. Ocasionalmente cero (sin fecha o sin evento establecido). A veces múltiples (resumen de reunión que cubre varias decisiones).

**Flag de privilegio por entrada (solo cuando postura_privilegio == B-mixto). Regla de tres estados — nunca decidir en silencio que un test subjetivo de privilegio no se cumple:**

- `priv: ok` — la fuente es **claramente** no privilegiada (escritos judiciales, correspondencia regulatoria, documentos públicos, comunicaciones con la contraparte sin intervención de nuestro abogado). Se usa solo cuando no hay una teoría de privilegio plausible.
- `priv: flag` — la fuente es claramente o probablemente privilegiada (comunicaciones con abogados, memorandos de producto de trabajo, borradores privilegiados, material de defensa conjunta). <!-- ADAPT-PE: Ver flag ADAPT-PE anterior sobre doctrinas de privilegio anglosajonas. --> **Valor por defecto para cualquier cosa incierta** — si la determinación del propósito dominante es dudosa, o la contemplación de litigio es límite, o el contenido es mixto, va aquí, no en `ok`.
- `priv: review` — fuente poco clara en sí misma, pero la skill no pudo determinarlo (sin metadatos de remitente/destinatario, ilegible, etc.).

Cuando `priv: flag` o `priv: review`, agregar `[SME VERIFY: estado de privilegio]` en línea para que el abogado lo vea durante la revisión. Sub-etiquetar renuncia al privilegio (puerta de una sola vía); sobre-etiquetar lo corrige el abogado en revisión (puerta de dos vías). Preferir el error recuperable. <!-- ADAPT-PE: "SME VERIFY" asume la existencia de un Subject Matter Expert (abogado revisor). En el contexto peruano, se referiría al abogado a cargo del caso como responsable de la revisión. -->

### Paso 4: Desduplicar

El mismo evento aparece en múltiples documentos: una reunión está en tres calendarios y genera un correo de resumen — eso es **un evento con cuatro fuentes**, no cuatro eventos. Consolidar. La entrada consolidada cita todas las fuentes.

### Paso 5: Etiquetar relevancia — según teoría del caso

Leer el hecho pivote y los hechos clave de `matter.md` (modo `--matter`) o de la sección `## Teoría del caso` de la configuración (modo `--documents`). Etiquetar cada evento:

- 🔴 **Clave** — el evento es parte del hecho pivote o un hecho clave a favor/en contra
- 🟡 **Relevante** — contexto, prueba indiciaria, respalda un argumento secundario
- ⚪ **Contexto** — útil para completitud, no va al escrito

**Disciplina:** una cronología de 300 entradas con 300 etiquetas 🔴 no tiene etiquetas. Reservar 🔴 para eventos que genuinamente moverían a un juzgador. En caso de duda, 🟡.

**Etiquetado límite:** cuando una entrada está entre 🔴 y 🟡 (o 🟡 y ⚪), etiquetar con la relevancia menor y agregar `[SME VERIFY — decisión de relevancia límite]` en línea. El criterio del abogado prevalecerá sobre la decisión de la skill. Una cronología que sobre-etiqueta con confianza es menos útil que una que expone su incertidumbre.

### Paso 6: Escribir

La salida por defecto es la cronología de trabajo. Variantes a pedido.

## Formatos de salida

### Cronología de trabajo (por defecto)

Ubicación: `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/chronology.md`. Completa, etiquetada, anotada. El documento de referencia con el que trabaja el abogado.

```markdown
[ENCABEZADO DE PRODUCTO DE TRABAJO — según configuración del plugin ## Salidas — varía según rol; ver `## Quién usa esto`]

> **Herencia de privilegio.** Esta cronología se deriva de documentos del caso que pueden estar amparados por privilegio abogado-cliente, protección de producto de trabajo, material de interés común / defensa conjunta, o una combinación. Hereda el estatus de protección de las fuentes. Distribuirla fuera del círculo de privilegio — a interesados del negocio fuera del encargo, a la contraparte, a un regulador — puede causar la renuncia a la protección tanto sobre la cronología como sobre las fuentes subyacentes. Almacenar con el material privilegiado del caso, marcar consistentemente con las convenciones de privilegio del despacho, y tomar decisiones de distribución deliberadamente. La elección de postura de privilegio capturada abajo es el sello de trazabilidad para cualquier decisión de distribución posterior. <!-- ADAPT-PE: Ver flag ADAPT-PE anterior sobre doctrinas de privilegio anglosajonas. Todo el concepto de "privilege inheritance", "waiver" y "privilege circle" requiere adaptación al régimen peruano de secreto profesional y reserva de la defensa. -->

# Cronología — [Nombre del Caso]

> Las etiquetas de relevancia (🔴/🟡/⚪) y los flags de privilegio (🔒) son lecturas de primer paso que requieren `[SME VERIFY]` antes de usarse en cualquier producto de trabajo externo (escritos, relación de hechos, informes para directorio, entregables a abogado externo).

**Caso:** [slug]
**Modo:** matter | documents
**Construida:** [YYYY-MM-DD]
**Fuentes:** [N] documentos en [tipos de fuente]
**Entradas:** [N] ([N] 🔴 / [N] 🟡 / [N] ⚪)
**Hecho pivote:** [una oración]
**Postura de privilegio:** A-revisado | B-mixto | C-abortado
**Entradas señalizadas:** [N] 🔒 *(solo presente cuando postura == B-mixto)*

---

## Línea de tiempo

| Fecha | Evento | Etiqueta | 🔒 | Fuentes |
|---|---|---|---|---|
| [YYYY-MM-DD] | [qué pasó, una oración] | 🔴/🟡/⚪ | [vacío / 🔒-flag / 🔒-review] | [rutas de archivo o identificación] |

---

## Eventos clave (🔴 solamente)

[Extraídos, cada uno con una línea sobre por qué importa para la teoría.]

### [fecha] — [título del evento]
- Qué: [una línea]
- Vínculo con la teoría: [por qué importa]
- Fuentes: [lista]

---

## Vacíos

**Rangos de fechas sin eventos:**
[rangos — ¿dónde están los documentos de este período?]

**Esperados pero faltantes:**
[eventos que esperaríamos ver documentados pero no aparecen — ej., "modificaciones contractuales entre 2024-06 y 2025-03 — no producidas"]

**Fuentes ilegibles:**
[fuentes declaradas en CLAUDE.md pero no accesibles en esta ejecución — ej., "exportación de eDiscovery — sin conector MCP; se necesita exportación"]

---

## Disciplina de marcadores

- `[VERIFY: afirmación fáctica — fecha, asistentes, contenido]` — aún no confirmado contra el documento fuente
- `[UNCERTAIN: caracterización jurídica — ej., si un evento constituye un hecho generador de obligación regulatoria]`
- `[CITE NEEDED: referencia al documento / foja / audiencia]` <!-- ADAPT-PE: "Bates / exhibit / depo page:line" son formatos de citación anglosajones. En Perú, las citas a documentos se hacen por: número de foja del expediente judicial, número de cuaderno, denominación del documento, o identificador del EJE (Expediente Judicial Electrónico). No existen las deposiciones (depo) en el proceso civil peruano. Se requiere mapeo a los formatos de citación forense peruana. -->
- `[SME VERIFY: estado de privilegio | decisión de relevancia límite]` — requiere criterio del abogado

---

## Versión
- v[N] construida el [fecha] desde [resumen de fuentes]
- v[N-1] construida el [fecha] (anterior, reemplazada)
```

### Cronología de Relación de Hechos (a pedido)

Filtrar solo 🔴 y 🟡 relevante. Presentar como prosa en orden narrativo cronológico — el esqueleto para la sección de hechos de un alegato. Cada párrafo es un evento o grupo estrechamente vinculado, con citas al expediente. <!-- ADAPT-PE: Ver flag ADAPT-PE anterior sobre "Statement of Facts" y formatos procesales peruanos. -->

**Filtro de privilegio por defecto:** cuando `postura_privilegio == B-mixto`, las entradas 🔒-flag y 🔒-review se **excluyen** por defecto. La variante de Relación de Hechos está destinada a eventual uso externo (escritos, disclosures, negociación con contraparte) — las entradas 🔒 no van ahí hasta que el abogado confirme el estado de privilegio. Si el usuario quiere incluir entradas 🔒 de todos modos, requerir aceptación explícita `--include-flagged`; capturar la aceptación en el encabezado de salida como registro permanente.

### Cronología específica por testigo (a pedido)

Filtrar a eventos donde un testigo nombrado es remitente, destinatario, asistente o sujeto. Alimenta la preparación de testigos y ayuda a reconstruir qué sabía un testigo y cuándo. <!-- ADAPT-PE: Ver flag ADAPT-PE anterior sobre preparación de testigos en el contexto peruano. -->

## Construcciones incrementales

Si `chronology.md` existe:

- Leer versión previa
- Construir nueva cronología desde las fuentes actuales
- Diferenciar: eventos nuevos (desde la última construcción), entradas modificadas (nuevas fuentes agregadas a eventos existentes), entradas eliminadas (raro; anotar por qué)
- Preservar el número de versión previa; escribir nueva versión con `v[N+1]`
- Mostrar resumen de lo que cambió

## Integración con matter.md / history.md

**Intencionalmente separados** (modo `--matter` de abogado interno). `history.md` es el registro corriente del abogado — decisiones, actualizaciones, hitos procesales, notas de estrategia interna. `chronology.md` es la línea de tiempo de hechos orientada a la estrategia procesal. Se superponen pero no se fusionan:

- Se emitió un hold → va en history.md (acción interna). Generalmente no en la cronología (no es un hecho de la controversia). <!-- ADAPT-PE: "Hold" en el contexto de litigación corporativa anglosajona se refiere a una orden interna de preservación de documentos (litigation hold) emitida cuando el litigio es razonablemente previsible, para cumplir con el deber de preservación de prueba electrónica bajo las reglas de eDiscovery. En Perú, no existe un equivalente procesal exacto. El deber de preservación de prueba en el proceso peruano opera bajo las reglas generales de buena fe procesal (Art. IV del Título Preliminar del CPC, Art. 109 CPC) y las consecuencias de la destrucción u ocultamiento de prueba (obstrucción a la justicia, indicio en contra). Se requiere análisis con un procesalista peruano. -->
- La contraparte envió una carta notarial de incumplimiento el 14 de marzo → va en chronology.md (🟡 — acredita su conocimiento). También en history.md si el intake lo referenció.
- Nuestro memorando de recomendación de reserva fue redactado → solo en history.md.

Cuando el abogado quiera eventos del historial en la cronología, puede copiarlos. Por defecto permanecen separados.

## Lo que esta skill no hace

- **Resolver contradicciones.** Cuando dos documentos dicen cosas distintas sobre cuándo ocurrió un evento, ambas entradas van con un flag. La resolución es decisión del abogado; puede requerir entrevista con testigos o prueba adicional.
- **Inventar eventos que no están en las fuentes.** Si no está en los documentos (ni en matter.md ni en la configuración como hecho capturado), no está en la cronología — pero "Vacíos" puede señalarlo como faltante.
- **Garantizar completitud.** Una cronología es tan buena como sus fuentes. Si la producción documental está en curso y solo ha llegado el 20%, la cronología lo refleja. Señalar la limitación.
- **Decidir el estado de privilegio por el usuario.** La barrera del Paso 0 fuerza la elección de postura; el flag `priv` por entrada captura la clasificación de primer paso. Las determinaciones reales de privilegio son decisión del abogado según flags `[SME VERIFY]`.

## Dudas de adaptación PE

A continuación se listan todos los flags `<!-- ADAPT-PE -->` insertados en este archivo, para consolidación por parte del supervisor legal peruano:

1. **Discovery / disclosure anglosajón vs. exhibición de documentos peruana:** El discovery anglosajón (disclosure/discovery) no existe en el proceso civil peruano. En Perú rige la exhibición de documentos (Art. 257-261 CPEP 2025 / Arts. 241-245 CPC) a pedido de parte y con autorización judicial. El alcance es sustancialmente más limitado.

2. **CPR 31.22 (Inglaterra) y Rule 26(c) (EE.UU.):** Normas procesales extranjeras sin equivalente directo en Perú. Las restricciones al uso de prueba en el proceso peruano provienen del debido proceso y del principio de licitud probatoria. El "contempt of court" no tiene equivalente exacto.

3. **Deposiciones (depositions):** No existen en el proceso civil peruano. La prueba testimonial se actúa en audiencia oral ante el juez. La preparación de testigos tiene restricciones éticas bajo el Código de Ética del Abogado Peruano.

4. **Sistema de numeración Bates:** No existe en Perú. Los documentos se identifican por foja, cuaderno, tomo del expediente judicial, o por identificador del EJE.

5. **Plataformas eDiscovery (Everlaw, Relativity, DISCO, Aurora):** No operan en el mercado peruano. Los equivalentes funcionales serían el EJE, la MPE, el SINOE, y sistemas de gestión documental internos.

6. **iManage y CLM:** Plataformas de gestión documental anglosajonas. En Perú se usan otros sistemas (SID, gestores documentales corporativos, carpetas compartidas).

7. **Statement of Facts (SoF) / brief:** No tiene equivalente estructural exacto en los escritos peruanos. Los hechos se exponen en los fundamentos de hecho de la demanda o en los antecedentes de ciertos recursos.

8. **Doctrinas de privilegio anglosajonas (attorney-client privilege, work-product doctrine, common interest, joint defense, Kovel):** Sin equivalente directo en Perú. La protección se fundamenta en el secreto profesional (Art. 30 Ley 30215, Código de Ética del Abogado Peruano, Art. 165 CPP). La doctrina de producto de trabajo no está reconocida. La renuncia (waiver) al privilegio es un concepto del common law sin desarrollo equivalente en la dogmática peruana.

9. **Statute of limitations vs. prescripción extintiva peruana:** Equivalente funcional en la prescripción extintiva (Art. 1989-2007 CC), pero las reglas de cómputo, suspensión e interrupción difieren. "Tolling events", "filing deadlines", "discovery cutoff" requieren mapeo a plazos procesales peruanos (CPC, CPEP 2025).

10. **Affirmative defenses (release, waiver, assumption of risk, comparative fault):** Equivalentes parciales en derecho civil peruano: renuncia de derechos, asunción de riesgo (no regulada explícitamente como defensa autónoma), concausa (Art. 1973 CC). Requiere mapeo preciso.

11. **Litigation hold:** No existe equivalente procesal exacto en Perú. El deber de preservación de prueba opera bajo las reglas generales de buena fe procesal y las consecuencias de la destrucción u ocultamiento de prueba.

12. **Categorías procesales anglosajonas (motion, complaint, response):** Tienen equivalentes en el CPC peruano: actos postulatorios (demanda, contestación, reconvención), medios impugnatorios (recursos de reposición, apelación, casación, queja). Requiere mapeo terminológico completo.

13. **Formatos de citación forense (Bates, exhibit, depo page:line):** En Perú, las citas se hacen por número de foja del expediente, número de cuaderno, denominación del documento, o identificador del EJE.

14. **SME VERIFY (Subject Matter Expert review):** El concepto asume un flujo de revisión por abogado revisor especializado. En el contexto peruano se refiere al abogado a cargo del caso como responsable último de la revisión jurídica.
