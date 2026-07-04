# MADRE — Arquitectura, puntos de dolor y repos candidatos

> **Qué es esto:** MADRE es un sistema personal de **mejora continua basado en ficheros** que corre en un solo PC
> (Windows + almacenamiento sincronizado) — un "cerebro documental" (reglas en Markdown/texto) + un loop de IA
> programado + subsistemas Python autónomos. Este documento es un **análisis honesto de su arquitectura, sus
> puntos de dolor reales y los repositorios open-source que podrían resolverlos**.
>
> **💬 Comenta, por favor.** Se publica en abierto justamente para recibir feedback: si conoces una herramienta
> mejor, ves un error, o resolviste un problema parecido, **abre un Issue** (o usa Discussions). Especialmente
> interesan ideas para los 3 dolores rojos: integridad del estado en ficheros, versionado/backup, y ejecutar un
> loop autónomo sin depender de que el PC esté encendido.
>
> *Nota: este README describe la arquitectura a nivel conceptual. No incluye credenciales, rutas privadas ni el
> detalle operativo interno; algunos nombres de fichero se citan solo como referencia de diseño.*

---

## (Contexto original del documento)

**Documento portátil** — pensado también para pegar en otra IA y pedirle repos que resuelvan estos puntos de dolor.
Cada punto de dolor trae su **frase de búsqueda lista** al final.

> **Actualización 2026-07-04:** este documento ya incluye una **§8-BIS "REPOS CANDIDATOS VERIFICADOS"** con búsqueda
> real en GitHub (`gh` API: estrellas, último push y licencia comprobados ese día) para los 3 dolores rojos (P1, P2,
> P3-BIS) + bonus P3/P5. Puedes pegar el documento para pedir MÁS candidatos, pero los mejores ya están cribados dentro.

---

## 0. CÓMO USAR ESTE DOCUMENTO (léelo tú, IA, antes de recomendar)

> **Prompt sugerido para pegar arriba de este documento:**
> «Eres un ingeniero que conoce el ecosistema open-source. Abajo tienes el inventario de un sistema
> personal de automatización llamado MADRE y sus puntos de dolor reales. Recomiéndame **repositorios
> de GitHub (o librerías PyPI / APIs gratis)** que resuelvan cada punto de dolor. **REGLAS:**
> (1) NO me recomiendes nada que ya esté en el INVENTARIO (§2) ni en YA ADOPTADO/DESCARTADO (§8).
> (2) Prioriza **gratis / open-source, mantenido activamente (commits < 6 meses), rápido (sin esperas
> largas), y fácil de integrar en un sistema basado en ficheros + Python + tareas de Windows (no-code,
> sin servidor 24/7)**. (3) Por cada candidato dame: nombre, ★ aprox, licencia, qué sustituye, qué
> trabajo elimina, y una pega honesta. (4) Si algo NO existe llave-en-mano, dímelo y propón el
> ensamblaje de piezas. Devuélvelo como TABLA comparativa por punto de dolor.»

**Lo que NO quiero que me recomienden** (para no perder el tiempo):
- Frameworks pesados que exijan servidor 24/7, Docker orquestado, o una base de datos gestionada
  (MADRE corre en 1 PC Windows con OneDrive, sin servidor).
- Servicios de **pago recurrente** o con cuota gratis ridícula, salvo que el salto de valor sea enorme.
- Cosas que **reinventen** el pipeline de vídeo, el loop, los hooks o los sensores que ya existen (§2).
- SaaS propietarios cerrados cuando hay equivalente OSS.

---

## 1. QUÉ ES MADRE (contexto en 6 líneas)

MADRE es un **sistema de mejora continua basado en ficheros** que corre en el PC de Tony (Windows + OneDrive).
No es una app: es un **cerebro documental** (reglas en `.md`/`.txt`) + un **loop de IA programado** (una
rutina de Claude Code que se auto-ejecuta ~4 veces/día) + **subsistemas Python autónomos** (tareas de Windows
que corren sin IA). Absorbe conocimiento (vídeos, enlaces), lo destila, aprende reglas de trabajo, y persigue
un objetivo: **generar beneficio de forma autónoma** (o al menos activos que ganen señal externa) sin que Tony
sea el cuello de botella. Su **cuello real hoy** = *activación / captura de valor externo* (produce mucho, activa
poco; ver §3). Todo el estado vive en ficheros dentro de la carpeta MADRE — esa es su fuerza (portable, auditable)
y su fragilidad (colisiones, sync, sin BD).

---

## 2. INVENTARIO — LO QUE MADRE YA TIENE (no lo dupliques)

### 2.1 El "cerebro" y las reglas
- **`CLAUDE.md`** (raíz) + **`SISTEMA/01_CORE/`** (`OBJETIVO_MADRE.txt`, `CONSTITUCION_MADRE.txt`,
  `ONTOLOGIA_OPERATIVA.txt`, `VECTORES_CRITICOS.txt`): la constitución, el kernel de decisión y ~16 reglas
  operativas (verificar-no-asumir, autonomía-por-defecto, buscar-antes-de-construir, adquirir-capacidades, etc.).
- **`MEMORY.md`** + carpeta `memory/` (fuera del repo, en `~/.claude/...`): memoria atómica tipo grafo
  (un fichero = un hecho, enlaces `[[nombre]]`). Índice en `MEMORY.md`.
- **`PROGRESO_META_PALANCAS.md`**: estado MEDIDO del cuello y las apuestas (escalera de palancas N0→N5).
- **`LIMITES_Y_PUNTOS_CIEGOS.md`**: registro vivo de límites (cada uno con un "falsador" — la señal del mundo
  que lo rompería) + puntos ciegos. Un watcher semanal (`madre-revisa-limites`) los re-chequea.
- **`MAPA_TEMAS.md`**, **`MODO_BUSQUEDA.md`** (cola de búsquedas pendientes), **`ADQUISICIONES.md`**
  (registro append-only de capacidades externas ya adoptadas), **`PENSAR COMO TONY.md`**, **`PENSAR COMO FABLE.md`**.

### 2.2 El LOOP de mejora continua (el motor "vivo")
- Rutina **`madre-mejora-continua`** (Claude Code scheduled task; cerebro en
  `~/.claude/scheduled-tasks/madre-mejora-continua/SKILL.md`). Cadencia ~cada 6h (cron `45 1,7,13,19 * * *`).
  **Migrada de Cowork → Claude Code el 2026-07-04** (registro en `MIGRACION_LOOP_2026-07-04/LEEME_MIGRACION.md`).
- **Diseño anti-colisión (riesgo #1 del sistema):** un solo escritor del CORE = el loop. Otras sesiones aportan
  dejando ficheros nuevos en **`BUZON_ENTRANTE/DESDE_CLAUDE_<fecha>_<tema>.txt`** (nombre único). Lock:
  `SISTEMA/.lock` y `.RUTINA_ACTIVA.lock`.
- **`BUZON.txt`** (bandeja) + **`00_COWORK_OBJETIVOS`** (objetivos + parking-lot de tareas opcionales).
- Herramientas Python del sistema en **`SISTEMA/04_TOOLS/`** (métricas, etc.).

### 2.3 Los HOOKS de enforcement (Claude Code, este PC)
En `.claude/hooks/*.py` + `.claude/settings.json` (algunos wireados global en `~/.claude/settings.json`):
- `check_pie_en_simple.py` — obliga a cerrar cada turno con un "pie en simple" (resumen llano). Autotest 35/35.
- `check_captura_correccion.py` — detecta cuando alguien vio un error/mejora que la IA no vio, y obliga a aprender.
- `check_contrato_alcance.py` — impide que un diagnóstico global re-scopee en silencio la misión local.
- `check_buzon_suffix.py` — bloquea crear `DESDE_CLAUDE_<fecha>.txt` sin sufijo de tema (anti-colisión).
- `check_traspaso_unico.py` — obliga a que los traspasos de chat sean ficheros únicos auto-referenciados.
- **Recall-first hook** (SessionStart) — inyecta el índice de PLAYBOOKS + punteros al empezar sesión.
- Todos **fail-open** (si el hook falla, deja pasar).

### 2.4 El PIPELINE de absorción de vídeo/enlaces (autónomo, sin IA)
Carpeta **`VIDEO_PIPELINE/`** + tarea Windows **`MADRE_VIDEO_INBOX`** (cada 3h, Python puro, **0 tokens de Claude**).
- **`procesar_link.py`** (motor, ~1300 líneas): descarga (yt-dlp/tikwm/fxtwitter/r.jina.ai) + extrae evidencia
  (transcripción + OCR + descripción visual) → JSON. Cadena de fallback robusta:
  **Gemini (multi-clave, rota 5) → Groq → OpenRouter → Whisper local (CPU) → OCR local RapidOCR (CPU)**.
- **`procesar_inbox.py`** (orquestador, ~500 líneas): lee `VIDEO_INBOX.txt`, compone bloques en
  `VIDEO_DESTILADO.txt`, gestiona colecciones TikTok, cuentas vigiladas, tags de prioridad (`#ecom`).
- **`salud_pipeline.py`** (centinela): detecta roturas silenciosas → `SALUD_PIPELINE.txt` (semáforo).
- **`router_vision.py`** (cascada de visión por confianza; listo pero AÚN sin cablear).
- Soporte: `clasificar_negocios.py`, `recomendar_cuentas.py`, `repartir_entrada.py`, `respaldo.py`,
  `reintentar_fallidos.py`, `comparar_modos.py`, `validar_apis.py`, `auto_fuentes.py`.
- **8 "guardas" de robustez** ya aplicadas (liveness positiva, degradar-no-morir, timeout-a-todo, lock PID-aware,
  rename atómico, normalizar-tipos, verificar-con-file-tools-no-bash). ~20 ficheros de test.
- Ficheros fijos (NO mover): `VIDEO_INBOX.txt`, `VIDEO_DESTILADO.txt`, `VIDEO_NUEVOS.txt`, `COLECCIONES.txt`,
  `ESTADO_COLECCIONES.txt`. Backlog ya destilado en `MEJORAS_MADRE.md` (4583 líneas) y `OPORTUNIDADES_NEGOCIO_TONY.md`.

### 2.5 Subsistemas de apoyo (cada uno = carpeta + tarea/script)
- **`SENSOR_RESURRECCION/`** — indexa conocimiento dormido (`SISTEMA/DESCARTES.tsv`) y lo resucita cuando se
  cumple su condición; surfacea al panel «MADRE te dice» (#madreHabla). Semanal.
- **`COSECHA_CHATS/`** — tarea Windows `MADRE_COSECHA_CHATS` (cada 3h): recoge el "pie 📍" de chats abandonados
  → `PANEL_CHATS.md` + buzón + aviso ntfy. Permite abandonar chats sin ritual de cierre.
- **`SEGUNDA_OPINION/`** — `segunda_opinion.py`: panel de modelos externos GRATIS (Kimi/Gemini/Groq/OpenRouter)
  para 2ª opinión en momentos clave. Planes gratis limitan (429). Plantilla `PARA_REVISION_EXTERNA.md`.
- **`NOTIFICADOR_MOVIL/`** — ntfy + hooks: avisa al móvil cuando un chat de Code termina/espera.
- **`VIGIA_LATIDO/`** — heartbeat watcher; `ULTIMO_ESTADO.txt`. Vigila que las tareas sigan vivas.
- **`WIDGETS/`** — almacén de widgets; MADRE los puntúa/promueve con evidencia.
- **`TRASPASOS/`** + `TRASPASO_DE_CHAT.md` — protocolo de traspaso de chat (ficheros únicos, canónico = router).
- **El "cuaderno" / web** — primer activo público. Vive en un **repo de git APARTE** (NO dentro
  de MADRE), desplegado en GitHub Pages, backend en **Supabase** (comentarios/feedback, free tier). App con un
  **🌳 mapa** como núcleo; artículos "field notes" en dev.to. Estado ~v0.25. Rutina `cuaderno-feedback` (horaria)
  procesa el feedback. Fricción: el repo separado exige sincronizar docs a mano; señal externa aún = 0.

### 2.6 Capacidades externas ya conectadas (MCP / conectores)
- **Nimble MCP** (`nimble_search`, `nimble_extract`) — búsqueda web + render de páginas JS, **funciona headless**
  dentro de la tarea programada (rompió los límites L2 y L7). Es de pago/token.
- **Bright Data** (scraping, pago/token), **GitHub**, **Gmail**, **Desktop Commander** (solo READ-only headless).
- **Claude Code dynamic workflows** (orquestación multi-agente nativa; ya validado).

---

## 3. EL CUELLO #1 (dónde MADRE cree que está su dolor principal)

> **Según `PROGRESO_META_PALANCAS.md` y `LIMITES_Y_PUNTOS_CIEGOS.md`, el cuello dominante NO es generar. Es
> ACTIVAR / CAPTURAR VALOR EXTERNO.** Hoy: **0 € de ingreso autónomo, 0 estrellas orgánicas, 0 usuarios reales.**

Tres límites abiertos lo describen:
- **L1 — Sin actuador externo que produzca beneficio solo:** las "manos" de MADRE alcanzan la red (leen, llaman
  APIs) pero **ninguna cierra valor con un tercero** sin un humano por ciclo.
- **L5 — Activo commodity:** lo que produce (resúmenes de gurús, análisis) es demasiado intercambiable para mover
  un selector externo (comprador, estrella). *Este es el cuello aguas-arriba.*
- **L6 — Construir y NO publicar/activar:** modo de fallo dominante y probado (79 vídeos, 0 publicaciones).
- **Punto ciego PC2 — Sobre-genera y sub-activa:** produce stock muerto (ideas N3 sin usar) y activa poco.

**Implicación para la búsqueda de repos:** los repos MÁS valiosos son los que ayuden a **cerrar el bucle hacia
afuera** (distribución autónoma, captura de demanda real, publicación/lanzamiento semi-automático, medir señal
externa), no los que añadan más capacidad de *generar* hacia adentro.

---

## 4. PUNTOS DE DOLOR PRIORIZADOS

> Formato por punto: **descripción · por qué duele · qué mejora al resolverlo · 🔎 frase de búsqueda GitHub ·
> (sugerencias sin verificar)**. Prioridad: **P1 = más doloroso**.

### 🔴 P1 — Colisión multi-escritor y pérdida de datos en ficheros (OneDrive + varias IAs/chats)
- **Descripción:** todo el estado son ficheros planos en una carpeta sincronizada por OneDrive. Varias sesiones
  de IA + el loop + tareas de Windows + el móvil pueden escribir "a la vez". Ya ha corrompido/pisado contenido
  real (una nota se sobrescribió el 2026-06-26; el propio CLAUDE.md lo llama "riesgo #1").
- **Por qué duele:** la defensa actual es *disciplina* (un solo escritor del CORE, nombres únicos, locks caseros,
  hooks). Es frágil: depende de que cada actor respete la convención y de que OneDrive no sirva copias viejas.
  **Evidencia real:** OneDrive ya **truncó** un backup de `CONSTITUCION` a mitad de escritura (151→115 líneas);
  el `.lock` a veces no se puede borrar (EPERM en disco sincronizado). No hay atomicidad garantizada — por eso
  la regla "verifica con Read/Grep, NUNCA con bash" (bash lee copias viejas del mount de OneDrive).
- **Qué mejora:** un mecanismo de **concurrencia/append seguro** (log estructurado, journaling, o una BD embebida
  de 1 fichero) que haga imposible perder una escritura, sin montar un servidor.
- **🔎 GitHub:** `embedded single-file database file-based concurrent append-only Python` ·
  `git-backed structured storage crash-safe local` · `multi-writer file lock library python cross-platform`
- **Sugerencias (sin verificar):** SQLite (WAL mode) o **DuckDB** como capa de estado; **`filelock`** (py) para
  locks robustos multiplataforma; **`sqlitedict`**/**`TinyDB`** para KV sobre 1 fichero; **Dolt** (git-for-data)
  si se quiere versionado tipo git de datos.
- ✅ **Repos VERIFICADOS (2026-07-04): ver §8-BIS** → `filelock`, SQLite+`sqlite-utils`, `litestream`, Dolt, Kopia.

### 🔴 P2 — Sin copia de seguridad fuera del PC (todo vive en 1 máquina + OneDrive)
- **Descripción:** MADRE ES la carpeta. Si el PC muere o OneDrive corrompe, no hay respaldo externo versionado.
  Hay `respaldo.py` local y `SISTEMA/03_ARCHIVO/_backups/` (que además está **inflado** con cientos de `.bak`).
- **Por qué duele:** riesgo existencial + los backups locales saturan disco y ensucian el árbol.
- **Qué mejora:** **backup automático versionado a un remoto gratis** (git privado, o snapshots cifrados a un
  bucket), con retención/poda que funcione en unidades sincronizadas (la poda falla hoy con EPERM en OneDrive).
  **Ángulo clave:** MADRE **no usa git** — el control de versiones es OneDrive (historial de solo **90 días** por
  defecto). Migrar el estado a un **repo git privado** daría: commits atómicos, historial/blame, revertir a un
  instante exacto, y respaldo fuera del PC — resolviendo P1 y P2 de una vez. (Es la "oportunidad #1" que señalan
  los propios diagnósticos internos.)
- **🔎 GitHub:** `automated incremental backup git remote snapshot retention python` ·
  `restic borg backup scheduled windows` · `backup rotation prune EPERM onedrive safe`
- **Sugerencias (sin verificar):** **restic** o **BorgBackup** (backup incremental cifrado + poda); **git** a un
  repo privado de GitHub como respaldo versionado; **Kopia** (backup con UI + retención).
- ✅ **Repos VERIFICADOS (2026-07-04): ver §8-BIS** → **Kopia** (mejor encaje Windows), restic, `litestream`, git.

### 🔴 P3 — Sin actuador externo autónomo: MADRE no cierra valor con terceros sola (L1)
- **Descripción:** el objetivo de MADRE es generar beneficio sin que Tony sea el cuello, pero ninguna rutina
  headless puede **publicar, vender, o cerrar un trato** sin acción humana por ciclo. Los MCP de escritura
  (Gmail, navegador logueado) solo existen en sesión de chat, no en runs programados (L3).
- **Por qué duele:** es el techo del proyecto. Todo lo demás alimenta un cuello que sigue cerrado.
- **Qué mejora:** un **actuador headless fiable** — poder publicar contenido, abrir PRs/issues, cross-postear,
  o mantener una **sesión autenticada persistente** (cookies) desde una tarea de Windows, sin login manual.
- **🔎 GitHub:** `headless authenticated session persistence playwright storage_state cookies` ·
  `auto post social media cross-poster self-hosted api` · `scheduled publish github pr bot python`
- **Sugerencias (sin verificar):** **Playwright** `storage_state` / `launch_persistent_context` (sesión
  logueada persistente — ya está en la cola de MADRE como falsador de L1/L3); **Postiz** (cross-posting OSS —
  ojo: MADRE lo marcó "táctico-solo-Tony", verificar); **`gh` CLI** / PyGithub para actuar sobre repos.
- ✅ **Repos VERIFICADOS (2026-07-04): ver §8-BIS** → `gh` CLI / PyGithub (actuador GitHub), Playwright, Mixpost/Postiz.

### 🔴 P3-BIS — El loop solo corre si la app Claude Code está ABIERTA en el PC de Tony (sin ejecución en la nube)
- **Descripción:** el loop de mejora continua es una *scheduled task de Claude Code* → **solo se dispara mientras
  la app está abierta en el PC**. Si Tony apaga el equipo, se cae la red, o la app se cierra, el loop se para.
  El watchdog externo (`VIGIA_LATIDO`, tarea de Windows) **avisa** si el CORE no se escribe en 24h, pero **no puede
  reiniciar** el loop. (Ya pasó: 61h de loop caído el 2026-07-02, detectado por el vigía.)
- **Por qué duele:** contradice el objetivo de MADRE (mejorar sin que Tony sea el cuello). El "cerebro" que aprende
  depende de la máquina de Tony encendida. No hay respaldo en la nube.
- **Qué mejora:** un **scheduler en la nube** (GitHub Actions / un runner barato) que ejecute el loop contra el
  repo (idealmente git, ver P2) sin depender del PC — o al menos un auto-reinicio fiable del task local.
- **🔎 GitHub:** `github actions scheduled agent commit to repo cron self-hosted` ·
  `run llm agent on schedule cloud cheap serverless` · `autonomous repo bot scheduled headless`
- **Sugerencias (sin verificar):** **GitHub Actions** con `schedule:` (gratis para repos, corre sin el PC);
  un **self-hosted runner** en un mini-VPS; frameworks tipo **AutoGPT**/agent-runners solo como referencia (pesados,
  filtrar por "sin servidor 24/7" si se quiere seguir en 1 PC). Nota: hoy MADRE ya alcanza red headless vía Nimble
  MCP dentro del task — la pieza que falta es el *disparo* independiente del PC.
- ✅ **Repos VERIFICADOS (2026-07-04): ver §8-BIS** → **GitHub Actions `schedule:`** (la vía) + `git-auto-commit-action`; **dagu** para orquestador local file-based; `act` para probar.

### 🟠 P4 — Loop lento / esperas: cadencia cron cada ~6h y latencias de absorción
- **Descripción:** el loop corre 4 veces/día (cron). No hay reacción a eventos (un drop en el buzón espera a la
  siguiente pasada). El pipeline de vídeo espera cuotas Gemini, descargas yt-dlp (15-60s), timeouts de YouTube
  largos (>20min → 168MB → timeout 300s).
- **Por qué duele:** feedback lento; cosas que podrían dispararse por evento esperan horas. Días de alto volumen
  → absorción cae a 0 [HECHO] por cuota agotada, sin recarga automática.
- **Qué mejora:** **disparo por eventos** (file-watcher que lance la pasada cuando cambia el buzón) + **cola de
  trabajos** ligera; y para vídeo, subtítulos-primero para YouTube largos.
- **🔎 GitHub:** `file watcher trigger script on change windows python watchdog` ·
  `lightweight local job queue no server python persistent` · `event driven automation local filesystem`
- **Sugerencias (sin verificar):** **`watchdog`** (py, file events); **`APScheduler`** (scheduling + intervalos
  finos en proceso); **huey** / **RQ** (colas ligeras, aunque RQ pide Redis — preferir huey con SQLite).

### 🟠 P5 — Sobre-genera y sub-activa: falta captura de DEMANDA externa real (PC2 + L5)
- **Descripción:** MADRE destila mucho conocimiento pero no sabe **qué quiere el mundo AHORA**. No hay una
  entrada de señal de demanda (qué problemas piden en Reddit/HN/GitHub issues) que oriente qué construir.
- **Por qué duele:** construye commodities que nadie pidió (L5). El cuello no es generar, es *elegir bien apuntado*.
- **Qué mejora:** **minería de demanda** — agregación + clustering de "pain points" reales desde foros/issues para
  priorizar qué activo construir con probabilidad de captar señal.
- **🔎 GitHub:** `reddit hackernews pain point mining demand discovery clustering` ·
  `scrape user problems feature requests aggregate rank python` · `market demand signal indie hacker idea validation`
- **Sugerencias (sin verificar):** **GitHub Search API** (`sort:reactions`) + **Algolia HN API** (ambos gratis —
  ya identificados por MADRE); estudiar (no adoptar) **GummySearch** (taxonomía de dolor en Reddit),
  **BigIdeasDB**/pipelines tipo "idea mining" como referencia de arquitectura.

### 🟠 P6 — Dependencia de cuota Gemini para la descripción visual (único punto de pago/cuota real del pipeline)
- **Descripción:** transcripción (Whisper) y OCR (RapidOCR) ya son locales/gratis, pero la **descripción visual**
  depende de Gemini→Groq→OpenRouter (todos cuota/opt-in). Días de volumen alto → se agota → "mejor algo que nada".
- **Por qué duele:** es el único eslabón que puede parar la absorción de la parte visual, y no avisa "recarga claves".
- **Qué mejora:** un **modelo de visión local (VLM) gratis** que corra en CPU aceptable, o un caption/visión local
  como fallback real, para no depender de cuota cloud.
- **🔎 GitHub:** `local vision language model CPU image captioning offline lightweight` ·
  `on-device VLM inference gguf llama.cpp vision` · `free image description model no api quota`
- **Sugerencias (sin verificar):** **moondream** (VLM diminuto, corre local); **llama.cpp** + un VLM cuantizado
  (LLaVA/Qwen-VL gguf); **BLIP-2**/**Florence-2** para captioning local (verificar coste CPU).

### 🟡 P7 — Fragilidad de scrapers de terceros (tikwm, fxtwitter, r.jina.ai)
- **Descripción:** slideshows de TikTok (`/photo/`) dependen de **tikwm** (3rd-party, se cae/bloquea); texto de
  Twitter/X de **fxtwitter/vxtwitter**; shares de ChatGPT de **r.jina.ai**. Sin garantía de disponibilidad.
- **Por qué duele:** cuando uno cae, ese tipo de contenido deja de absorberse silenciosamente (lo detecta el
  centinela, pero no lo arregla).
- **Qué mejora:** librerías/extractores mantenidos con **múltiples backends** para TikTok foto, X, y lectores web,
  que reduzcan el "single point of failure".
- **🔎 GitHub:** `tiktok photo slideshow downloader api multiple fallback maintained` ·
  `twitter x post text extractor no login python maintained` · `readability web article extractor self-hosted`
- **Sugerencias (sin verificar):** **gallery-dl** (soporta TikTok/foto y muchísimos sitios, muy mantenido);
  **snscrape** (X sin login — verificar estado actual); **trafilatura** / **Postlight Parser** (extracción de
  artículos web robusta, self-hosted, evita depender de r.jina.ai).

### 🟡 P8 — "Bomba de teoría": sin ejecutor nativo, MADRE es cerebro documental (autonomía ejecutable 3/10)
- **Descripción:** el diagnóstico de autonomía (`SISTEMA/03_ARCHIVO/DIAGNOSTICO_10.txt`) puntúa "autonomía
  ejecutable / código" en **3/10**: MADRE no tiene un runtime propio (`main.py`/`executor.py` son "ficheros
  fantasma"), depende de la plataforma externa (Claude Code / tareas Windows) para actuar. Sin estímulo externo,
  los loops tienden a "modo investigación recursivo" y generan conceptos auto-validados sin uso.
- **Por qué duele:** riesgo de hipertrofia inútil (mucha gobernanza, poco mundo movido) + ningún watchdog propio
  si una tarea externa falla.
- **Qué mejora:** un **orquestador/runtime ligero** que encadene los scripts como un DAG con estado, reintentos y
  watchdog — sin servidor, sobre ficheros.
- **🔎 GitHub:** `lightweight local workflow orchestrator DAG python no server file-based` ·
  `task runner retries watchdog scheduled windows python` · `durable local pipeline state machine embedded`
- **Sugerencias (sin verificar):** **Prefect** (flujos con estado, se puede correr local); **`doit`** (task runner
  tipo make, file-based, ligero); **Dagster** (más pesado, quizá overkill — verificar). Nota: MADRE ya validó que
  para **1 PC** los frameworks 24/7 (Uptime-Kuma/healthchecks) NO encajan — filtrar por "sin servidor".

### 🟡 P9 — Backups/archivo inflados y poda que falla en OneDrive (EPERM)
- **Descripción:** `SISTEMA/03_ARCHIVO/_backups/` tiene cientos de `.bak`; la poda (cap=7) falla con errores
  EPERM en unidades sincronizadas → el disco puede saturarse y el árbol se ensucia (dificulta findability).
- **Por qué duele:** operativo, pero acumulativo: degrada la navegación y arriesga espacio en disco.
- **Qué mejora:** una **rotación/retención de backups** robusta ante bloqueos de sync (reintentos, mover-a-papelera
  en vez de borrar, compresión).
- **🔎 GitHub:** `log backup rotation retention compress prune windows onedrive safe python` ·
  `file cleanup policy TTL move to trash recoverable` · `logrotate equivalent python cross-platform`
- **Sugerencias (sin verificar):** **`send2trash`** (borrado recuperable, evita perder datos); lógica tipo
  **logrotate**; comprimir `.bak` viejos a un único `.zip` con retención por antigüedad.

### 🟢 P10 — Cold-start de audiencia: los primeros ~100 usuarios/estrellas no llegan solos (L4)
- **Descripción:** GitHub tiene descubrimiento orgánico pero el arranque necesita un empujón humano (post en la
  comunidad correcta). MADRE no puede sembrar audiencia sola sin caer en spam.
- **Por qué duele:** aunque produzca un buen activo (L5), sin distribución no gana señal externa → el bucle no cierra.
- **Qué mejora:** herramientas/plantillas de **lanzamiento** (Show HN, awesome-lists PRs, dev.to) semi-automatizadas
  y éticas, + medición de la señal resultante.
- **🔎 GitHub:** `product launch checklist show hn awesome list submission automation` ·
  `github readme stars growth organic distribution toolkit` · `dev content cross-post scheduler open source`
- **Sugerencias (sin verificar):** listas tipo **awesome-launch** / plantillas "Show HN"; **`gh` CLI** para PRs
  automáticos a awesome-lists (con revisión humana); analítica sin cookies **GoatCounter** (ya en cola de MADRE)
  para medir la señal.

---

## 5. INEFICIENCIAS ACTUALES A REEMPLAZAR (actual → problema → qué busco)

| # | Actual (lo que hay hoy) | Problema | Qué tipo de solución busco |
|---|---|---|---|
| I1 | Estado en ficheros planos + locks caseros + convención "1 escritor" | Colisiones, escrituras perdidas, depende de disciplina | Capa de persistencia concurrente-segura sobre 1 fichero (SQLite WAL/DuckDB) o append-log |
| I2 | Verificación con Read/Grep porque **bash da copias viejas en OneDrive** | OneDrive sirve lecturas obsoletas/truncadas → conteos falsos | Fuente de verdad no sujeta a lag de sync (BD local fuera de la ruta sincronizada, o sync-aware) |
| I3 | Loop por **cron cada ~6h** | Latencia alta; nada reacciona a eventos | Disparo por evento (file-watcher) + intervalos finos, sin servidor |
| I4 | Descripción visual depende de **cuota Gemini** (cloud) | Se agota en días de volumen; no recarga sola | VLM local en CPU como fallback real (moondream/llama.cpp) |
| I5 | Scrapers de terceros sueltos (**tikwm/fxtwitter/r.jina.ai**) | Single point of failure por tipo de contenido | Extractores mantenidos multi-backend (gallery-dl/trafilatura) |
| I6 | Backups `.bak` a mano + poda cap=7 que **falla (EPERM)** | Disco saturable, árbol ensuciado, poda inoperante | Backup incremental cifrado + retención robusta (restic/borg) + borrado recuperable |
| I7 | Sin ejecutor propio; **scripts sueltos** lanzados por tareas Windows | Sin DAG, sin estado, sin watchdog, sin reintentos unificados | Orquestador ligero file-based (doit/Prefect-local) |
| I8 | 2ª opinión con **planes gratis** que dan 429 | Rate limits cortan las consultas en momentos clave | Gateway multi-proveedor con rotación de claves (LiteLLM — ya candidato) |
| I9 | Fallback web final = **Nimble / Bright Data (pago/token)** | Coste por uso; dependencia externa de pago | Maximizar cobertura con lectores gratis antes del fallback de pago |
| I10 | Colecciones TikTok **añadidas a mano** a `COLECCIONES.txt` | Descubrimiento manual, no automático | Auto-descubrimiento de "carpetas/cuentas seguidas" |
| I11 | Versionado = **OneDrive** (historial 90 días), **sin git** | Sin commits atómicos, sin revertir a instante exacto, sin blame | Repo git (mejor privado) como fuente de verdad del estado |
| I12 | Loop = task de Claude Code **atado a la app abierta** | Se para si el PC se apaga; sin nube; el vigía avisa pero no reinicia | Scheduler en la nube (GitHub Actions) o auto-reinicio fiable |
| I13 | Fechas "falsador" revisadas **a mano** | Una hipótesis caducada puede quedarse en el CORE si nadie la revisa | Auto-killer por fecha: al vencer sin señal → archivar + avisar |
| I14 | Hooks sin **suite de tests** ni CI; fallan en silencio (fail-open) | Bugs de hook se descubren en producción; degradación silenciosa | pytest + CI para los hooks; health-check de que el hook corre |
| I15 | `SKILL.md` del loop vive en 2 sitios (vivo en `~/.claude` + copia en repo) | Pueden divergir si se edita la UI sin re-sincronizar | Verificación/auto-heal de que copia-viva == copia-repo |

---

## 6. CRITERIOS DE INTEGRACIÓN (filtro para juzgar cada repo candidato)

Un repo candidato **entra** solo si cumple la mayoría de:
1. **Gratis / open-source** (licencia permisiva MIT/Apache/BSD ideal; AGPL/GPL ok para uso propio). Evitar "sin licencia".
2. **Mantenido activamente** (commits en los últimos ~6 meses, issues respondidas, releases recientes).
3. **Rápido / sin esperas largas** (nada que meta latencias de minutos por ítem; preferible local/CPU).
4. **Fácil de integrar en no-code basado en ficheros** (Python puro o CLI; **sin servidor 24/7**, sin Docker
   orquestado obligatorio, sin BD gestionada externa; corre en 1 PC Windows con OneDrive).
5. **No reinventa lo que MADRE ya tiene** (ver §2 y §8). Aporta una capacidad que hoy NO existe o la hace
   claramente mejor (más rápido, gratis en vez de pago, más robusto).
6. **Reversible / bajo riesgo** (se puede quitar sin romper el CORE; no intercepta el flujo de Claude Code como
   proxy — MADRE ya rechazó una herramienta por "modo proxy delante de Claude Code").
7. **Peldaño correcto** (regla de MADRE): preferir **librería > servicio/API > repo entero > técnica/paper >
   construir el hueco**. Un servicio de pago con dependencia recurrente va por debajo de una librería.

**Además, veredicto POR PIEZAS, no repo-entero SÍ/NO:** si solo un módulo/patrón sirve, se roba esa parte y se
adapta. Y **aprender ≠ redistribuir**: se puede copiar la *idea/arquitectura* de cualquier repo aunque no se adopte.

---

## 7. RESUMEN — TODAS LAS FRASES DE BÚSQUEDA JUNTAS (copia-pega)

```
P1  embedded single-file database file-based concurrent append-only Python
P1  multi-writer file lock library python cross-platform
P2  automated incremental backup git remote snapshot retention python
P2  restic borg backup scheduled windows prune
P3  headless authenticated session persistence playwright storage_state cookies
P3  auto post social media cross-poster self-hosted api
P3B github actions scheduled agent commit to repo cron self-hosted
P3B run llm agent on schedule cloud cheap serverless autonomous
P4  file watcher trigger script on change windows python watchdog
P4  lightweight local job queue no server python persistent
P5  reddit hackernews pain point mining demand discovery clustering
P5  scrape user problems feature requests aggregate rank python
P6  local vision language model CPU image captioning offline lightweight
P6  on-device VLM inference gguf llama.cpp vision
P7  tiktok photo slideshow downloader api multiple fallback maintained
P7  readability web article extractor self-hosted trafilatura
P8  lightweight local workflow orchestrator DAG python no server file-based
P8  task runner retries watchdog scheduled windows python
P9  log backup rotation retention compress prune windows onedrive safe python
P9  file cleanup policy TTL move to trash recoverable python
P10 product launch checklist show hn awesome list submission automation
P10 github readme stars organic distribution toolkit
```

---

## 8. NO RE-RECOMENDAR — LO QUE MADRE YA ADOPTÓ / DESCARTÓ (de `ADQUISICIONES.md` + `MODO_BUSQUEDA.md`)

**YA adoptado o candidato firme (no re-proponer):**
- **LiteLLM** (52k★) — gateway multi-proveedor / rotación de claves (candidato P1 para el 429).
- **deepeval** (16.6k★) — juez LLM por rúbrica.
- **Claude Code dynamic workflows** — orquestación multi-agente nativa (validado).
- **GoatCounter** (5.8k★) — analítica sin cookies (para la primera landing).
- **Tailwind standalone CLI** (sin Node), **FineWeb-Edu** (receta juez mini español), **Playwright storage_state**
  (sesión persistente — candidato para P3), **LLMLingua + sumy** (compresión de contexto local),
  **GitHub Search API + Algolia HN** (ranking de demanda — relevante a P5).

**YA DESCARTADO (no re-proponer):**
- **Headroom-ai** (55.8k★) — rechazado por "modo proxy" (intercepta la API de Claude Code) — viola el criterio §6.6.
- **Umami / analítica pesada** — rechazado (Goodhart: medir el vacío mientras la señal externa = 0).
- Uptime-Kuma / healthchecks.io — rechazados para VIGIA_LATIDO (exigen servidor 24/7; script propio de 200 líneas
  es mejor para 1 PC).
- **projectmem** (118★) / **LangMem** (1.5k★) — NO adoptar, solo aprender su patrón "Memory-as-Governance".
- **free-llm-gateway** (140★, sin licencia), y varios repos **alucinados por IAs** (sari-maji/*, noorkhokhar99/*,
  ai-mem → 404): **verificar TODA sugerencia con `gh`/visitando el repo antes de fiarse** — las IAs inventan repos.

---

## 8-BIS. REPOS CANDIDATOS VERIFICADOS (búsqueda con `gh` API, 2026-07-04)

> Verificados uno a uno con la API de GitHub el **2026-07-04**: ⭐estrellas, **último push** (señal de mantenimiento)
> y **licencia** son datos reales de ese día, no de memoria. Recordatorio de licencias (regla 13 de MADRE):
> para **uso propio** cualquier licencia vale (AGPL/GPL incluidas); solo importa si algún día **redistribuyes**
> el código. Veredicto = encaje con "1 PC Windows + OneDrive + ficheros + Python, sin servidor 24/7".

### 🔴 P1 + P2 — Integridad del estado, versionado, locks y backup externo

| Repo | ⭐ | Últ. push | Lic. | Qué aporta | Veredicto de encaje |
|---|---|---|---|---|---|
| [tox-dev/filelock](https://github.com/tox-dev/filelock) | 961 | 2026-07-03 | MIT | Lock de fichero **multiplataforma** en Python | ✅ **ADOPTAR YA** — reemplaza los locks caseros; Windows-safe, 5 líneas |
| [benbjohnson/litestream](https://github.com/benbjohnson/litestream) | 13.8k | 2026-07-02 | Apache-2.0 | **Replicación streaming de SQLite** a un remoto (S3/disco) | ✅ **FUERTE para P2** si el estado pasa a SQLite: backup continuo sin cron |
| [dolthub/dolt](https://github.com/dolthub/dolt) | 23.8k | 2026-07-04 | Apache-2.0 | **"Git for Data"**: BD SQL con commit/branch/diff/merge | 🟡 Potente pero es un motor nuevo (curva); útil solo si estructuras el estado en tablas. Para `.md/.txt`, git normal encaja mejor |
| [msiemens/tinydb](https://github.com/msiemens/tinydb) | 7.5k | 2026-05-28 | MIT | BD documental sobre **1 fichero JSON**, sin servidor | 🟡 Opción ligera para KV/doc; ojo con concurrencia alta (no es su fuerte) |
| [simonw/sqlite-utils](https://github.com/simonw/sqlite-utils) | 2.1k | 2026-07-04 | Apache-2.0 | CLI + lib para SQLite (WAL da concurrencia real) | ✅ Complemento si migras el estado a SQLite (la vía más robusta para P1) |
| [gitwatch/gitwatch](https://github.com/gitwatch/gitwatch) | 1.7k | 2026-06-11 | GPL-3.0 | Auto-commit al cambiar un fichero/carpeta | 🟡 Usa `inotify` (Linux/mac) → en **Windows necesita WSL**; si no, una tarea programada con `git commit` hace lo mismo |
| [iterative/dvc](https://github.com/iterative/dvc) | 15.7k | 2026-07-04 | Apache-2.0 | Versionado de datos/experimentos sobre git | 🔴 Orientado a datasets ML grandes; **overkill** para `.md/.txt`. Aprender, no adoptar |
| **Backup** [kopia/kopia](https://github.com/kopia/kopia) | 13.6k | 2026-07-02 | Apache-2.0 | Backup **Windows nativo + GUI**, incremental, cifrado, retención | ✅ **MEJOR ENCAJE P2** — snapshots fuera del PC + poda robusta (arregla el EPERM de OneDrive) |
| **Backup** [restic/restic](https://github.com/restic/restic) | 34.8k | 2026-07-01 | BSD-2 | Backup rápido/cifrado, CLI pura | ✅ Alternativa a Kopia si prefieres CLI sin GUI |
| [borgbackup/borg](https://github.com/borgbackup/borg) | 13.5k | 2026-07-04 | (BSD) | Archivador dedup + compresión + cifrado | 🟡 Excelente pero soporte Windows más flojo (pensado Linux/mac) |
| ~~piskvorky/sqlitedict~~ | 1.2k | **2022-12-07** | Apache-2.0 | dict persistente sobre SQLite | 🔴 **DESCARTAR** — último push hace ~4 años (abandonado) |

**Recomendación P1/P2 (mínimo esfuerzo, máximo blindaje):** `filelock` (locks ya) + migrar el estado "caliente"
a **git** (repo privado) con commit por tarea programada + **Kopia** para snapshots externos. Si algún día quieres
consultas/concurrencia de verdad, SQLite (WAL) + `sqlite-utils` + `litestream`. Dolt solo si el estado se vuelve tablas.

### 🔴 P3-BIS — Disparar el loop desde la nube (sin depender del PC)

| Repo / primitivo | ⭐ | Últ. push | Lic. | Qué aporta | Veredicto |
|---|---|---|---|---|---|
| **GitHub Actions `schedule:`** (nativo, sin repo) | — | — | — | **Cron en la nube GRATIS** para repos: corre sin el PC y commitea al repo | ✅ **LA RESPUESTA PRINCIPAL** — es el patrón "git scraping" de Simon Willison. Requiere que el estado esté en git (une con P1/P2) |
| [stefanzweifel/git-auto-commit-action](https://github.com/stefanzweifel/git-auto-commit-action) | 2.6k | 2026-06-28 | MIT | Que el job de Actions **commitee** los resultados | ✅ Adoptar junto con Actions (el "guardar" del loop en la nube) |
| [dagucloud/dagu](https://github.com/dagucloud/dagu) | 3.6k | 2026-07-04 | GPL-3.0 | Motor de workflows **local-first**, DAGs en **YAML**, binario único, **sin BD**, "usa cualquier agente IA para gestionar los DAGs" | ✅ **FUERTE** — encaja no-code/ficheros; corre en Windows (Go). Resuelve además P8 (ejecutor/DAG con reintentos) |
| [nektos/act](https://github.com/nektos/act) | 70.9k | 2026-07-03 | MIT | Correr GitHub Actions **localmente** para probar antes de subir | ✅ Útil dev/test, no producción |
| [mcuadros/ofelia](https://github.com/mcuadros/ofelia) | 3.9k | 2026-06-26 | MIT | Cron para **contenedores Docker** | 🔴 Solo si dockerizas (MADRE no usa Docker hoy) |
| [aptible/supercronic](https://github.com/aptible/supercronic) | 2.6k | 2026-07-03 | MIT | Cron para contenedores | 🔴 Igual que Ofelia: requiere contenedor |
| [windmill-labs/windmill](https://github.com/windmill-labs/windmill) | 17k | 2026-07-04 | (fuente abierta) | Plataforma de workflows/UIs potente | 🔴 **Servidor + BD 24/7** → overkill para 1 PC. Aprender, no adoptar |

**Recomendación P3-BIS:** el camino barato y correcto = **estado en git + GitHub Actions con `schedule:`** (gratis,
corre sin el PC). Si quieres un orquestador local con estado/reintentos/watchdog que también sirva a P8, **dagu** es
el mejor candidato file-based. Los cron-de-Docker se descartan salvo que MADRE pivote a contenedores.

### 🟠 P3 + P5 — Actuador externo y captura de demanda (bonus)

| Repo | ⭐ | Últ. push | Lic. | Qué aporta | Veredicto |
|---|---|---|---|---|---|
| [cli/cli](https://github.com/cli/cli) (`gh`) | 45.1k | 2026-07-03 | MIT | Actuar sobre GitHub (issues, PRs, releases) headless | ✅ **ADOPTAR** para el actuador "abrir PR/issue autónomo" (ya lo tienes instalado y logueado) |
| [PyGithub/PyGithub](https://github.com/PyGithub/PyGithub) | 7.7k | 2026-06-16 | LGPL-3.0 | Igual desde Python (tipado) | ✅ Alternativa programática al `gh` |
| [microsoft/playwright-python](https://github.com/microsoft/playwright-python) | 14.8k | 2026-07-02 | Apache-2.0 | **Sesión autenticada persistente** (`storage_state`) | ✅ Ya en tu cola: actuador web logueado headless (falsador de L1/L3) |
| [inovector/mixpost](https://github.com/inovector/mixpost) | 3.4k | 2026-03-16 | MIT | Publicar/programar en redes, self-host, sin suscripción (alt. Buffer) | 🟡 Actuador de distribución ligero (MIT); pega Laravel/PHP + servidor |
| [gitroomhq/postiz-app](https://github.com/gitroomhq/postiz-app) | 32.7k | 2026-07-03 | AGPL-3.0 | Scheduler social "agentic" | 🟡 Potente pero self-host = servidor; MADRE ya lo marcó "táctico-solo-Tony" — verificar encaje real antes de invertir |

**Nota P5 (captura de demanda):** MADRE **ya buscó** esto (`MODO_BUSQUEDA.md`) y concluyó que **no hay OSS
llave-en-mano**; la vía verificada es **GitHub Search API (`sort:reactions`) + Algolia HN API** (gratis) + un
clustering propio. No hay repo que adoptar aquí, solo primitivos ya identificados → no re-buscar.

---

## 9. NOTA DE HONESTIDAD (separar dolor real de ruido)

- **Dolor REAL y estructural (ataca el cuello o la integridad del sistema):** P1 (colisión/pérdida), P2 (sin backup
  externo / sin git), P3 (sin actuador), P3-BIS (loop atado al PC, sin nube), P5 (sin captura de demanda). Estos tocan
  la restricción activa (activación/captura de valor) o la integridad/autonomía del estado. **Nota:** P1+P2+P3-BIS
  podrían resolverse en gran parte con **un solo movimiento: migrar el estado a git + disparar el loop desde la nube.**
- **Dolor REAL pero local (mejora operativa, no mueve el cuello):** P4, P6, P7, P8, P9. Duelen y valen, pero
  resolverlos no genera beneficio externo por sí solo — son "quitar fricción".
- **Cuidado con el punto ciego PC4:** que un repo "toque muchos ficheros" o "sea elegante" no es progreso si no
  mueve el mundo. El filtro final: *¿este repo acerca a MADRE a cerrar valor con un tercero, o solo añade máquina
  interna?* Prioriza los primeros.
- **Sugerencias de repos = HIPÓTESIS sin verificar.** Toda recomendación (mía o de la IA a la que pegues esto) hay
  que verificarla contra el repo real (existe, licencia, mantenimiento, encaja en 1 PC sin servidor) antes de adoptar.
```
