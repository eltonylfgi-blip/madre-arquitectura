# MADRE — Puntos de dolor de un sistema personal de automatización (busco feedback + repos)

> **Qué es MADRE (a grandes rasgos):** un sistema personal que corre en **un solo ordenador** y cuyo objetivo es
> **mejorar de forma continua y producir valor con la mínima intervención humana**. Absorbe conocimiento, lo
> destila, acumula criterio y persigue generar resultados de forma autónoma.
>
> *Este documento comparte el **problema** y **qué busco**, no la implementación interna.*
>
> **💬 Para qué lo publico:** busco **feedback y recomendaciones de repositorios open-source**. Si conoces una
> forma mejor de resolver alguno de estos puntos de dolor, o un repo que encaje, **abre un Issue o una Discussion**.
> Me interesan sobre todo los 3 dolores rojos: integridad del estado en ficheros, versionado/backup externo, y
> ejecutar el proceso autónomo sin depender de que el PC esté encendido. ¡Gracias!

---

## El problema de fondo (el "cuello")

MADRE **genera mucho** (análisis, resúmenes, ideas, pequeños activos) pero le cuesta lo de fuera: **activar y
capturar valor real con terceros** (usuarios, señal, ingresos). En una frase: *el cuello no es producir hacia
dentro, es cerrar el bucle hacia afuera.* Casi todo lo que sigue son fricciones que o tocan ese cuello, o restan
tiempo/robustez para llegar a él.

Restricciones estructurales (a nivel concepto):
- **Sin actuador externo autónomo:** alcanza la red (lee, consulta APIs) pero no cierra valor con un tercero
  (publicar, vender, cerrar un trato) sin un humano en cada ciclo.
- **Activo aún demasiado "commodity":** lo que produce es fácilmente sustituible, así que no mueve a un selector
  externo (un comprador, una estrella).
- **Construir y no publicar:** tendencia a pulir cosas y no sacarlas al mundo.

---

## Puntos de dolor priorizados (+ cómo buscar solución)

> Formato: problema · por qué duele · qué tipo de solución busco · 🔎 frase de búsqueda para GitHub.
> Restricción transversal: **1 solo PC, basado en ficheros, sin servidor 24/7, gratis/open-source.**

### 🔴 P1 — Integridad del estado cuando todo son ficheros
- **Problema:** el "estado" del sistema son ficheros en una carpeta sincronizada en la nube (tipo OneDrive/Drive),
  y a veces escriben varios procesos/agentes casi a la vez.
- **Por qué duele:** las escrituras pueden pisarse o perderse, y la sincronización a veces sirve copias viejas o
  truncadas. La integridad depende de disciplina, no de garantías.
- **Qué busco:** una capa de **persistencia concurrente-segura** o versionado que haga imposible perder una
  escritura, **sin montar un servidor** (algo embebido, basado en ficheros).
- **🔎 GitHub:** `embedded single-file database concurrent append-only Python` · `cross-platform file lock library python`

### 🔴 P2 — Sin copia de seguridad externa ni versionado real
- **Problema:** el sistema vive en una sola máquina; el "versionado" es el historial limitado de la nube y no hay
  respaldo externo versionado. Hoy **no usa git**.
- **Por qué duele:** riesgo de pérdida total, y no se puede volver a un instante exacto del pasado.
- **Qué busco:** **backup incremental cifrado a un remoto** + versionado tipo git, con retención robusta que
  funcione en discos sincronizados.
- **🔎 GitHub:** `automated incremental backup encrypted retention windows` · `git version control for data`

### 🔴 P3-BIS — El proceso que "aprende" solo corre si el ordenador está encendido
- **Problema:** el proceso autónomo de mejora depende de que un equipo/app concreto esté abierto. Si se apaga, se para.
- **Por qué duele:** contradice el objetivo (autonomía): el sistema no mejora si la máquina no está encendida.
- **Qué busco:** disparar ese proceso **desde la nube** (cron/CI gratuito) o un runner que no dependa del PC.
- **🔎 GitHub:** `github actions scheduled agent commit to repo cron` · `run agent on schedule serverless cheap`

### 🟠 P3 — Sin actuador externo que cierre valor solo
- **Problema:** ninguna rutina automática puede publicar / vender / cerrar con un tercero sin un humano por ciclo.
- **Por qué duele:** es el techo del proyecto (el "cuello" de arriba).
- **Qué busco:** actuadores headless fiables — sesión autenticada persistente, publicación/cross-posting, o actuar
  sobre repos de forma programática.
- **🔎 GitHub:** `headless authenticated session persistence cookies` · `self-hosted social media cross-poster`

### 🟠 P5 — No sabe qué pide el mundo (captura de demanda)
- **Problema:** produce mucho hacia dentro, pero sin una señal de **demanda externa real** que oriente qué construir.
- **Por qué duele:** construye cosas que nadie pidió.
- **Qué busco:** minería/agregación de "pain points" reales (foros, issues) con clustering para priorizar.
- **🔎 GitHub:** `reddit hackernews pain point mining demand discovery` · `aggregate user feature requests rank`

### 🟡 Otros (operativos — quitan fricción, no mueven el cuello)
- **Dependencia de la cuota de un servicio cloud** para una capacidad concreta (visión/descripción de imágenes) →
  busco una alternativa **local/gratis**. 🔎 `local vision language model CPU offline image captioning`
- **Extractores/lectores de terceros frágiles** (se caen y una fuente deja de entrar en silencio) → busco
  librerías mantenidas con **múltiples backends**. 🔎 `maintained multi-backend downloader extractor python`
- **Backups/retención que se saturan y podan mal** en discos sincronizados. 🔎 `backup rotation prune retention safe`
- **Latencia por cadencia fija** (reacciona por horas, no por evento). 🔎 `file watcher trigger on change python`

---

## Criterios para juzgar un repo candidato

Un repo entra solo si cumple la mayoría de:
1. **Gratis / open-source** (licencia permisiva ideal; AGPL/GPL vale para uso propio).
2. **Mantenido activamente** (commits recientes, issues respondidas).
3. **Rápido / sin esperas largas** (preferible local/CPU; nada de minutos por ítem).
4. **Fácil de integrar en un sistema basado en ficheros** (Python puro o CLI; **sin servidor 24/7**, sin Docker
   orquestado obligatorio, sin base de datos gestionada externa).
5. **No reinventa** lo que ya hay; aporta una capacidad nueva o claramente mejor (más rápido, gratis en vez de pago,
   más robusto).
6. **Reversible / bajo riesgo** (se puede quitar sin romper nada; no intercepta el flujo como proxy).

---

## Repos candidatos ya verificados (búsqueda con la API de GitHub, 2026-07-04)

> ⭐estrellas · último push (mantenimiento) · licencia — comprobados ese día.

### P1 + P2 — Integridad del estado, versionado y backup
| Repo | ⭐ | Últ. push | Lic. | Para qué |
|---|---|---|---|---|
| [tox-dev/filelock](https://github.com/tox-dev/filelock) | 961 | 2026-07-03 | MIT | Lock de fichero multiplataforma (Python) |
| [benbjohnson/litestream](https://github.com/benbjohnson/litestream) | 13.8k | 2026-07-02 | Apache-2.0 | Replicación streaming de SQLite a un remoto |
| [simonw/sqlite-utils](https://github.com/simonw/sqlite-utils) | 2.1k | 2026-07-04 | Apache-2.0 | CLI/lib para SQLite (WAL = concurrencia real) |
| [dolthub/dolt](https://github.com/dolthub/dolt) | 23.8k | 2026-07-04 | Apache-2.0 | "Git for Data": BD SQL con commit/branch/diff |
| [msiemens/tinydb](https://github.com/msiemens/tinydb) | 7.5k | 2026-05-28 | MIT | BD documental sobre 1 fichero JSON |
| [kopia/kopia](https://github.com/kopia/kopia) | 13.6k | 2026-07-02 | Apache-2.0 | **Backup Windows nativo + GUI**, incremental, cifrado |
| [restic/restic](https://github.com/restic/restic) | 34.8k | 2026-07-01 | BSD-2 | Backup rápido/cifrado (CLI) |
| [gitwatch/gitwatch](https://github.com/gitwatch/gitwatch) | 1.7k | 2026-06-11 | GPL-3.0 | Auto-commit al cambiar ficheros (usa inotify → Windows via WSL) |

### P3-BIS — Ejecutar el proceso autónomo desde la nube
| Repo / primitivo | ⭐ | Últ. push | Lic. | Para qué |
|---|---|---|---|---|
| **GitHub Actions `schedule:`** (nativo) | — | — | — | Cron en la nube gratis: corre sin el PC y commitea al repo (patrón "git scraping") |
| [stefanzweifel/git-auto-commit-action](https://github.com/stefanzweifel/git-auto-commit-action) | 2.6k | 2026-06-28 | MIT | Que el job de Actions commitee resultados |
| [dagucloud/dagu](https://github.com/dagucloud/dagu) | 3.6k | 2026-07-04 | GPL-3.0 | Orquestador local-first, DAGs en YAML, binario único, sin BD |
| [nektos/act](https://github.com/nektos/act) | 70.9k | 2026-07-03 | MIT | Probar GitHub Actions localmente |

### P3 + P5 — Actuador externo y captura de demanda
| Repo | ⭐ | Últ. push | Lic. | Para qué |
|---|---|---|---|---|
| [cli/cli](https://github.com/cli/cli) | 45.1k | 2026-07-03 | MIT | Actuar sobre GitHub (issues/PRs) headless |
| [PyGithub/PyGithub](https://github.com/PyGithub/PyGithub) | 7.7k | 2026-06-16 | LGPL-3.0 | Igual desde Python |
| [microsoft/playwright-python](https://github.com/microsoft/playwright-python) | 14.8k | 2026-07-02 | Apache-2.0 | Sesión autenticada persistente (cookies) |
| [inovector/mixpost](https://github.com/inovector/mixpost) | 3.4k | 2026-03-16 | MIT | Publicar/programar en redes, self-host |

*Para captura de demanda (P5) no encontré un OSS llave-en-mano; la vía práctica son primitivos gratis
(GitHub Search API + Algolia HN) + clustering propio. Sugerencias bienvenidas.*

---

## Cómo ayudar

Abre un **Issue** o una **Discussion** con: (a) a qué punto de dolor responde, (b) el repo/enfoque que propones,
(c) por qué encaja (¿gratis? ¿mantenido? ¿sin servidor?), y (d) una pega honesta. Toda recomendación se
verificará antes de adoptarla. ¡Gracias por mirar!
