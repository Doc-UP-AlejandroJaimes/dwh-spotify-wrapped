# Examen Final — Mi Spotify Wrapped

**Universidad de Pamplona**
**Profesor:** Juan Alejandro Carrillo Jaimes
**Materia:** Bases de Datos II
**Periodo académico:** 2026-I
**Fecha de entrega:** Viernes 22 de mayo de 2026, 11:59 p.m.
**Modalidad:** Trabajo en parejas

---

## Descripción del proyecto

Durante las últimas semanas construyeron un pipeline ETL completo usando sus propias cuentas de Spotify como fuente de datos. El resultado es un mini Data Warehouse personal en PostgreSQL con un backend en FastAPI, un frontend en React o Next.js, y una capa de análisis exploratorio sobre los datos reales cargados.

Este examen evalúa la implementación completa del sistema, la capacidad de análisis sobre los propios datos y la comprensión de las decisiones técnicas tomadas durante el desarrollo.

---

## Instrucciones generales

1. El trabajo es en parejas. No se aceptan entregas individuales ni grupos de tres.
2. Cada integrante de la pareja debe poder defender y explicar cualquier parte del proyecto en una sustentación oral si el profesor lo solicita.
3. Ambos integrantes deben mostrar sus modelos de diseño generados con IA (mockups, diagramas de arquitectura o wireframes). No se acepta un solo modelo compartido.
4. Toda la entrega se hace a través del repositorio de GitHub del proyecto. El link al repositorio se entrega por el medio indicado por el profesor.
5. El repositorio debe tener commits que reflejen el avance real del proyecto. Un repositorio con un solo commit con todo el código no será aceptado.
6. El archivo `docs/` debe estar completo con la documentación de cada fase, incluyendo screenshots, prompts utilizados y técnicas de IA aplicadas según la estructura definida en el README.
7. Los datos del EDA deben ser datos reales cargados desde su propia cuenta de Spotify. No se acepta datos sintéticos ni copiados.
8. La carpeta `.env` y cualquier archivo con credenciales no debe estar en el repositorio.

---

## Distribución del trabajo

La pareja divide la implementación técnica entre los dos integrantes:

| Integrante A | Integrante B |
|---|---|
| Backend (FastAPI, ETL, base de datos) | Frontend (React o Next.js, diseño con IA) |
| Secciones 1, 2 y 4 del examen | Secciones 3 y 4 del examen |
| Ambos responden la Sección 5 (preguntas técnicas) de forma individual |

> La Sección 4 (EDA) es responsabilidad compartida pero cada integrante debe haber ejecutado el notebook y poder explicar los hallazgos.

---

## Sección 1 — Backend (Integrante A)

### 1.1 Implementación del pipeline ETL

El backend debe tener implementadas las tres fases del ETL para los cuatro endpoints obligatorios de Spotify:

- `GET /v1/me` → `dim_users`
- `GET /v1/me/top/artists` → `dim_artists`
- `GET /v1/me/top/tracks` → `dim_tracks`
- `GET /v1/me/player/recently-played` → `fact_listening_history`

Cada fase (`extract_*`, `transform_*`, `load_*`) debe estar separada en funciones con docstrings completos según la Regla 3 del documento de definiciones.

### 1.2 Carga incremental

El ETL debe usar el cursor `cursor_next_ms` de la tabla `etl_audit` para hacer cargas incrementales. No puede volver a cargar registros que ya existen en `fact_listening_history`.

### 1.3 Registro de auditoría

Cada ejecución del ETL debe quedar registrada en `etl_audit` con: tiempo de inicio, tiempo de fin, duración en ms, registros nuevos e insertados por tabla, y el cursor para la próxima ejecución.

### 1.4 Endpoints de la API

Todos los endpoints listados en la Regla 7 deben estar implementados y protegidos con el JWT de la aplicación:

```
GET  /v1/profile/me
GET  /v1/artists/top
GET  /v1/tracks/top
GET  /v1/history/recently-played
POST /v1/etl/run
GET  /v1/etl/status
```

### 1.5 Migraciones

El schema completo del DWH debe estar versionado en Alembic. El comando `alembic upgrade head` ejecutado sobre una base de datos vacía debe crear todas las tablas correctamente.

---

## Sección 2 — Base de datos

### 2.1 DDL ejecutable

El repositorio debe incluir el script DDL completo en `docs/01-ddl-migrations.md`. Ejecutarlo desde cero en una base de datos limpia debe producir exactamente el mismo schema que Alembic.

### 2.2 Datos cargados

Evidencia de al menos tres ejecuciones exitosas del ETL en días distintos, visible en la tabla `etl_audit`. El `fact_listening_history` debe tener mínimo 100 registros para que el análisis del EDA sea significativo.

> Si la cuenta de Spotify tiene poca actividad reciente, la pareja debe ejecutar el ETL diariamente desde hoy y asegurarse de acumular suficientes datos antes de la entrega.

---

## Sección 3 — Frontend (Integrante B)

### 3.1 Páginas implementadas

El frontend debe tener las cinco rutas funcionales:

| Ruta | Requisito |
|---|---|
| `/login` | Botón "Conectar con Spotify", flujo PKCE completo sin copiar tokens |
| `/callback` | Recibe el JWT, lo guarda en `localStorage`, redirige al dashboard |
| `/dashboard` | Mínimo 4 widgets con datos reales del backend |
| `/profile` | Perfil completo del usuario con avatar y datos de Spotify |
| `/etl` | Estado del DWH, historial de ejecuciones, botón "Sincronizar" con log |

### 3.2 Integración real con el backend

Los datos del dashboard deben venir del backend, no directamente de la Spotify API. Todas las rutas protegidas deben enviar el `Bearer <token>` automáticamente.

### 3.3 Estado vacío

La aplicación debe manejar el caso en que las tablas del DWH están vacías (antes del primer ETL) con un mensaje claro que indique al usuario que debe sincronizar sus datos.

### 3.4 Diseño generado con IA

Entregar el archivo de diseño (Figma, Stitch, Uizard u otra herramienta) con las vistas de las 5 páginas. Cada vista debe tener un párrafo escrito por el estudiante justificando las decisiones de UX tomadas.

Incluir en `docs/03-frontend-implementation.md`:
- Herramienta de IA utilizada para el diseño
- Prompt exacto utilizado para generar los mockups
- Capturas de pantalla del diseño y del resultado implementado

---

## Sección 4 — Análisis Exploratorio de Datos (EDA)

El EDA se implementa en un **Google Colab o Jupyter Notebook** conectado directamente a la base de datos Neon. Los análisis deben hacerse sobre los datos reales de la cuenta de Spotify del estudiante.

El notebook debe entregarse en la carpeta `notebooks/` del repositorio con el nombre `eda_spotify_[nombre1]_[nombre2].ipynb`.

### 4.1 Carga y exploración inicial

- Conectar al DWH usando `psycopg2` o `SQLAlchemy` con las credenciales del `.env`.
- Cargar las tablas relevantes en DataFrames de pandas.
- Mostrar para cada tabla: número de filas, tipos de datos, porcentaje de valores nulos por columna.
- Identificar y documentar cualquier anomalía encontrada en los datos.

### 4.2 Análisis temporal

- ¿A qué horas del día escuchan más música? Gráfico de barras con conteo por hora.
- ¿Cuáles son sus días de semana con más reproducciones? Gráfico de barras ordenado.
- Construir un **heatmap de actividad** (eje X: hora del día, eje Y: día de semana, color: número de reproducciones). Este gráfico revela el patrón de consumo musical completo de la persona.
- Interpretación escrita: ¿qué dice este patrón sobre sus hábitos? ¿Hay momentos del día claramente asociados a ciertos contextos (trabajo, estudio, transporte)?

### 4.3 Análisis de artistas y géneros

- Top 10 artistas más escuchados en el historial reciente (con conteo de reproducciones).
- Explotar el array `genres` de `dim_artists` con `UNNEST` en SQL o `.explode()` en pandas. Graficar los 15 géneros más frecuentes.
- ¿Qué tan concentrada está su escucha? ¿El 80% de las reproducciones corresponde al 20% de los artistas? (principio de Pareto). Construir una curva de concentración o un gráfico de participación acumulada.
- Interpretación escrita: ¿son oyentes variados o muy enfocados en ciertos artistas?

### 4.4 Análisis de popularidad

- Distribución de popularidad de sus top tracks (histograma).
- Calcular: promedio, mediana, mínimo y máximo de popularidad.
- Clasificar sus canciones en tiers: `underground` (<30), `emerging` (30–60), `mainstream` (60–80), `viral` (>80). Graficar la distribución de tiers con un pie chart o bar chart.
- Interpretación escrita: ¿escuchan música popular o underground? ¿Qué dice eso de su perfil como oyente?

### 4.5 Análisis de duración

- ¿Cuánto duran en promedio las canciones que escuchan? Distribución de `duration_ms` convertida a minutos.
- ¿Hay diferencia en la duración promedio según el género musical dominante del artista?
- Construir un boxplot de duración agrupado por los 5 géneros más frecuentes.

### 4.6 Pregunta propia

Cada integrante de la pareja debe formular **una pregunta original** que no esté en las secciones anteriores y responderla con datos del DWH. La pregunta debe:

- No poder responderse con un simple `SELECT *`.
- Requerir al menos un `JOIN` o una agregación.
- Tener al menos una visualización asociada.
- Tener una interpretación escrita de mínimo 3 oraciones.

Ejemplos de preguntas válidas (no usar estas, formular las propias):
- ¿Existe correlación entre la popularidad de un artista y la frecuencia con la que aparece en el historial?
- ¿Las canciones escuchadas en contexto `playlist` son más o menos populares que las escuchadas en contexto `album`?
- ¿Cuál es el artista que más varía en términos de géneros dentro de su catálogo?

### 4.7 Conclusiones generales

Un párrafo de mínimo 150 palabras por integrante respondiendo: ¿qué aprendieron de sus propios datos que no sabían antes? ¿Qué limitación del modelo de datos les impidió responder alguna pregunta que querían hacer? ¿Qué tabla o columna adicional agregarían al DWH para enriquecer el análisis?

---

## Sección 5 — Preguntas técnicas (individual)

Cada integrante responde estas preguntas de forma individual en un archivo `docs/technical_answers_[nombre].md`. Las respuestas deben ser propias — respuestas idénticas entre integrantes de la misma pareja se calificarán con cero.

**Pregunta 1**
¿Cuál es la granularidad de la tabla `fact_listening_history`? ¿Qué representa exactamente una fila? ¿Por qué `played_at` no puede ser clave primaria por sí sola y cómo se resolvió ese problema en el modelo?

**Pregunta 2**
El ETL usa `ON CONFLICT (spotify_id) DO NOTHING` para las dimensiones y `ON CONFLICT (user_id, played_at) DO NOTHING` para la tabla de hechos. ¿Qué propiedad garantiza esto? ¿Qué pasaría si no existiera esa cláusula y el ETL se ejecutara dos veces el mismo día?

**Pregunta 3**
`dim_tracks` tiene una FK hacia `dim_artists`. ¿Qué tipo de schema produce esa relación entre dimensiones? ¿Cuál sería la alternativa en un star schema puro? ¿Por qué se decidió mantener la FK en este modelo y cuál es el trade-off?

**Pregunta 4**
Expliquen el flujo completo de autenticación desde que el usuario hace clic en "Conectar con Spotify" hasta que el frontend tiene el JWT guardado en `localStorage`. ¿Qué es PKCE y por qué se usa en lugar del flujo de autorización estándar?

**Pregunta 5**
¿Para qué sirve el campo `cursor_next_ms` en la tabla `etl_audit`? ¿Qué problema resuelve en el contexto de la carga incremental del historial de reproducciones? ¿Qué pasaría si un estudiante escucha 80 canciones en un día sin ejecutar el ETL?

---

## Rúbrica de evaluación

| Sección | Criterio | Porcentaje |
|---|---|---|
| **Backend** | ETL funcional con las 3 fases, endpoints implementados, migraciones Alembic | 25% |
| **Frontend** | 5 páginas funcionales, integración real con backend, diseño con IA documentado | 20% |
| **EDA** | Análisis completo, visualizaciones con interpretación escrita, pregunta propia, conclusiones | 30% |
| **Preguntas técnicas** | Respuestas individuales, correctas y con argumentación propia | 15% |
| **Documentación** | `docs/` completo con screenshots, prompts y técnicas de IA por fase | 10% |

### Criterios de penalización

- Repositorio sin historial de commits que refleje avance real: **−20 puntos sobre 100**
- Datos del EDA no corresponden a una cuenta real de Spotify: **nota de cero en Sección 4**
- Respuestas idénticas en Sección 5 entre integrantes de la misma pareja: **nota de cero en Sección 5 para ambos**
- Credenciales (`.env`, tokens, client secret) expuestos en el repositorio: **nota de cero en la entrega completa**

---

## Entrega

| Ítem | Ubicación en el repositorio |
|---|---|
| Código backend | `backend/` |
| Código frontend | `frontend/` |
| Notebook EDA | `notebooks/eda_spotify_[nombre1]_[nombre2].ipynb` |
| Documentación por fase | `docs/00-initial-config.md` … `docs/05-analytical-queries.md` |
| Respuestas técnicas | `docs/technical_answers_[nombre].md` (uno por integrante) |
| Diseño con IA | `docs/03-frontend-implementation.md` + archivo Figma o capturas |

El link al repositorio se entrega por el medio indicado antes del **viernes 22 de mayo de 2026 a las 11:59 p.m.** No se aceptan entregas tardías.
