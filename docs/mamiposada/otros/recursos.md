# Recursos — POSMamiPosada

## Dependencias NuGet

| Paquete | Uso |
|---|---|
| `System.Data.OleDb` (9.0.6) | Conexión a base de datos Access |
| `PdfSharpCore` (1.3.67) | Generación de documentos PDF |
| `Microsoft.Xaml.Behaviors.Wpf` (1.1.135) | Behaviors para bindings avanzados en XAML |

## DLLs embebidas (`/Libs`)

| Librería | Uso |
|---|---|
| `BarcodeLib.dll` | Generación de códigos de barras |
| `EPPlus.dll` | Lectura/escritura de archivos Excel (etiquetas) |

## Recursos de la aplicación (`/Resources`)

- **Fonts/** — Tipografía Poppins (Regular).
- **Images/** — Iconos de acciones del TPV (pagar, pausar, devolver, pedidos, etc.).
- **ImagesVectoriales/** — Recursos vectoriales para la interfaz.

## Funcionalidades principales del TPV

| Módulo | Descripción |
|---|---|
| Pantalla de Ventas | Creación y gestión de tickets de venta |
| Gestión de Pedidos | Creación, edición, división y seguimiento de pedidos |
| Clientes | CRUD de clientes, selector de cliente por venta |
| Vales y Tickets Regalo | Creación y canje de vales y tickets regalo |
| Pagos | Pago total, parcial, multiforma (efectivo, tarjeta, etc.) |
| Devoluciones | Devolución y modificación de tickets existentes |
| Impresión | Tickets térmicos (ESC/POS), PDFs, envío por email |
| Etiquetas | Generación e impresión de etiquetas de productos |
| Fichaje / Picaje | Control de presencia de empleados |
| Gestor de Colas | Integración con sistema de turnos |
| Logs y Auditoría | Registro y visualización de acciones |
| Resumen de Artículos | Consulta de stock y movimientos |

## Herramientas de desarrollo recomendadas

- **Visual Studio 2022** — IDE principal.
- **Git** — Control de versiones.
- **Access Database Engine** — Para la base de datos local.
