# Configuración del Entorno de Desarrollo

Esta guía te ayudará a configurar tu entorno local para desarrollar en el proyecto Fichaje.

## 📋 Requisitos Previos

### Software Necesario

| Software       | Versión Mínima | Notas                         |
| -------------- | -------------- | ----------------------------- |
| **Node.js**    | 18.x           | LTS recomendado               |
| **npm**        | 9.x            | Incluido con Node.js          |
| **SQL Server** | 2019+          | Express Edition es suficiente |
| **Git**        | 2.x            | Para control de versiones     |
| **VS Code**    | Latest         | Editor recomendado            |

### Cuentas y Accesos

- [ ] Acceso al repositorio Git
- [ ] Credenciales de base de datos SQL Server
- [ ] Credenciales de Azure Blob Storage (para desarrollo)
- [ ] VPN/Acceso a red interna (si aplica)

## 🔧 Instalación Paso a Paso

### 1. Clonar el Repositorio

```bash
git clone https://github.com/tu-organizacion/fichaje.git
cd fichaje
```

### 2. Configurar Backend

#### 2.1. Instalar Dependencias

```bash
cd backend-AppServicios
npm install
```

#### 2.2. Configurar Variables de Entorno

Crear archivo `.env` en la raíz de `backend-AppServicios`:

```bash
# .env
# ============================================
# CONFIGURACIÓN DE BASE DE DATOS
# ============================================
DB_SERVER=localhost
DB_NAME=nombre_base_datos
DB_USER=tu_usuario
DB_PASSWORD=tu_password

# ============================================
# CONFIGURACIÓN DE SERVIDOR
# ============================================
PORT=3000
# No se usa BASE_API_URL con /v1

# ============================================
# JWT (AUTENTICACIÓN)
# ============================================
JWT_SECRET=tu_secret_key_muy_segura_aqui
JWT_EXPIRES_IN=24h

# ============================================
# AZURE BLOB STORAGE
# ============================================
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...
AZURE_STORAGE_CONTAINER_NAME=fichaje-dev

# ============================================
# FTP (OPCIONAL)
# ============================================
FTP_HOST=ftp.ejemplo.com
FTP_USER=usuario_ftp
FTP_PASSWORD=password_ftp
FTP_PORT=21

# ============================================
# EMAIL (OPCIONAL)
# ============================================
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=tu_email@gmail.com
SMTP_PASSWORD=tu_password_app
```

> [!IMPORTANT] > **Nunca** subas el archivo `.env` al repositorio. Ya está incluido en `.gitignore`.

#### 2.3. Configurar Base de Datos

**Opción A: Usar base de datos existente**

Solicita al equipo:

- Servidor SQL Server
- Nombre de base de datos
- Credenciales de acceso

**Opción B: Crear base de datos local**

```sql
-- En SQL Server Management Studio (SSMS)
CREATE DATABASE FichajeDB;
GO

-- Ejecutar scripts de creación de tablas
-- (Solicitar scripts al equipo)
```

#### 2.4. Verificar Conexión

```bash
node -e "const {sequelize} = require('./config/dbConfig'); sequelize.authenticate().then(() => console.log('✅ Conexión exitosa')).catch(err => console.error('❌ Error:', err));"
```

#### 2.5. Iniciar Backend

```bash
npm start
```

Deberías ver:

```
Servidor Node.js corriendo en puerto 3000
```

### 3. Configurar Frontend

#### 3.1. Instalar Dependencias

```bash
cd ../front-AppServicios
npm install
```

> [!NOTE]
> La instalación puede tardar varios minutos debido al tamaño de Angular y sus dependencias.

#### 3.2. Configurar Variables de Entorno

Crear archivo `.env` en la raíz de `front-AppServicios`:

```bash
# .env
API_URL=http://localhost:3000
```

Verificar configuración en `src/environments/environment.ts`:

```typescript
export const environment = {
  production: false,
  apiUrl: "http://localhost:3000",
};
```

#### 3.3. Iniciar Frontend

```bash
npm start
# o
ng serve
```

Deberías ver:

```
** Angular Live Development Server is listening on localhost:4200 **
✔ Compiled successfully.
```

Abre tu navegador en `http://localhost:4200`

## 🗄️ Configuración de Base de Datos

### Estructura de Instancia SQL Server

```
SQL Server (KONGSERVER)
└── FichajeDB
    ├── Tablas
    │   ├── USUARIOS
    │   ├── EMPRESA
    │   ├── CONTROL_ASISTENCIAS
    │   ├── PROYECTOS
    │   └── ... (ver Model/init-models.js)
    └── Stored Procedures (si aplica)
```

### Configuración de Sequelize

El proyecto usa **Sequelize** como ORM. Configuración en `config/dbConfig.js`:

```javascript
const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASSWORD,
  {
    host: process.env.DB_SERVER,
    dialect: "mssql",
    timezone: "+00:00", // UTC
    dialectOptions: {
      options: {
        instanceName: "KONGSERVER", // Nombre de instancia SQL Server
        encrypt: false,
        enableArithAbort: true,
        trustServerCertificate: true,
      },
    },
    logging: false, // Cambiar a console.log para debug
  }
);
```

### Generar Modelos Automáticamente

Si necesitas regenerar los modelos desde la base de datos:

```bash
cd backend-AppServicios
node generate-models.js
```

Esto creará/actualizará archivos en `Model/` basándose en el schema de la BD.

## ☁️ Configuración de Azure Blob Storage

### Obtener Credenciales

1. Acceder al portal de Azure
2. Ir a Storage Account
3. Copiar Connection String

### Configurar en .env

```bash
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=tuaccount;AccountKey=tukey;EndpointSuffix=core.windows.net
AZURE_STORAGE_CONTAINER_NAME=fichaje-dev
```

### Crear Contenedor (si no existe)

```javascript
// Ejecutar una vez
const { BlobServiceClient } = require("@azure/storage-blob");
const blobServiceClient = BlobServiceClient.fromConnectionString(
  process.env.AZURE_STORAGE_CONNECTION_STRING
);
const containerClient = blobServiceClient.getContainerClient("fichaje-dev");
await containerClient.createIfNotExists();
```

## 🔐 Datos de Prueba

### Crear Usuario de Prueba

```sql
-- En SQL Server
INSERT INTO EMPRESA (nombre, activo)
VALUES ('Empresa Test', 1);

-- Obtener id_empresa generado
DECLARE @id_empresa INT = SCOPE_IDENTITY();

-- Crear usuario admin
-- Password: "admin123" (hasheado con bcrypt)
INSERT INTO USUARIOS (nombre, email, password, rol, id_empresa, activo)
VALUES (
  'Admin Test',
  'admin@test.com',
  '$2b$10$ejemplo_hash_bcrypt_aqui',
  'admin',
  @id_empresa,
  1
);
```

O usar el endpoint de superadmin (si está configurado):

```bash
POST http://localhost:3000/empresa/crear-empresa
{
  "nombre": "Empresa Test",
  "usuario": "admin",
  "password": "admin123"
}
```

### Login de Prueba

```bash
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "email": "admin@test.com",
  "password": "admin123"
}
```

Respuesta:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "nombre": "Admin Test",
    "empresa": 1,
    "rol": "admin"
  }
}
```

## 🛠️ Herramientas Recomendadas

### VS Code Extensions

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "angular.ng-template",
    "ms-mssql.mssql",
    "humao.rest-client",
    "ms-azuretools.vscode-azurestorage"
  ]
}
```

### SQL Server Management Studio (SSMS)

Descarga: https://aka.ms/ssmsfullsetup

Útil para:

- Explorar base de datos
- Ejecutar queries
- Ver estructura de tablas
- Debug de stored procedures

### Postman / REST Client

Para probar endpoints de la API:

**Ejemplo con REST Client (VS Code):**

Crear archivo `api-tests.http`:

```http
### Login
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "email": "admin@test.com",
  "password": "admin123"
}

### Obtener proyectos (requiere token)
GET http://localhost:3000/proyectos
Authorization: Bearer {{token}}
```

## 🐛 Troubleshooting

### Error: "Cannot connect to SQL Server"

**Solución:**

1. Verificar que SQL Server esté corriendo
2. Verificar nombre de instancia (`KONGSERVER`)
3. Habilitar TCP/IP en SQL Server Configuration Manager
4. Verificar firewall

```bash
# Verificar conexión
sqlcmd -S localhost\KONGSERVER -U tu_usuario -P tu_password
```

### Error: "JWT secret is not defined"

**Solución:**
Asegúrate de que `JWT_SECRET` esté en tu `.env`:

```bash
JWT_SECRET=una_clave_secreta_muy_larga_y_segura
```

### Error: "Azure Blob Storage connection failed"

**Solución:**

1. Verificar Connection String
2. Verificar que el contenedor existe
3. Verificar permisos de la cuenta

### Error: "Port 3000 already in use"

**Solución:**

```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# Cambiar puerto en .env
PORT=3001
```

### Error: "Module not found"

**Solución:**

```bash
# Limpiar node_modules y reinstalar
rm -rf node_modules package-lock.json
npm install
```

### Frontend no se conecta al Backend

**Solución:**

1. Verificar CORS en backend (`server.js`):

```javascript
app.use(cors());
```

2. Verificar URL en `environment.ts`:

```typescript
apiUrl: "http://localhost:3000";
```

## 📊 Verificar Instalación

### Checklist de Verificación

- [ ] Backend inicia sin errores
- [ ] Frontend inicia sin errores
- [ ] Conexión a SQL Server exitosa
- [ ] Login funciona
- [ ] Puedes ver datos en la aplicación
- [ ] Azure Blob Storage conectado (si aplica)

### Script de Verificación

Crear `verify-setup.js` en backend:

```javascript
const { sequelize } = require("./config/dbConfig");
const db = require("./Model");

async function verify() {
  try {
    // 1. Verificar conexión DB
    await sequelize.authenticate();
    console.log("✅ Conexión a base de datos OK");

    // 2. Verificar modelos
    const empresas = await db.EMPRESA.count();
    console.log(`✅ Modelos OK (${empresas} empresas en BD)`);

    // 3. Verificar variables de entorno
    const required = ["JWT_SECRET", "DB_NAME", "PORT"];
    const missing = required.filter((key) => !process.env[key]);
    if (missing.length > 0) {
      console.log(`❌ Faltan variables: ${missing.join(", ")}`);
    } else {
      console.log("✅ Variables de entorno OK");
    }

    console.log("\n🎉 Setup completo!");
  } catch (error) {
    console.error("❌ Error:", error.message);
  }
  process.exit(0);
}

verify();
```

Ejecutar:

```bash
node verify-setup.js
```

## 🚀 Siguiente Paso

Una vez configurado el entorno:

1. Lee [Introducción al Proyecto](intro.md) si no lo has hecho
2. Explora [Vista General de la Arquitectura](../arquitectura/overview.md)
3. Revisa [Convenciones de Código](../guias/convenciones.md)
4. ¡Empieza a desarrollar!

---

**¿Problemas?** Contacta al equipo técnico interno.
