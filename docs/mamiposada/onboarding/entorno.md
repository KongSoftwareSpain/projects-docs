# Entorno de desarrollo — POSMamiPosada

## Requisitos previos

- **Visual Studio 2022** (o superior) con la carga de trabajo *Desarrollo de escritorio .NET*.
- **.NET 8 SDK** (target `net8.0-windows`).
- **Microsoft Access Database Engine** (Jet 4.0 / ACE) — necesario para la conexión OLE DB.
- **Git** para clonar el repositorio.

## Setup paso a paso

1. **Clonar el repositorio:**
   ```bash
   git clone <url-del-repositorio> POSMamiPosadaKairo
   ```

2. **Abrir la solución** `POSMamiPosadasKairo.sln` en Visual Studio.

3. **Configurar la base de datos local:**
   - Copiar `Model/Utils/config.local.template.json` → `Model/Utils/config.local.json`.
   - Editar `config.local.json` con la ruta real al archivo Access (`.mdb`) en tu equipo:
     ```json
     {
       "RutaBaseDatos": "C:\\Tu\\Ruta\\Personal\\Datos.mdb"
     }
     ```
   - En Visual Studio, seleccionar `config.local.json` y establecer:
     - **Acción de compilación:** `Contenido`
     - **Copiar en el directorio de salida:** `Copiar si es más reciente`

4. **Configurar conexión al Gestor de Colas** *(opcional)*:
   - Editar el archivo `ConexionGestorCola.ini` en la raíz del proyecto con la URL y credenciales de la API del gestor de colas.

5. **Compilar y ejecutar** (`F5` o `Ctrl+F5`).
   - La plataforma objetivo es **x86**; asegúrate de tenerla seleccionada.

## Archivos no versionados

| Archivo | Descripción |
|---|---|
| `config.local.json` | Configuración local de cada desarrollador (ruta BD). Está en `.gitignore`. |
| `config.dat` | Datos encriptados de configuración generados en ejecución. |

## Errores comunes

- **"Falta el archivo de configuración local: config.local.json"** → No se ha creado el archivo a partir de la plantilla.
- **Error de conexión OLE DB** → Verificar que la ruta en `config.local.json` es correcta y que el motor Access (Jet / ACE) está instalado.
- **Error de subprocesos (UI thread)** → Si accedes a la UI desde `Task.Run` o `async/await`, usar `Dispatcher.Invoke(...)`.
