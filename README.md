# Guía de setup: opencode + GitHub Copilot en Windows

> Preparado para el equipo de ingeniería de BAM Center — Mayo 2026

---

## ¿Qué es opencode?

opencode es un agente de IA open source que trabaja directamente en tu terminal. Le das instrucciones en lenguaje natural y él lee tu código, edita archivos, ejecuta comandos y completa tareas de desarrollo de forma autónoma. Piensa en ello como un compañero de equipo que puede ejecutar, no solo sugerir.

Con vuestra licencia de GitHub Copilot, podéis usarlo **sin coste adicional** — sin API keys, sin suscripciones extra.

---

## 1. Instalación en Windows

### Paso 1: Instalar opencode CLI

Instala primero la CLI para que opencode esté disponible en tu sistema y pueda ejecutarse desde cualquier proyecto. Abre **PowerShell** o **Windows Terminal** y ejecuta una de estas opciones:

```powershell
# Opción recomendada si ya tenéis Node.js
npm i -g opencode-ai@latest
```

Si preferís usar un gestor de paquetes de Windows:

```powershell
# Con Scoop
scoop install opencode

# Con Chocolatey
choco install opencode
```

Si no tenéis Scoop instalado y queréis usarlo:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

Verifica que la CLI está disponible:

```powershell
opencode --version
```

### Paso 2: Instalar opencode Desktop

La forma recomendada de usar opencode en el día a día es la app de escritorio. Después de verificar que la CLI funciona, instala Desktop para trabajar desde la interfaz visual y abrir tus proyectos desde ahí.

Descargad el instalador de Windows (x64) desde: https://opencode.ai/download

Una vez instalado:

1. Abre opencode Desktop
2. Selecciona la carpeta de tu proyecto
3. Conecta GitHub Copilot desde la interfaz o usando `/connect` dentro de la sesión
4. Usa **Tab** para cambiar entre agentes, igual que en la CLI

---

## 2. Conectar GitHub Copilot

Este es el paso clave — conectar vuestra licencia de Copilot para tener acceso a modelos potentes sin pagar API.

1. Abre una terminal en la carpeta de cualquier proyecto Python
2. Ejecuta `opencode`
3. Dentro de opencode, escribe `/connect`
4. Selecciona **GitHub Copilot**
5. Se abrirá el navegador — ve a `github.com/login/device` e introduce el código que te muestre
6. Autoriza la conexión

Listo. Ahora puedes seleccionar modelos con `/models`. Deberías ver modelos como Claude, GPT y Gemini disponibles a través de tu suscripción de Copilot.

---

## 3. Tu primer proyecto con opencode

```powershell
# Navega a tu proyecto
cd C:\Users\tu-usuario\proyectos\mi-proyecto-python

# Lanza opencode
opencode
```

Prueba algo sencillo para verificar que todo funciona:

```
Explícame la estructura de este proyecto y qué hace cada módulo
```

Si responde analizando tus archivos, la conexión funciona.

### Pruebas más interesantes

```
Añade manejo de errores a todas las funciones de utils.py y escribe tests con pytest
```

```
Crea un script de data augmentation para imágenes usando albumentations
```

```
Revisa el código de training.py y sugiere optimizaciones de rendimiento
```

---

## 4. Agentes: Build vs Plan

opencode tiene dos agentes integrados que puedes alternar con la tecla **Tab**:

- **Build** — Agente completo: lee, escribe, ejecuta. Para desarrollo activo.
- **Plan** — Solo lectura: analiza código y propone cambios sin tocar nada. Para explorar y planificar.

Empieza con **Plan** si quieres entender un codebase antes de modificarlo.

---

## 5. Configurar el agente Python ML (compartido entre proyectos)

opencode tiene dos niveles de configuración que se complementan:

- **Agente custom global** — Un agente reutilizable que vive en `~/.config/opencode/agents/` y funciona en cualquier proyecto. Define *cómo* trabaja el agente (reglas de Python, convenciones de ML, permisos).
- **AGENTS.md por proyecto** — Un fichero en la raíz de cada repo que define *qué* es este proyecto concreto (estructura, comandos, particularidades).

### Paso 1: Instalar el agente custom global

Copia el fichero `agents/python-ml-dev.md` (incluido con esta guía) a tu carpeta de agentes:

```powershell
# Crear la carpeta si no existe
mkdir -p $HOME\.config\opencode\agents

# Copiar el fichero del agente
copy agents\python-ml-dev.md $HOME\.config\opencode\agents\python-ml-dev.md
```

La próxima vez que abras opencode, el agente `python-ml-dev` aparecerá como opción al pulsar **Tab** junto a Build y Plan.

### Paso opcional: sincronizar los agentes desde este repo con un symlink

Si queréis que todo el equipo use los agentes versionados en este repo y reciba actualizaciones con `git pull`, podéis sustituir la carpeta global de agentes de opencode por un symlink a la carpeta `agents/` del repo.

**Importante:** esto elimina cualquier agente global que ya exista en `~/.config/opencode/agents/`. Hacedlo solo si queréis reemplazar esa carpeta por los agentes compartidos del repo.

```powershell
# Ajusta esta ruta a la ubicación real del repo en tu máquina
$repoAgents = "C:\Users\tu-usuario\proyectos\CodingAgents\agents"

# Eliminar la carpeta global actual de agentes de opencode
Remove-Item -Recurse -Force "$HOME\.config\opencode\agents"

# Crear el symlink hacia los agentes versionados del repo
New-Item -ItemType SymbolicLink -Path "$HOME\.config\opencode\agents" -Target $repoAgents
```

Si Windows no permite crear el symlink, ejecutad PowerShell como administrador o activad el modo desarrollador de Windows. Después de crear o cambiar agentes, cerrad y volved a abrir opencode para que cargue la nueva configuración.

Este agente ya trae configurado:

- System prompt especializado en Python + ML + visión por computador
- Reglas de seguridad (no borrar datos, no sobreescribir checkpoints, avisar antes de entrenamientos largos)
- Convenciones de código (type hints, docstrings, pathlib, etc.)
- Permisos: puede editar y ejecutar, pero bloquea `rm -rf` y pide confirmación para `sudo`

### Paso 2: Crear un AGENTS.md en cada proyecto

El agente global sabe *cómo* trabajar con Python/ML. El `AGENTS.md` le dice *qué* es este proyecto concreto.

Dentro de opencode, ejecuta:

```
/init
```

Esto escanea tu proyecto y genera un `AGENTS.md` automáticamente. Luego revísalo y añade lo que falte. Un buen `AGENTS.md` de proyecto incluye:

```markdown
# AGENTS.md — [Nombre del proyecto]

## Stack
- Python 3.10+, PyTorch, albumentations
- Entorno: conda activate mi-entorno
- GPU: NVIDIA RTX 3080 (CUDA 12.x)

## Comandos
- Tests: `python -m pytest tests/`
- Entrenar: `python train.py --config configs/default.yaml`
- Lint: `ruff check .`

## Estructura
- `src/` — Código fuente
- `data/` — Datasets (NO modificar originales)
- `models/` — Checkpoints guardados
- `configs/` — Config de experimentos

## Particularidades de este proyecto
- [Ejemplo: "Usamos un formato de anotación propio, ver docs/annotation_format.md"]
- [Ejemplo: "El modelo base es ResNet18 preentrenado en ImageNet"]
```

**Commitead el AGENTS.md al repo** — así todo el equipo se beneficia.

### Cómo funciona todo junto

Cuando abres opencode en un proyecto y seleccionas `python-ml-dev` con Tab:

1. opencode carga el system prompt del agente global (reglas de Python, ML, seguridad)
2. opencode lee el `AGENTS.md` del proyecto (estructura, comandos, particularidades)
3. El agente trabaja con ambos contextos combinados

---

## 6. Tips prácticos

### Atajos dentro de opencode

- **Tab** — Cambiar entre agentes (Build / Plan)
- **/models** — Cambiar de modelo
- **/connect** — Conectar un proveedor
- **/init** — Generar o mejorar AGENTS.md
- **/clear** — Limpiar la conversación actual

### Buenos hábitos

1. **Empieza con tareas pequeñas y verificables** — "añade un test para esta función" antes de "refactoriza todo el módulo"
2. **Usa Plan primero** — Deja que el agente analice antes de que escriba
3. **Commitea tu AGENTS.md** — Es parte del proyecto, todo el equipo se beneficia
4. **Sesiones cortas y enfocadas** — Una tarea por sesión funciona mejor que conversaciones largas
5. **Verifica siempre** — El agente es potente pero no infalible. Revisa lo que genera.

### Modelos recomendados para Python/ML

Desde Copilot tendréis acceso a varios modelos. Para tareas de código Python:

- **Claude Sonnet** — Buen equilibrio velocidad/calidad para la mayoría de tareas
- **GPT-5** — Fuerte en razonamiento y tareas complejas
- **Gemini** — Buena opción para contextos grandes (archivos largos)

Probad varios y quedaos con el que mejor os funcione para vuestro tipo de trabajo.

---

## 7. Recursos

- Documentación oficial: https://opencode.ai/docs/
- Proveedores y configuración: https://opencode.ai/docs/providers/
- Setup en Windows (WSL): https://opencode.ai/docs/windows-wsl/
- Agentes y personalización: https://opencode.ai/docs/agents/
- Especificación AGENTS.md: https://agents.md/

---

## ¿Problemas?

| Problema | Solución |
|---|---|
| `opencode` no se reconoce como comando | Verifica que está en tu PATH. Con Scoop se añade automáticamente. |
| No aparecen modelos tras `/connect` | Asegúrate de que tu cuenta de GitHub tiene Copilot activo. |
| Errores de permisos en Windows | Ejecuta la terminal como administrador para la instalación. |
| El agente no entiende el proyecto | Crea o mejora tu `AGENTS.md` con `/init`. |
| Terminal con caracteres raros | Usa Windows Terminal (no cmd.exe). |

---

*Preparado por [Tu nombre] — Masterclass BAM Center, 20 mayo 2026*
*¿Dudas? Escríbeme a [tu email]*
