# Configuración de Entorno SQL Server Express (Base de Datos Local)

## 1. Instalación de SQL Server Express

Para trabajar con bases de datos en local, primero debes instalar **SQL Server Express**, si aún no lo tienes:

- Descarga el instalador desde la página oficial:
  - Selecciona la opcion gratuita de **Express**
    👉 [Descargar SQL Server Express](https://www.microsoft.com/es-es/sql-server/sql-server-downloads)
- Elige la opción de **Instalación Básica** durante el proceso.

> Esto instalará SQL Server con la configuración mínima necesaria para comenzar rápidamente.

---

## 2. Verificar la Conexión a SQL Server Express

Una vez instalado SQL Server Express, verifica los datos de conexión y configura el entorno.

### 2.1 Abrir SQL Server Management Studio (SSMS)

- Abre **SQL Server Management Studio (SSMS)**.

### 2.2 Conectar al Servidor

- En la ventana de inicio de sesión:
  - **Servidor**: `localhost\SQLEXPRESS` _(o el nombre real de la instancia)_.
  - **Autenticación**: Selecciona:
    - `Autenticación de Windows` o `SQL Server Authentication` si ya has creado un usuario.
  - Opcional:
    - Marcar `Encrypt = Mandatory`.
    - Marcar `Trust Server Certificate`.

---

## 3. Ejecutar Script de Generación de Base de Datos

- Selecciona la conexión a `localhost`, abre el script de generación y ejecútalo.

Tipos de importación disponibles:

- 🔹 **Importacion automatica**:  
  Ejecuta el archivo dado. _(solicítalo al administrador de la BD)_.
- 🔹 **Importación Personalizada**(Opcional):
  Para generar un script personalizado desde una base de datos existente:
  1. Abre SSMS y selecciona la base de datos a exportar.
  2. Clic derecho > **Tareas > Generar Scripts...**
  3. Selecciona:
  - **Objetos**: Todos o algunos.
  - **Salida**: Script, archivo, nueva ventana, etc.
  4. En **Opciones Avanzadas > Types of data to script**:
  - `Schema only`: Solo estructura.
  - `Data only`: Solo los datos.
  - `Schema and Data`: Estructura + datos.

---

## 4. Habilitar el Login con SQL Server

1. Haz clic derecho sobre el servidor y selecciona **Propiedades**.
2. Ve a **Security**.
3. Cambia a:  
   `SQL Server and Windows Authentication mode`.

---

## 5. Habilitar y Configurar el Protocolo TCP/IP

1. Abre **SQL Server Configuration Manager**.
2. Ve a:  
   `Configuración de red de SQL Server > Protocolos de SQLEXPRESS`.
3. Habilita el protocolo **TCP/IP**.

### 5.1 Establecer Puerto Fijo (recomendado)

1. En la configuración de TCP/IP, ve a la pestaña **Direcciones IP**.
2. En **IPAll**:
   - Borra el valor de `TCP Dynamic Ports`.
   - Establece un puerto en `TCP Port` (por ejemplo, `1433`).
3. Guarda los cambios y **reinicia el servicio**:
   - Dirígete a `Servicios > SQL Server (SQLEXPRESS)`.
   - Haz clic derecho sobre el servicio y selecciona **Reiniciar**.

---

## 6. Verificar Conexión

- Asegúrate de que puedes conectarte desde:
  - SSMS.
  - Tu aplicación backend (verifica las variables de entorno y conexión).
  - Asi deberia verse tu .env:

---

## Ejemplo de archivo `.env`

```env
# Configuración de la Base de Datos SQL Server
DB_NAME="DataBase"                # Nombre de la base de datos
DB_USER="Usuario"                       # Usuario de la base de datos
DB_PASSWORD="Contraseña"           # Contraseña del usuario
DB_SERVER="DESKTOP-AAXXX\SQLEXPRESS"  # Nombre del servidor y la instancia de SQL Server
```
