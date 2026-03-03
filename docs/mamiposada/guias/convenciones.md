# Guías y convenciones — POSMamiPosada

## Estructura de archivos

- Cada **View** (`.xaml` + `.xaml.cs`) tiene su **ViewModel** correspondiente en la misma posición relativa dentro de `ViewModel/`.
- Las ventanas principales van en `Views/`; los diálogos y pop-ups en `Modals/`.
- Nomenclatura de archivos:
  - Vistas: `NombreVentana.xaml`
  - ViewModels: `NombreVentanaViewModel.cs`
  - Modales: `ModalNombre.xaml` / `ModalNombreViewModel.cs`

## Nomenclatura de código

- **PascalCase** para clases, propiedades y métodos públicos.
- **camelCase** para campos privados y variables locales.
- Prefijo `_` para campos privados con backing field (ej: `_impresoraActual`).
- Clases de acceso a datos con prefijo `Api` (ej: `ApiTickets`, `ApiClientes`).
- Enums en su carpeta dedicada con nombre descriptivo (ej: `EstadoTicket`, `ModoAperturaPV`).

## MVVM y bindings

- Implementar `INotifyPropertyChanged` en ViewModels y modelos que se bindean a la UI.
- Usar `RelayCommand` (en `Model/Utils/`) para los `ICommand`.
- **Nunca** referenciar `View` desde `ViewModel` directamente.
- Code-behind (`.xaml.cs`) solo para lógica de UI que no es posible resolver con bindings.

## Acceso a datos

- Todas las consultas van en las clases `Api*` dentro de `Model/Data/`.
- Usar consultas parametrizadas para evitar inyección SQL.
- La conexión se obtiene de `VariablesGlobales.Instancia.Conexion`.

## Hilos y dispatcher

- Si se accede a elementos de la UI desde `Task.Run` o `async/await`, usar:
  ```csharp
  Application.Current.Dispatcher.Invoke(() => { /* Código UI */ });
  ```

## Git

- Commits descriptivos y en español.
- No versionar `config.local.json`, `config.dat` ni archivos de `bin/` / `obj/`.
- Pull requests pequeños y revisados.
