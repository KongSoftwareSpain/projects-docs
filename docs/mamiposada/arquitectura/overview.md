# Arquitectura — POSMamiPosada

Visión general del sistema y sus componentes principales.

## Patrón MVVM

La aplicación sigue estrictamente el patrón **Model-View-ViewModel**:

```
POSMamiPosadasKairo/
├── Model/                  # Capa de datos y lógica de negocio
│   ├── Models/             # Entidades del dominio
│   ├── Data/               # Acceso a datos (API* / OLE DB)
│   ├── Services/           # Servicios transversales
│   ├── Enums/              # Enumeraciones del dominio
│   ├── Converters/         # Conversores de datos
│   ├── DTOs/               # Data Transfer Objects
│   ├── Utils/              # Utilidades y helpers
│   └── VariablesGlobales.cs  # Singleton de estado global
│
├── View/                   # Capa de presentación (XAML)
│   ├── Views/              # Ventanas principales
│   ├── Modals/             # Ventanas modales / diálogos
│   ├── Components/         # Componentes reutilizables
│   ├── Converters/         # Conversores de UI (IValueConverter)
│   └── MainWindow.xaml     # Ventana raíz (contenedor)
│
├── ViewModel/              # Capa de lógica de presentación
│   ├── Views/              # ViewModels de ventanas principales
│   └── Modals/             # ViewModels de modales
│
├── Resources/              # Recursos estáticos
│   ├── Fonts/              # Tipografías (Poppins)
│   ├── Images/             # Iconos e imágenes raster
│   └── ImagesVectoriales/  # Imágenes vectoriales
│
└── Libs/                   # DLLs de terceros embebidas
    ├── BarcodeLib.dll
    └── EPPlus.dll
```

## Estado global — `VariablesGlobales`

Singleton (`VariablesGlobales.Instancia`) que mantiene el estado compartido de la aplicación:

- **Conexión a BD** (`OleDbConnection`) — Se abre al iniciar la app.
- **Ticket actual** — El ticket de venta en curso.
- **Usuario actual** — El usuario logueado.
- **Cliente actual** — El cliente seleccionado para la venta.
- **Almacén actual** — El almacén operativo.
- **Serie y Tarifa** — La serie de facturación y tarifa activas.
- **Impresora actual** — Impresora seleccionada para tickets y etiquetas.
- **Token API** — Token de autenticación para el Gestor de Colas.
- **Datos de empresa** — Configuración de la empresa predeterminada.

Implementa `INotifyPropertyChanged` para notificar cambios a la UI.

## Capa de datos (`Model/Data/`)

Clases con prefijo `Api*` que encapsulan las consultas OLE DB contra la base de datos Access:

| Clase | Responsabilidad |
|---|---|
| `ApiTickets` | CRUD de tickets, líneas de ticket, pagos, estadísticas |
| `ApiArticulos` | Consulta de artículos y stock |
| `ApiClientes` | CRUD de clientes |
| `ApiUsuarios` | Autenticación y gestión de usuarios |
| `ApiSeries` | Series de facturación |
| `ApiTarifas` | Tarifas de precios |
| `ApiIvas` | Tipos de IVA |
| `ApiAlmacenes` | Gestión de almacenes |
| `ApiVales` | Vales y tickets regalo |
| `ApiConfiguracionEmpresa` | Datos fiscales y de la empresa |
| `ApiPicajeUsuarios` | Fichaje / control de presencia |
| `GestorColas/` | Integración con API REST externa de gestión de turnos |

## Modelos principales (`Model/Models/`)

| Modelo | Descripción |
|---|---|
| `Ticket` | Ticket de venta con líneas, totales, impuestos, impresión |
| `LineaTicket` | Línea individual de un ticket (artículo, cantidad, precio, IVA) |
| `Articulo` / `ArticuloStock` / `ArticuloResumen` | Artículo del catálogo con variantes de información |
| `Cliente` | Datos del cliente |
| `Usuario` | Datos del usuario / operador |
| `Vale` | Vale / ticket regalo |
| `ConfiguracionEmpresa` | Datos fiscales y configuración global |
| `Serie` / `Tarifa` / `Almacen` | Entidades de configuración comercial |
| `Lote` | Control de lotes de producto |
| `TurnoLlamado` | Turnos del gestor de colas |

## Servicios (`Model/Services/`)

| Servicio | Responsabilidad |
|---|---|
| `EtiquetaService` | Generación e impresión de etiquetas de producto |
| `GestorColasService` | Comunicación con la API REST del gestor de turnos |
| `TokenService` | Gestión del token de autenticación de la API |
| `LoteService` | Lógica de gestión de lotes |
| `LogReaderService` | Lectura y filtrado de logs de la aplicación |

## Utilidades (`Model/Utils/`)

| Utilidad | Descripción |
|---|---|
| `ImprimirUtilities` | Impresión directa de tickets vía comandos ESC/POS |
| `PdfUtilities` | Generación de PDFs (PdfSharpCore) |
| `RawPrinterHelper` | Envío raw de datos a impresora |
| `EmailService` | Envío de correos (ticket digital) |
| `LogAcciones` | Registro de acciones / auditoría |
| `ConfiguracionLocal` | Lectura del `config.local.json` |
| `EncryptedConfigManager` | Gestión de configuración encriptada (`config.dat`) |
| `IniFile` | Lectura de archivos `.ini` |
| `RelayCommand` | Implementación de `ICommand` para MVVM |
| `Toast` | Componente de notificaciones tipo toast |

## Flujo de inicio

1. `App.OnStartup()` → Registra encoding, llama a `VariablesGlobales.Instancia.IniciarConexion()`.
2. Se abre la conexión OLE DB a la base de datos Access.
3. Se carga `MainWindow` que actúa como contenedor de navegación.
4. El usuario navega entre vistas: Login → Menú Principal → Pantalla de Ventas / Pedidos / etc.

## Principios de diseño

- **MVVM estricto** — Separación clara Model / View / ViewModel.
- **Singleton global** — `VariablesGlobales` centraliza el estado compartido.
- **Data Access directo** — Consultas OLE DB sin ORM.
- **Impresión nativa** — Soporte ESC/POS para impresoras térmicas.
- **Plataforma x86** — Requerido por el driver Jet OLEDB 4.0.
