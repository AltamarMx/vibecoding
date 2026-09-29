# PLAN.md — Presentación "Vibe coding para cálculos de energía: del archivo climático al reporte reproducible" (50 min)

Plan de trabajo para producir una exposición de **50 minutos**, sus screencasts y el repositorio plantilla que la sustenta. Se deriva del plan de seminario original (110 min) recortando actividades y video en vivo; el material completo queda como tarea en YouTube y en el repositorio.

## 0. Punto de partida (estado del repositorio al 2026-09-28)

| Elemento | Estado |
|---|---|
| Repositorio git | Inicializado, sin commits |
| `pyproject.toml` | `vibecoding` 0.1.0, Python ≥3.13, `pvlib>=0.16.1`, `matplotlib>=3.11.2` |
| `.python-version` | 3.13 |
| `uv.lock` | Existe (generado por uv 0.12.5) |
| Quarto | 1.9.37 instalado |
| `main.py`, `README.md` | Plantilla vacía de `uv init` |

Decisiones ya implícitas: uv como único gestor, pvlib 0.16.1 (versión estable actual) y Python 3.13.

## 1. Agenda de 50 minutos

| Bloque | Min | Contenido | Actividad |
|---|---|---|---|
| 0. Apertura | 3 | Sección "Entonces… ¿realizarías el análisis de un artículo o tesis con vibe coding?" | Mano alzada: ¿usaste IA para hacer una tarea o trabajo en este ultimo mes? |
| 1. ¿Qué es el vibe coding? | 5 | Karpathy (feb 2025) → ingeniería agéntica (Sequoia, abr 2026). Espectro autocompletado → chat → agente | — |
| 2. Por qué programar sigue importando | 10 | Tres evidencias (Anthropic 50 % vs 67 %; Veracode 56 %; reproducibilidad 68.3 %) y los seis errores silenciosos del dominio energético | **Un** "encuentra el error" (azimut) |
| 3. Demo 1: EDA de un EPW | 8 | Clip de ~4 min (capítulo 1c: heatmap, `coerce_year=2024`, GHI > 0 con Sol bajo el horizonte) + comentario | "Predice la salida": ¿qué imprime `df.index[0]`? |
| 4. Demo 2: app Shiny de capitales | 8 | Clip de ~4 min (capítulo 2a: app funcionando y error de azimut) + capturas de 2c (prueba tautológica → prueba física) | Votación: ¿qué capital tiene mayor irradiación anual en plano inclinado? |
| 5. Reproducibilidad aun con vibe coding | 10 | uv + lockfile, git y bitácora de IA, pruebas físicas con pytest, checksums, Quarto; `git diff v1-vibe v2-ingenieria` | Checklist en pantalla (se lleva impreso o por QR) |
| 6. Cierre | 4 | Tres ideas para llevarse, repo plantilla, lecturas | Pregunta de reflexión (30 s) |
| Colchón | 2 | Preguntas o fallas técnicas | — |

Reglas de tiempo: ningún clip supera 4 min; los videos completos (6 capítulos) se enlazan como tarea; las actividades son de respuesta rápida (mano alzada o votación), sin trabajo en parejas.

## 2. Decisiones de diseño

- **Formato:** Quarto revealjs en `slides/index.qmd`, sin ejecución de código (`execute: enabled: false`); figuras ya renderizadas desde `analysis/`.
- **Video en vivo:** solo dos clips de ~4 min (1c y 2a). El capítulo 2c se muestra como capturas (antes/después de la prueba). Los seis capítulos completos van en dos listas de YouTube (no listadas) como tarea.
- **Respaldo sin red:** MP4 locales en `slides/videos/` (ignorados en git) y capturas en `slides/img/` para cada momento clave.
- **Ciudad del video 1:** Hermosillo (UTC−7, sin horario de verano, alta irradiación).
- **App (video 2):** Shiny Express para el prototipo, Core para la versión refactorizada; despliegue en Posit Connect Cloud con datos PVGIS precalculados en Parquet. Enlace por QR en la última diapositiva.
- **Repositorio plantilla:** este mismo repo, renombrado a `vibe-energia`, con etiquetas `v1-vibe` y `v2-ingenieria`.
- **Idioma:** diapositivas y narración en español; citas y nombres de API en inglés tal cual.

## 3. Fases y entregables

### Fase A — Esqueleto del repositorio plantilla (1 sesión)

- [ ] Renombrar el proyecto a `vibe-energia` en `pyproject.toml`; borrar `main.py`; layout `src/energia/`.
- [ ] `uv add pandas plotly pyarrow shiny` y `uv add --dev pytest jupyter ruff`.
- [ ] Carpetas: `data/{raw,interim,processed}`, `src/energia`, `analysis`, `app`, `scripts`, `tests`, `outputs`, `slides/{img,videos}`, `docs`.
- [ ] `.gitignore`: `.venv/`, `outputs/`, `slides/videos/*.mp4`, `data/interim/`, `_site/`, `*_files/`.
- [ ] `AGENTS.md` (solo uv; `data/raw` inmutable; unidades en nombres; azimut 180 = sur; índice EPW = inicio de intervalo; toda función en `src/` lleva prueba; no añadir dependencias sin preguntar). `CLAUDE.md` que remita a `AGENTS.md`.
- [ ] `docs/bitacora-ia.md` con la plantilla de registro (fecha, herramienta/modelo, prompt, aceptado, corregido, por qué).
- [ ] `README.md` con "reproducir en 3 comandos": `uv sync --locked`, `uv run pytest`, `uv run quarto render analysis/01_eda_epw.qmd`.
- [ ] Primer commit: "chore: esqueleto del proyecto".

Verificación: `uv sync --locked && uv run pytest` corre y `uv run quarto --version` responde.

### Fase B — Código y datos de referencia (1-2 sesiones)

Objetivo: tener la versión "ingeniería" funcionando antes de grabar, para conocer de antemano el resultado correcto contra el que se contrasta cada error de la IA.

- [ ] EPW TMYx de Hermosillo (climate.onebuilding.org, variante 2007-2021) en `data/raw/`; `CHECKSUMS.sha256` y `FUENTES.md`.
- [ ] `src/energia/epw.py`: `cargar_epw(ruta, coerce_year=2023)`, `resumen_mensual_temp`, `grados_dia` (media diaria, base 18 °C), `matriz_hora_dia`.
- [ ] `src/energia/solar.py`: `posicion_solar_centrada` (+30 min), `cielo_claro`, `poa_mensual(lat, lon, tz, alt, tilt, az, modelo)`.
- [ ] `src/energia/capitales.csv` (32 capitales: lat, lon, alt, UTC estándar, fuente) y `capitales.py`.
- [ ] `scripts/descargar_pvgis.py`: `get_pvgis_tmy` → `data/interim/pvgis_tmy/*.parquet`, conversión UTC → local, reintentos ante 429/529.
- [ ] `tests/test_epw.py`, `tests/test_solar.py`, `tests/test_capitales.py`: 8760 filas, GHI ≥ 0, cero de noche, ≤ 1.2 × cielo claro, balance anual, cierre GHI ≈ DNI·cosθz + DHI, azimut 180 > 0 en diciembre, tilt 0 ⇒ POA = GHI, 32 filas con rangos válidos, checksums.
- [ ] `analysis/01_eda_epw.qmd`: metadatos y calidad, temperatura, radiación, heatmaps, celda final de procedencia (`pvlib.__version__`, hash del EPW, `git rev-parse --short HEAD`).
- [ ] `app/app.py` (Core) que solo llama a `src/energia/solar.py`; `uv export --format requirements-txt --no-hashes -o requirements.txt`.
- [ ] Etiquetar `v2-ingenieria`.

Verificación: `uv run pytest -q` en verde; el reporte renderiza con figuras; la app corre con `uv run shiny run --reload app/app.py`. Si Quarto toma otro Python, exportar `QUARTO_PYTHON=.venv/bin/python`.

### Fase C — Grabación de los screencasts (2-3 sesiones)

Prioridad de grabación: **1c y 2a primero** (son los que se proyectan), luego 2c, y al final 1a, 1b y 2b (solo para la lista de YouTube). Si el tiempo de producción escasea, 1a, 1b y 2b se pueden omitir sin afectar la charla.

Preparación común:
- [ ] Rama `v1-vibe` desde el esqueleto de la Fase A (sin `src/`, sin pruebas).
- [ ] OBS 1920×1080 @ 30 fps; fuente 20 pt / zoom 150-175 %; tema de alto contraste; Keycastr; una ventana a la vez; sin claves ni correos visibles.
- [ ] Micrófono de solapa o USB; cuarto sin eco.
- [ ] Prompts escritos de antemano y registrados en `docs/bitacora-ia.md` conforme se usan.

| Cap. | Se proyecta | Prompt clave | Error que se espera capturar | Plan si la IA no se equivoca |
|---|---|---|---|---|
| 1c | **Sí (clip 4 min)** | Heatmaps hora × día de temp_air, ghi, dni | Barra "W/m²"; `coerce_year=2024` (366 días); GHI > 0 con Sol bajo el horizonte | Captura estática en diapositiva |
| 2a | **Sí (clip 4 min)** | App Shiny Express para 32 capitales | `surface_azimuth=0` "sur"; Chihuahua UTC−7; signo `Etc/GMT` | Pedir "azimut 0 = sur como en PVsyst" |
| 2c | Capturas | Refactorizar a funciones puras + pytest | Prueba tautológica | Sustituir por prueba física |
| 1a | Tarea | Proyecto uv para EPW | Paquete inexistente / `pip install` | Prompt más ambiguo |
| 1b | Tarea | Lectura EPW + resumen temp + grados-día | `map_variables=True` → `TypeError`; grados-día en °C·h | Idem |
| 2b | Tarea | Descargar TMY PVGIS y cachear | Desempaquetar 4 valores; "PVGIS-NSRDB"; olvida UTC | Idem |

Post-producción:
- [ ] Recortar 1c y 2a a ≤ 4 min para la charla (versión "clip") y conservar la versión completa para YouTube.
- [ ] Acelerar ×4-×8 las esperas del modelo con letrero "espera real: N s".
- [ ] Anotaciones en cada error ("⚠ la IA supuso azimut 0 = sur").
- [ ] Subtítulos en español corregidos (vibe coding, pvlib, EPW, GHI) y .srt propio.
- [ ] Subir como "no listado"; capítulos en la descripción; enlace al repo + hash del commit + pvlib 0.16.1 + Python 3.13 + fecha.
- [ ] MP4 de los clips en `slides/videos/`; capturas de cada momento clave en `slides/img/`.

### Fase D — Diapositivas en Quarto (1-2 sesiones)

- [ ] `slides/_quarto.yml`: `format: revealjs`, `theme: [default, styles.scss]`, `slide-number: true`, `embed-resources: false`, `execute: enabled: false`, `lang: es`.
- [ ] `slides/styles.scss`: fuente de código ≥ 28 px.
- [ ] `slides/index.qmd` según el guion de la sección 4.
- [ ] Cada clip en su propia diapositiva, sin columnas, con `{{< video https://www.youtube-nocookie.com/embed/ID?start=S&end=E >}}` y una diapositiva gemela con `![](videos/clip.mp4)`.
- [x] Esqueleto creado: `slides/_quarto.yml`, `slides/index.qmd` (28 diapositivas + 2 gemelas locales, marcadores VIDEO/CAPTURA/QR/TODO), `slides/styles.scss`.
- [x] Render: `rm -rf slides/_site slides/index_files && uv run quarto render slides` (con Quarto 1.9.37 el render incremental a veces omite `index_files/libs/revealjs`; el render limpio lo evita). Preview con `uv run quarto preview slides`.
- [x] Imágenes del bloque 1: `img/karpathy.png` (Gladwin Analytics, 2019, Wikimedia Commons, CC BY 3.0) y `img/karpathy-tweet.png` (captura del embed oficial de X; el embed recorta el texto largo).
- [ ] **Publicación en GitHub Pages:** `uv run quarto publish gh-pages slides` renderiza `slides/` y sube `slides/_site` a la rama `gh-pages`. En el repo: Settings → Pages → Source: "Deploy from a branch" → `gh-pages` / `(root)`. URL resultante: `https://altamarmx.github.io/vibecoding/`.
- [ ] Sustituir cada marcador VIDEO por el shortcode de YouTube y la gemela por `![](videos/clip-XX.mp4)`; los MP4 no van a git ni a Pages (solo al USB de respaldo).

Verificación: renderiza sin red, la fuente se lee a 1024×768 y el conteo queda en ≈ 25 diapositivas de contenido (≈ 1.5-2 min por diapositiva).

### Fase E — Ensayo y logística (1 semana antes)

- [ ] Ensayo cronometrado completo; meta 46-48 min sin colchón.
- [ ] Probar en el equipo del aula el día anterior: YouTube, MP4 local, resolución del proyector, audio.
- [ ] Publicar el repo como "template" en GitHub con etiquetas `v1-vibe` y `v2-ingenieria`.
- [ ] Desplegar la app en Posit Connect Cloud y enlazarla por QR.
- [ ] Checklist de reproducibilidad como PDF de una página (QR en la diapositiva 22).

## 4. Guion de diapositivas (índice de `slides/index.qmd`, ≈ 30 diapositivas; la numeración de los bloques 2-6 se corre en +2)

**Bloque 1a — Karpathy, la definición y la comunidad (4 min), justo después del título**
1. Título, ponente, fecha, QR al repo.
2. ¿Quién es Andrej Karpathy? Stanford/CS231n (enlace a Fei-Fei Li), OpenAI, Tesla Autopilot, "Software 2.0", Zero to Hero, Eureka Labs.
3. El tuit del 2 feb 2025 (captura del embed) con las citas clave; "proyectos desechables".
4. ¿Qué es, entonces, el vibe coding? Solo la definición operativa en una frase.
5. ¿Qué anda haciendo la comunidad? Tres columnas: juguetes virales (MenuGen, fly.pieter.com), apps sin programadores (Lovable, Bolt, v0, Replit) y ciencia (encuesta Anthropic 2026: 20 % usa agentes de código; app Protección solar de Ener-Habitat en Shinylive, hecha con vibe coding).

**Bloque 0 — Entonces… ¿realizarías el análisis de un artículo o tesis con vibe coding? (3 min)**
6. Una pregunta: ¿usaron IA para algo de su trabajo esta semana?

**Bloque 0b — Sí, pero con estas consideraciones**
7. Repite la pregunta del separador y responde "Sí, pero con estas consideraciones:".
8. Ocho consideraciones (índice): entorno (uv), espacio de trabajo y estructura, plan inicial en Markdown, herramientas y reglas del espacio de trabajo (AGENTS.md: solo uv, no tocar `data/001_raw/`), ejecución parte por parte con Quarto + Python, revisar TODO, pedir ideas / retroalimentar / poner a prueba, recrear y revisar el prototipo (human in the loop).
9. Ejemplo aplicado (Torreón, Coahuila): explorar un EPW; caracterizar temperatura, radiación y viento; cuantificar el calor con el modelo adaptativo de ASHRAE 55 (grados-hora sobre el límite de 80 %); potencial solar mensual en superficie horizontal (kWh/m²·mes).
10-17. Un clip por consideración, todos sobre el ejemplo de Torreón (VIDEO, ≤ 3-4 min cada uno): uv · crear espacio de trabajo · plan en MD · reglas para el agente · EDA con Quarto y Python · exploración de código · validación de un caso · recrear desde cero, explicar y comparar con referencia externa.

**Cierre**
- Entonces… ¿realizarías el análisis de una tesis con vibe coding? Sí, si viene con A) entorno y datos bajo control, B) un plan que revisaste, C) una persona que revisó, probó y recreó.
- Para seguir: repo de Torreón, lista de los ocho videos, diapositivas, lecturas.
- Pregunta final: ¿qué parte de tu próximo análisis no delegarías? + contacto.

## 5. Calendario sugerido (relativo a la fecha del seminario, D)

| Semana | Fase | Hito |
|---|---|---|
| D−5 | A + B | Repo con `v2-ingenieria`, pruebas en verde, reporte renderizado |
| D−4 | C | Grabados 1c, 2a y 2c; clips recortados; bitácora al día |
| D−3 | C + D | Grabados 1a, 1b, 2b (si hay tiempo); diapositivas completas, render sin red |
| D−2 | E | Ensayo cronometrado; app desplegada; repo publicado como template |
| D−1 | E | Prueba en el aula; MP4 y capturas en el equipo de la charla |

## 6. Riesgos que afectan este plan

- Sin internet en el aula → MP4 locales + `embed-resources: false` + carpeta completa en USB.
- La IA no comete el error en la toma → prompt más ambiguo o captura estática; decirlo a la audiencia.
- PVGIS caído al precalcular → reintentos y descarga temprana (Fase B, no la víspera).
- Cambios de API antes del seminario → `uv.lock` fijo; versión indicada en cada video.
- Sobrepasar el tiempo → si a los 30 min no se ha llegado al bloque 5, saltar la diapositiva 20 y la 25; el bloque 5 es el mensaje central y no se recorta.

## 7. Decisiones abiertas (responder antes de la Fase C)

1. Fecha del seminario.
2. ¿Ciudad del video 1: Hermosillo (propuesta) o CDMX?
3. ¿La app se publica en Connect Cloud antes de la charla o solo se muestra en local?
4. ¿Herramienta de IA para grabar: Claude Code, Codex, Cursor u otra? Conviene una sola y nombrarla en la bitácora.
5. ¿Nombre final del repositorio: `vibe-energia` (propuesto) o mantener `vibecoding`?
