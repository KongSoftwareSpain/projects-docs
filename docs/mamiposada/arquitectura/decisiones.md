# Decisiones técnicas — POSMamiPosada

## ADR-001: Base de datos Access (Jet OLEDB 4.0)

- **Decisión:** Usar Microsoft Access (`.mdb`) como base de datos local mediante `OleDbConnection` con el proveedor Jet 4.0.
- **Motivo:** Portabilidad extrema — la BD viaja como un único archivo junto al ejecutable, sin necesidad de instalar un servidor de base de datos. Ideal para un TPV de escritorio distribuido en múltiples puntos.
- **Alternativas consideradas:** SQL Server LocalDB, SQLite.
- **Consecuencias:** Requiere compilación **x86** (Jet 4.0 no dispone de driver de 64 bits). Sin soporte de procedimientos almacenados complejos. La BD lleva contraseña de protección.

## ADR-002: Patrón MVVM manual

- **Decisión:** Implementar MVVM sin framework externo (sin Prism, sin MVVM Toolkit).
- **Motivo:** Simplicidad y menor curva de aprendizaje. El equipo utiliza `RelayCommand` propio e `INotifyPropertyChanged` manual.
- **Alternativas consideradas:** CommunityToolkit.Mvvm, Prism.
- **Consecuencias:** Más código boilerplate (commands, property changed). Mayor control sobre el flujo.

## ADR-003: Singleton de estado global (`VariablesGlobales`)

- **Decisión:** Centralizar todo el estado compartido (conexión, usuario, ticket, impresoras, etc.) en un singleton estático.
- **Motivo:** Acceso sencillo desde cualquier capa sin inyección de dependencias.
- **Alternativas consideradas:** Contenedor IoC / DI.
- **Consecuencias:** Acoplamiento alto al singleton. Dificultad para tests unitarios. Se debe tener cuidado con el acceso desde hilos secundarios (usar `Dispatcher.Invoke`).

## ADR-004: Impresión directa ESC/POS

- **Decisión:** Enviar comandos ESC/POS directamente a la impresora térmica vía `RawPrinterHelper`.
- **Motivo:** Control total del formato del ticket. Compatible con la mayoría de impresoras térmicas de punto de venta.
- **Alternativas consideradas:** Generación de PDF e impresión vía driver Windows.
- **Consecuencias:** Código de formateo delicado. Se mantiene un servicio de impresión móvil adicional por email (PDF).

## ADR-005: Librerías embebidas (DLLs en `/Libs`)

- **Decisión:** Incluir `BarcodeLib.dll` y `EPPlus.dll` como referencias directas en vez de paquetes NuGet.
- **Motivo:** Versiones específicas estabilizadas; evitar incompatibilidades con actualizaciones automáticas.
- **Alternativas consideradas:** Paquetes NuGet.
- **Consecuencias:** Actualización manual de las DLLs. No hay gestión automática de dependencias transitivas.

## ADR-006: Configuración local no versionada

- **Decisión:** Cada desarrollador tiene su `config.local.json` (fuera de Git) generado a partir de `config.local.template.json`.
- **Motivo:** Cada entorno tiene rutas y configuraciones diferentes.
- **Consecuencias:** Riesgo de que un desarrollador olvide crearlo y obtenga errores de ejecución. Se mitiga con un mensaje de error explícito.
