[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/taxuspt-garmin-mcp-badge.png)](https://mseep.ai/app/taxuspt-garmin-mcp)

# Servidor Garmin MCP

Este servidor Model Context Protocol (MCP) se conecta a Garmin Connect y expone tus datos de actividad física y salud a Claude y otros clientes compatibles con MCP.

El acceso a la API de Garmin se realiza mediante la excelente biblioteca [python-garminconnect](https://github.com/cyberjunky/python-garminconnect).

## Funciones

- Lista actividades recientes con soporte de paginación
- Obtiene información detallada de las actividades
- Edita actividades: nombre, tipo, descripción/notas, tipo de evento, esfuerzo percibido (RPE) y sensaciones
- Accede a métricas de salud (pasos, frecuencia cardíaca, sueño, estrés y respiración)
- Consulta datos de composición corporal
- Consulta el estado y la preparación para el entrenamiento
- Accede a métricas de FTP de ciclismo y umbral de lactato
- Gestiona material y equipamiento, incluidas las notas de texto libre devueltas por `get_gear`
- Accede a entrenamientos y planes de entrenamiento
- Inspecciona estructuras detalladas de los pasos de los entrenamientos, incluidos grupos de repeticiones y objetivos de ritmo de natación
- Agregados semanales de salud (pasos, estrés y minutos de intensidad)
- Análisis avanzado de ciclismo: zonas de potencia, análisis de archivos FIT e información sobre cambios electrónicos DI2
- Tendencias de carga de entrenamiento (CTL/ATL/TSB), HRV, VO2 máx. y frecuencia respiratoria
- Curva de duración de potencia, detección de subidas con VAM, deriva cardíaca (desacoplamiento aeróbico) y cálculos de W/kg

### Cobertura de herramientas

Este servidor MCP implementa **más de 110 herramientas** que cubren aproximadamente el 90 % de la biblioteca [python-garminconnect](https://github.com/cyberjunky/python-garminconnect) (v0.3.2):

- ✅ Gestión de actividades (20 herramientas) - incluye herramientas de escritura para tipo, descripción, tipo de evento, esfuerzo percibido y sensaciones
- ✅ Salud y bienestar (32 herramientas) - incluye herramientas personalizadas de resumen ligero
- ✅ Entrenamiento y rendimiento (13 herramientas) - incluye tendencias de CTL/ATL/TSB, HRV, VO2 máx. y respiración
- ✅ Entrenamientos (8 herramientas)
- ✅ Dispositivos (7 herramientas)
- ✅ Gestión de equipamiento (5 herramientas)
- ✅ Seguimiento del peso (5 herramientas)
- ✅ Retos e insignias (10 herramientas)
- ✅ Nutrición (9 herramientas) - registros de alimentos, comidas, alimentos personalizados, registro de alimentos y resúmenes de ingesta de varios días
- ✅ Salud femenina (3 herramientas)
- ✅ Perfil de usuario (3 herramientas)
- ✅ Constructores de entrenamientos de alto nivel (4 herramientas) - crean y programan entrenamientos sin escribir JSON
- ✅ Recorridos (5 herramientas) - listar / obtener detalles / subir GPX como recorrido / descargar GPX / eliminar recorrido
- ✅ Análisis de actividades (2 herramientas) - análisis de archivos FIT y curva de duración de potencia; requiere potenciómetro y/o Di2
- ✅ Descarga de archivos de actividad (2 herramientas) - descarga archivos de actividad en formato FIT, GPX, TCX o CSV

> **Nota:** Las herramientas de análisis de actividades requieren un potenciómetro compatible (por ejemplo, Garmin Rally, Favero Assioma o PowerTap P1) y/o cambio electrónico Shimano Di2 / SRAM eTap. La dependencia `fitparse` se instala automáticamente.

### Notas del equipamiento

Cada elemento del array `gear` devuelto por `get_gear` incluye un campo `notes`
que contiene el valor de texto libre de Notas mostrado en Garmin Connect. El equipamiento sin
un valor en Notas devuelve `null`; el resto de campos existentes del equipamiento no cambia.

### Descarga de archivos de actividad

Dos herramientas permiten descargar al disco el archivo original de una actividad:

- **`download_activity_file(activity_id, format="fit", output_dir=None)`** — descarga la actividad y la guarda en el directorio configurado. `format` acepta `fit` (por defecto), `gpx`, `tcx` o `csv`.
- **`set_fit_download_dir(path)`** — establece y guarda de forma persistente el directorio de descarga predeterminado (se escribe en el archivo de configuración).

**Dónde se guardan los archivos, por orden de prioridad:**

1. Argumento `output_dir` — sobrescritura puntual, no persistente.
2. Variable de entorno `GARMIN_FIT_DOWNLOAD_DIR`.
3. Configuración persistente establecida mediante `set_fit_download_dir`.

**Comportamiento en la primera ejecución:** si no hay ningún directorio configurado, `download_activity_file` devuelve `status: "needs_setup"`. El asistente preguntará dónde quieres guardar los archivos (sugiriendo el directorio actual como valor predeterminado), llamará a `set_fit_download_dir` para guardar tu elección y volverá a intentar la descarga automáticamente.

### Endpoints omitidos intencionadamente

Algunos endpoints no están implementados por motivos de rendimiento o complejidad:

**Gran volumen de datos:**
- `get_activity_details()` - Devuelve trazas GPS y datos de gráficos de gran tamaño (50 KB-500 KB). Para resúmenes, utiliza `get_activity()`.

**Formatos de entrenamiento especializados:**
- `upload_running_workout()`, `upload_cycling_workout()`, `upload_swimming_workout()` - Subidas de entrenamientos específicas por deporte. Utiliza `upload_workout()` para entrenamientos generales.

**Mantenimiento y operaciones destructivas:**
- `delete_activity()`, `delete_blood_pressure()` - Las operaciones destructivas requieren especial cuidado.
- Métodos internos/de autenticación: `login()`, `resume_login()`, `connectapi()`, `download()` - La biblioteca los gestiona automáticamente.

Si necesitas alguno de estos endpoints, [abre una incidencia](https://github.com/Taxuspt/garmin_mcp/issues).

## Filtrado de herramientas

Este servidor registra más de 110 herramientas de forma predeterminada, lo que puede suponer bastante contexto
para que un LLM lo mantenga en cada sesión. Puedes exponer únicamente las herramientas que necesites mediante
dos variables de entorno opcionales:

| Variable de entorno | Efecto |
|---|---|
| `GARMIN_ENABLED_TOOLS` | **Lista permitida** separada por comas — si se establece, *solo* se registran estas herramientas. |
| `GARMIN_DISABLED_TOOLS` | **Lista denegada** separada por comas — las herramientas indicadas se omiten. Se ignora si existe una lista permitida. |

Los nombres de las herramientas no distinguen entre mayúsculas y minúsculas. Si no se establece ninguna de las dos variables, se registran todas las herramientas
(comportamiento predeterminado sin cambios). Los nombres que no coincidan con ninguna herramienta se ignoran y se muestra una
advertencia en stderr, lo que facilita detectar errores tipográficos.

Ejemplo: exponer solo sueño, estrés y actividades recientes:

```json
"env": {
  "GARMIN_ENABLED_TOOLS": "get_sleep_data,get_stress_summary,get_activities"
}
```

## Herramientas de entrenamiento de alto nivel

Estas herramientas de construcción permiten que un LLM cree y programe entrenamientos sin escribir directamente el JSON de Garmin.

### `create_walk_run_workout`

Crea un entrenamiento por intervalos de caminar/correr con un objetivo opcional de zona de frecuencia cardíaca.

```json
{
  "name": "W3 Mié 2:2",
  "run_seconds": 120,
  "walk_seconds": 120,
  "repeats": 9,
  "warmup_min": 10,
  "cooldown_min": 8,
  "hr_zone": "Z3"
}
```

Devuelve: `{"status": "success", "workout_id": 1234567890, ...}`

### `create_z2_walk_workout`

Crea un entrenamiento continuo de caminar en Z2.

```json
{
  "name": "Z2 Walk 45m",
  "duration_min": 45,
  "hr_min": 110,
  "hr_max": 130
}
```

Devuelve: `{"status": "success", "workout_id": 1234567890, ...}`

### `create_strength_workout`

Crea un entrenamiento de fuerza a partir de una lista de ejercicios. Cada ejercicio se convierte en un paso basado en repeticiones y el
nombre se conserva en la descripción del paso. El nombre también se envía como `exerciseName`, pero Garmin solo
lo conserva cuando coincide con una de sus propias claves de ejercicios (por ejemplo, `FARMERS_CARRY`); cualquier otro
valor se acepta pero después se guarda vacío.

`category` es opcional y se envía directamente. Si se omite, la clave no se incluye en el
payload, algo que Garmin acepta. Si se proporciona, debe ser una de las categorías de ejercicios de Garmin;
cualquier otra, incluidas `OTHER` y `UNASSIGNED`, se rechaza con
`400 - Invalid category`. La lista completa está publicada en
[`Exercises.json`](https://connect.garmin.com/web-data/exercises/Exercises.json).

```json
{
  "name": "Full Body A",
  "exercises": [
    {"name": "Sentadillas", "sets": 3, "reps": 12, "rest_seconds": 90},
    {"name": "Flexiones",   "sets": 3, "reps": 15, "rest_seconds": 60},
    {"name": "Peso muerto", "sets": 3, "reps": 10, "rest_seconds": 90},
    {"name": "Farmers Carry 40m", "sets": 3, "reps": 1, "rest_seconds": 90, "category": "CARRY"}
  ]
}
```

Devuelve: `{"status": "success", "workout_id": 1234567890, ...}`

### `schedule_week`

Programa varios entrenamientos en una sola llamada.

```json
{
  "week": [
    {"date": "2026-05-12", "workout_id": 1234567890},
    {"date": "2026-05-14", "workout_id": 1234567891}
  ]
}
```

Devuelve: `{"status": "complete", "scheduled": [...]}`

### Ejemplo de flujo completo

```text
create_walk_run_workout(name="W3 Mié 2:2", run_seconds=120, walk_seconds=120,
                        repeats=9, warmup_min=10, cooldown_min=8)
  → workout_id = 1560092011

schedule_workout(workout_id=1560092011, date="2026-05-06")
  → OK
```

Después de sincronizar el reloj, el entrenamiento aparece en el calendario del Forerunner 965.

### Condiciones de finalización de `upload_workout` en formato raw

Al crear JSON de entrenamientos personalizados para `upload_workout` o `upload_workouts`,
`endCondition.conditionTypeId` y `endCondition.conditionTypeKey` deben coincidir
con el mapeo canónico de Garmin. Garmin trata el `conditionTypeId` numérico como
la fuente de verdad; si la clave y el ID entran en conflicto, Garmin guarda la condición que
corresponde al ID.

Por ejemplo, esto no es válido para una condición de finalización por frecuencia cardíaca porque el ID `4` corresponde a
`calories`, no a `heart.rate`:

```json
{
  "endCondition": {
    "conditionTypeId": 4,
    "conditionTypeKey": "heart.rate"
  },
  "endConditionValue": 145
}
```

Utiliza el ID `6` para la frecuencia cardíaca:

```json
{
  "endCondition": {
    "conditionTypeId": 6,
    "conditionTypeKey": "heart.rate"
  },
  "endConditionValue": 145
}
```

Para una condición de finalización basada en una zona de frecuencia cardíaca, utiliza `endConditionZone` (1-5) en el
mismo objeto `endCondition` y omite `endConditionValue`. Si se envían ambos,
Garmin conserva la zona y descarta el valor (comprobado contra la API de producción
el 2026-09-01):

```json
{
  "endCondition": {
    "conditionTypeId": 6,
    "conditionTypeKey": "heart.rate",
    "endConditionZone": 2
  }
}
```


IDs habituales de las condiciones de finalización:

| ID | Clave |
|---:|---|
| 1 | `lap.button` |
| 2 | `time` |
| 3 | `distance` |
| 4 | `calories` |
| 5 | `power` |
| 6 | `heart.rate` |
| 7 | `iterations` |
| 8 | `fixed.rest` |
| 9 | `fixed.repetition` |
| 10 | `reps` |
| 11 | `training.peaks.tss` |

### Tipos de objetivo de `upload_workout` en formato raw

Al crear JSON de entrenamientos Garmin en bruto, `targetType.workoutTargetTypeId` y
`targetType.workoutTargetTypeKey` deben usar el mapeo canónico de Garmin. Garmin
trata el ID numérico como autoritativo: un payload que no coincida, como
`{"workoutTargetTypeId": 6, "workoutTargetTypeKey": "heart.rate"}`, se guarda como
`pace.zone`, porque el ID `6` significa `pace.zone`.

Para un rango personalizado de frecuencia cardíaca, utiliza el tipo de objetivo con ID `4`, `heart.rate.zone`, e
incluye el rango de ppm en `targetValueOne` / `targetValueTwo`. Estos campos de valores
pertenecen al paso del entrenamiento, junto a `targetType`; no los anides dentro del
objeto `targetType`:

```json
{
  "targetType": {
    "workoutTargetTypeId": 4,
    "workoutTargetTypeKey": "heart.rate.zone"
  },
  "targetValueOne": 143,
  "targetValueTwo": 157
}
```

La misma estructura se aplica a un rango personalizado de ritmo de carrera. Los límites de ritmo usan metros
por segundo:

```json
{
  "targetType": {
    "workoutTargetTypeId": 6,
    "workoutTargetTypeKey": "pace.zone"
  },
  "targetValueOne": 1.9607843,
  "targetValueTwo": 2.0833333
}
```

Ese ejemplo representa `8:00–8:30 min/km`. El límite numérico inferior aparece
primero por coherencia con el ejemplo de frecuencia cardíaca; Garmin normaliza
cualquiera de los dos órdenes. Garmin descarta silenciosamente los valores anidados dentro de `targetType`, dejando
un objetivo de ritmo sin rango activo. Las herramientas de subida corrigen ese error de anidado cuando es inequívoco,
pero rechazan la solicitud si los valores anidados y los valores del paso entran en conflicto.

Para una zona de FC de Garmin con nombre, utiliza el mismo tipo de objetivo con `zoneNumber`:

```json
{
  "targetType": {
    "workoutTargetTypeId": 4,
    "workoutTargetTypeKey": "heart.rate.zone"
  },
  "zoneNumber": 3
}
```

Utiliza `zoneNumber` o `targetValueOne` / `targetValueTwo` en un objetivo, pero no
ambos. Garmin trata la zona con nombre como autoritativa y descarta silenciosamente un
rango personalizado coexistente, por lo que las herramientas de subida rechazan esa estructura ambigua.

## Instalación con un clic (Claude Desktop)

La forma más sencilla de añadir este servidor a Claude Desktop es mediante el archivo de extensión de escritorio `.dxt`, sin necesidad de editar JSON.

### Descargar e instalar

1. Descarga la última versión de `garmin-mcp.dxt` desde la [página de Releases](https://github.com/Taxuspt/garmin_mcp/releases).
2. Arrastra el archivo `.dxt` a la ventana de Claude Desktop, **o** haz doble clic en él, **o** ve a **Settings → Extensions → Install Extension** y selecciona el archivo.
3. Claude Desktop te pedirá la configuración opcional (ruta de tokens, correo electrónico y contraseña).

### Primera autenticación

La extensión instala y ejecuta el servidor automáticamente, pero debes autenticarte una vez con Garmin antes de poder obtener datos:

```bash
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth
```

Esto guarda los tokens OAuth en `~/.garminconnect`. A partir de ese momento, el servidor funciona sin credenciales en la configuración.

> **Nota:** Los tokens son válidos durante aproximadamente 6 meses. Vuelve a ejecutar `garmin-mcp-auth` cuando caduquen.

### Compilar tú mismo el `.dxt`

```bash
bash scripts/build_dxt.sh   # produces garmin-mcp.dxt in the repo root
```

---

## Configuración

### Inicio rápido para clientes MCP

La forma más sencilla de usar este servidor MCP con Claude Desktop, [Codex](https://openai.com/codex/) u otro cliente MCP es autenticarse una vez antes de añadir el servidor a la configuración.

#### Requisitos previos

- Python 3.12+
- Cuenta de Garmin Connect
- Puede ser necesario MFA si está activado en tu cuenta

#### Paso 1: Preautenticación, una sola vez

Antes de añadir el servidor a tu cliente MCP, autentícate una vez desde la terminal:

```bash

# Install and run authentication tool
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth

# You'll be prompted for:
# - Email (or set GARMIN_EMAIL env var)
# - Password (or set GARMIN_PASSWORD env var)
# - MFA code (if enabled on your account)

# OAuth tokens will be saved to ~/.garminconnect
```

Puedes verificar tus credenciales en cualquier momento con:
```bash
uv run garmin-mcp-auth --verify
```

**Nota:** También puedes establecer las credenciales mediante variables de entorno:
```bash
GARMIN_EMAIL=your@email.com GARMIN_PASSWORD=secret garmin-mcp-auth
```

Si no tienes MFA activado, también puedes omitir `garmin-mcp-auth` y pasar `GARMIN_EMAIL` y `GARMIN_PASSWORD` como variables de entorno directamente a tu cliente MCP, si este lo admite. Para una mayor seguridad, es preferible utilizar el flujo de preautenticación anterior y mantener las credenciales fuera de la configuración del cliente MCP.

#### Paso 2: Configurar Claude Desktop

Añade lo siguiente a la configuración MCP de Claude Desktop **SIN** credenciales:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "garmin": {
      "command": "uvx",
      "args": [
        "--python",
        "3.12",
        "--from",
        "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ]
    }
  }
}
```

**Importante:** No es necesario incluir `GARMIN_EMAIL` ni `GARMIN_PASSWORD` en la configuración. El servidor utiliza los tokens guardados.

#### Paso 3: Reiniciar el cliente MCP

Tus datos de Garmin ya estarán disponibles para tu cliente MCP.

Para Codex y otros clientes, consulta los ejemplos siguientes.

---

### Configuración para desarrollo

1. Instala los paquetes necesarios en un entorno nuevo:

```bash
uv sync
```

## Ejecutar el servidor

### Configuración

Las credenciales de Garmin Connect se leen de variables de entorno:

- `GARMIN_EMAIL`: Tu dirección de correo electrónico de Garmin Connect
- `GARMIN_EMAIL_FILE`: Ruta a un archivo que contiene tu dirección de correo electrónico de Garmin Connect
- `GARMIN_PASSWORD`: Tu contraseña de Garmin Connect
- `GARMIN_PASSWORD_FILE`: Ruta a un archivo que contiene tu contraseña de Garmin Connect
- `GARMIN_IS_CN`: Establécelo en `true` para usar Garmin Connect China (garmin.cn) en lugar de la versión internacional (predeterminado: `false`)
- `GARMIN_FIT_DOWNLOAD_DIR`: Directorio predeterminado para los archivos de actividad descargados. Si se establece, omite la solicitud de configuración inicial de `download_activity_file`.
- `GARMIN_FIT_CONFIG`: Ruta al archivo de configuración persistente del directorio de descargas (predeterminado: `~/.garminconnect_fit_config.json`).

Los secretos basados en archivos son útiles en determinados entornos, por ejemplo dentro de un contenedor Docker. Ten en cuenta que no puedes establecer a la vez `GARMIN_EMAIL` y `GARMIN_EMAIL_FILE`; del mismo modo, tampoco puedes establecer simultáneamente `GARMIN_PASSWORD` y `GARMIN_PASSWORD_FILE`.

### Transporte

De forma predeterminada, el servidor se comunica mediante **stdio**, que es lo que esperan Claude Desktop, MCP Inspector y la mayoría de clientes locales. Para servir mediante **HTTP** en su lugar, por ejemplo al ejecutarlo en un contenedor o en Kubernetes, establece el transporte mediante variables de entorno:

- `GARMIN_MCP_TRANSPORT`: `stdio` (predeterminado), `streamable-http` o `sse`
- `GARMIN_MCP_HOST`: dirección de escucha para los transportes HTTP (predeterminado `127.0.0.1`; establece `0.0.0.0` únicamente cuando el endpoint esté protegido por un proxy inverso con autenticación)
- `GARMIN_MCP_PORT`: puerto de escucha para los transportes HTTP (predeterminado `8000`)
- `GARMIN_MCP_CALL_TIMEOUT`: tiempo límite por solicitud, en segundos, para las llamadas a Garmin (predeterminado `90`). La API de Garmin ocasionalmente deja una solicitud bloqueada indefinidamente; sin este límite, la llamada queda colgada hasta que vence el propio tiempo de espera del cliente MCP y este informa de que todo el servidor no responde. Cuando se agota el tiempo, la herramienta devuelve en su lugar un error claro que permite volver a intentarlo. Establécelo en `0` para desactivar este límite.

```bash
GARMIN_MCP_TRANSPORT=streamable-http garmin-mcp
```

Cuando se selecciona un transporte HTTP:

- Los clientes MCP se conectan a la ruta **`/mcp`** (por ejemplo, `http://localhost:8000/mcp`).
- Se expone un endpoint sencillo **`GET /healthz`** para comprobaciones de disponibilidad y estado.

El propio servidor **no realiza autenticación** en el endpoint HTTP. Colócalo detrás de un proxy inverso (nginx, Traefik, Authelia, etc.) si es accesible más allá de localhost.

### Garmin Connect China (garmin.cn)

Si utilizas Garmin Connect China (garmin.cn) en lugar de la versión internacional, establece la variable de entorno `GARMIN_IS_CN` en `true`:

```bash
# Pre-authenticate with Garmin Connect China
GARMIN_IS_CN=true garmin-mcp-auth

# Or use the CLI flag
garmin-mcp-auth --is-cn
```

Para Claude Desktop, añade `GARMIN_IS_CN` a la sección `env`:

```json
{
  "mcpServers": {
    "garmin": {
      "command": "uvx",
      "args": [
        "--python",
        "3.12",
        "--from",
        "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ],
      "env": {
        "GARMIN_IS_CN": "true"
      }
    }
  }
}
```

Para Docker, añade `GARMIN_IS_CN=true` a tu archivo `.env` o descoméntalo en `docker-compose.yml`.

### Probar el servidor localmente con MCP Inspector

Inspector se ejecuta directamente mediante npx sin necesidad de instalación. Ejecútalo desde la raíz del proyecto:

```bash
npx @modelcontextprotocol/inspector uv run garmin-mcp
```

Podrás inspeccionar y probar las herramientas.

### Con Claude Desktop

1. Crea una configuración en Claude Desktop:

Edita el archivo de configuración de Claude Desktop:

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Tienes dos opciones para ejecutar el MCP localmente con Claude.

#### Directamente desde GitHub sin clonar el repositorio

1. Añade esta configuración del servidor:

```json
{
  "mcpServers": {
    "garmin": {
      "command": "uvx",
      "args": [
        "--python",
        "3.12",
        "--from",
        "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ],
      "env": {
        "GARMIN_EMAIL": "YOUR_GARMIN_EMAIL",
        "GARMIN_PASSWORD": "YOUR_GARMIN_PASSWORD"
      }
    }
  }
}
```

Es posible que tengas que indicar la ruta completa a `uvx`; puedes comprobarla con `which uvx`.

2. Reinicia Claude Desktop.

#### Directamente desde tu copia local del repositorio

1. Añade esta configuración del servidor:

```
{
  "mcpServers": {
    "garmin-local": {
      "command": "uv",
      "args": [
        "--directory",
        "<full path to your local repository>/garmin_mcp",
        "run",
        "garmin-mcp"
      ]
    }
  }
}
```

2. Reinicia Claude Desktop.

### Con Codex

Codex utiliza TOML para configurar servidores MCP. Añade una de las siguientes entradas a `~/.codex/config.toml` después de autenticarte con `garmin-mcp-auth`.

También puedes pedir a tu cliente compatible con MCP que lo configure por ti. Por ejemplo:

```text
Install the Garmin MCP server from https://github.com/Taxuspt/garmin_mcp, authenticate with garmin-mcp-auth, and add it to my MCP configuration without storing my Garmin email or password.
```

#### Directamente desde GitHub sin clonar el repositorio

```toml
[mcp_servers.garmin]
command = "uvx"
args = [
  "--python",
  "3.12",
  "--from",
  "git+https://github.com/Taxuspt/garmin_mcp",
  "garmin-mcp"
]
```

#### Directamente desde tu copia local del repositorio

```toml
[mcp_servers.garmin-local]
command = "uv"
args = [
  "--directory",
  "/full/path/to/garmin_mcp",
  "run",
  "garmin-mcp"
]
```

Reinicia tu cliente MCP después de guardar el archivo.

### Con opencode

[opencode](https://opencode.ai) carga automáticamente un `opencode.json` a nivel de proyecto cuando se inicia desde la raíz de un repositorio, por lo que quienes clonen este repositorio tendrán Garmin MCP configurado contra el código fuente local sin ninguna configuración adicional.

#### Desde un clon de este repositorio, recomendado para desarrollo

Este repositorio incluye un [`opencode.json`](./opencode.json) que ejecuta el MCP mediante `uv run garmin-mcp`, de forma que siempre sigue el árbol de trabajo actual.

```bash
git clone https://github.com/Taxuspt/garmin_mcp.git
cd garmin_mcp
uv sync                # install dependencies
garmin-mcp-auth        # one-time Garmin login (skip if ~/.garminconnect already exists)
opencode               # launches with the garmin MCP attached
```

Comprueba que el servidor está conectado:

```bash
opencode mcp list
# ●  ✓ garmin   connected
#       uv run garmin-mcp
```

#### Desde cualquier otro directorio, instalación desde GitHub

Añade el servidor a la configuración global de opencode en `~/.config/opencode/opencode.json` después de ejecutar `garmin-mcp-auth`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "garmin": {
      "type": "local",
      "command": [
        "uvx",
        "--python",
        "3.12",
        "--from",
        "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ],
      "enabled": true,
      "timeout": 30000
    }
  }
}
```

Reinicia opencode después de guardar el archivo. La primera ejecución de `uvx` descarga y almacena el paquete en caché, por lo que el arranque inicial puede tardar unos segundos.

### Con Docker

Docker proporciona un entorno aislado y coherente para ejecutar el servidor MCP.

#### Inicio rápido con Docker Compose, recomendado

1. Crea un archivo `.env` con tus credenciales:

```bash
echo "GARMIN_EMAIL=your_email@example.com" > .env
echo "GARMIN_PASSWORD=your_password" >> .env
```

2. Inicia el contenedor:

```bash
docker compose up -d
```

3. Consulta los logs para supervisar el servidor:

```bash
docker compose logs -f garmin-mcp
```

#### Usar Docker directamente

```bash
# Build the image
docker build -t garmin-mcp .

# Run the container
docker run -it \
  -e GARMIN_EMAIL="your_email@example.com" \
  -e GARMIN_PASSWORD="your_password" \
  -v garmin-tokens:/root/.garminconnect \
  garmin-mcp
```

#### Usar secretos basados en archivos, más seguro

Para una mayor seguridad, especialmente en entornos de producción, utiliza secretos basados en archivos en lugar de variables de entorno:

1. Crea un directorio para los secretos y añade tus credenciales:

```bash
mkdir -p secrets
echo "your_email@example.com" > secrets/garmin_email.txt
echo "your_password" > secrets/garmin_password.txt
chmod 600 secrets/*.txt
```

2. Edita [docker-compose.yml](docker-compose.yml) y descomenta la sección de secretos:

```yaml
services:
  garmin-mcp:
    environment:
      - GARMIN_EMAIL_FILE=/run/secrets/garmin_email
      - GARMIN_PASSWORD_FILE=/run/secrets/garmin_password
    secrets:
      - garmin_email
      - garmin_password

secrets:
  garmin_email:
    file: ./secrets/garmin_email.txt
  garmin_password:
    file: ./secrets/garmin_password.txt
```

3. Inicia el contenedor:

```bash
docker compose up -d
```

#### Gestionar MFA con Docker

Si tienes activada la autenticación multifactor (MFA) en tu cuenta de Garmin:

1. Ejecuta el contenedor en modo interactivo:

```bash
docker compose run --rm garmin-mcp
```

2. Cuando se te solicite, introduce tu código MFA:

```
Garmin Connect MFA required. Please check your email/phone for the code.
Enter MFA code: 123456
```

3. Los tokens OAuth se guardarán en el volumen de Docker (`garmin-tokens`), por lo que no tendrás que volver a autenticarte en las siguientes ejecuciones.

4. Una vez configurado MFA, puedes ejecutar el contenedor normalmente:

```bash
docker compose up -d
```

#### Gestión del volumen Docker

Los tokens OAuth se almacenan en un volumen Docker persistente para evitar tener que volver a autenticarte:

```bash
# List volumes
docker volume ls

# Inspect the tokens volume
docker volume inspect garmin_mcp_garmin-tokens

# Remove the volume (will require re-authentication)
docker volume rm garmin_mcp_garmin-tokens
```

#### Usarlo con Claude Desktop mediante Docker

Para usar el servidor MCP en Docker con Claude Desktop, puedes configurarlo para que se comunique con el contenedor. Sin embargo, ten en cuenta que los servidores MCP suelen comunicarse mediante stdio, que funciona mejor ejecutando directamente el proceso. Para despliegues basados en Docker, considera utilizar en su lugar el método estándar con `uvx` mostrado en la sección [Con Claude Desktop](#con-claude-desktop).


## Ejemplos de uso

Una vez conectado en Claude, puedes hacer preguntas como:

- "Show me my recent activities"
- "What was my sleep like last night?"
- "How many steps did I take yesterday?"
- "Show me the details of my latest run"
- "Analyze my last ride's power zones and compare to my training zones"
- "Show me my CTL, ATL, and TSB trend for the last 6 weeks"
- "What was my power duration curve from yesterday's ride? Estimate my FTP."
- "Analyze the FIT data from my last cycling activity — how was my shifting quality on the climbs?"
- "Show me my HRV trend for the last 2 weeks and flag any recovery concerns"
- "What's my season best 20-minute power and when did I set it?"

## Solución de problemas

### "Failed to spawn process: No such file or directory"

Si Claude Desktop no encuentra `uvx`, es porque `uvx` no está en el PATH que utiliza Claude Desktop. Para solucionarlo:

1. Busca dónde está instalado `uvx`:
```bash
which uvx
```

2. Utiliza la ruta completa en tu configuración. Por ejemplo, si `uvx` se encuentra en `/Users/username/.cargo/bin/uvx`:
```json
{
  "mcpServers": {
    "garmin": {
      "command": "/Users/username/.cargo/bin/uvx",
      "args": [
        "--python",
        "3.12",
        "--from",
        "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ]
    }
  }
}
```

### Problemas de inicio de sesión

Si tienes problemas para iniciar sesión:

1. Comprueba que tus credenciales son correctas
2. Comprueba si Garmin Connect requiere una verificación adicional
3. Asegúrate de que el paquete garminconnect está actualizado

### Logs

Para otros problemas, consulta los logs de Claude Desktop en:

- macOS: `~/Library/Logs/Claude/mcp-server-garmin.log`
- Windows: `%APPDATA%\Claude\logs\mcp-server-garmin.log`

### Autenticación multifactor de Garmin Connect (MFA)

#### Cómo funciona MFA con los servidores MCP

Los servidores MCP se ejecutan como procesos en segundo plano sin acceso directo a una terminal. Si tu cuenta de Garmin tiene MFA activado, debes autenticarte una vez mediante la herramienta de preautenticación antes de poder ejecutar el servidor.

#### Recomendado: herramienta de preautenticación

La forma más sencilla de gestionar MFA es utilizar la herramienta de autenticación específica:

```bash
garmin-mcp-auth
```

Esto guarda los tokens OAuth en `~/.garminconnect` para usos posteriores. El servidor utilizará automáticamente estos tokens cuando se ejecute en Claude Desktop u otros clientes MCP.

**Opciones adicionales:**

```bash
# Use environment variables for credentials
GARMIN_EMAIL=you@example.com GARMIN_PASSWORD=secret garmin-mcp-auth

# Verify existing tokens
garmin-mcp-auth --verify

# Force re-authentication (e.g., when tokens expire)
garmin-mcp-auth --force-reauth

# Use custom token location
garmin-mcp-auth --token-path ~/.garmin_tokens
```

#### Alternativa: primera ejecución manual

También puedes autenticarte ejecutando el servidor una vez de forma interactiva:

```bash
# Store credentials in files for security
echo "your_email@example.com" > ~/.garmin_email
echo "your_password" > ~/.garmin_password
chmod 600 ~/.garmin_email ~/.garmin_password

# Run server interactively to authenticate
GARMIN_EMAIL_FILE=~/.garmin_email GARMIN_PASSWORD_FILE=~/.garmin_password \
  uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp

# Enter MFA code when prompted
# Tokens will be saved automatically
# Now add to Claude Desktop config without credentials
```

Después de la autenticación inicial, configura Claude Desktop **sin** credenciales, ya que los tokens estarán guardados:

```json
{
  "mcpServers": {
    "garmin": {
      "command": "uvx",
      "args": [
        "--python",
        "3.12",
        "--from",
        "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ]
    }
  }
}
```

#### Usar Docker con MFA

Si utilizas Docker, sigue la sección [Gestionar MFA con Docker](#gestionar-mfa-con-docker) anterior para disponer de un flujo sencillo con almacenamiento persistente de tokens.

#### Solución de problemas de MFA

**Error: "MFA authentication required but no interactive terminal available"**

Solución:
1. Abre una terminal
2. Ejecuta: `garmin-mcp-auth`
3. Introduce tus credenciales y el código MFA
4. Reinicia Claude Desktop

**Token caducado**

Los tokens OAuth caducan periódicamente (aproximadamente cada 6 meses). Vuelve a autenticarte:
```bash
garmin-mcp-auth --force-reauth
```

**Comprobar que los tokens funcionan**
```bash
garmin-mcp-auth --verify
```

## Pruebas

Este proyecto incluye pruebas completas para todas las herramientas MCP. **Actualmente todas las pruebas pasan correctamente (100 %)**.

### Ejecutar las pruebas

```bash
# Run all integration tests (default - uses mocked Garmin API)
uv run pytest tests/integration/

# Run tests with verbose output
uv run pytest tests/integration/ -v

# Run a specific test module
uv run pytest tests/integration/test_health_wellness_tools.py -v

# Run end-to-end tests (requires real Garmin credentials)
uv run pytest tests/e2e/ -m e2e -v
```

### Estructura de las pruebas

- **Pruebas de integración** (más de 200 pruebas): prueban todas las herramientas MCP utilizando la integración FastMCP con respuestas simuladas de la API de Garmin
- **Pruebas end-to-end** (4 pruebas): prueban el servidor MCP real y la API de Garmin; requieren credenciales válidas

## Reinstalar desde una ruta local

Si estás trabajando desde una copia local o un fork:

```bash
uv tool install --python 3.12 --force C:\Users\aresd\Desktop\programacion\garmin_mcp
```