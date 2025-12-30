# Decisiones Arquitectónicas y Advertencias

Este documento detalla las decisiones técnicas importantes del proyecto y las advertencias críticas que todo desarrollador debe conocer.

## ⚠️ ADVERTENCIAS CRÍTICAS

### 🔴 Conexión Paralela a Base de Datos Access (LEGACY)

> [!CAUTION] > **PROBLEMA ARQUITECTÓNICO GRAVE**: El sistema mantiene una conexión paralela a una base de datos Microsoft Access heredada que coexiste con SQL Server.

**Contexto:**

- El proyecto originalmente usaba Access como base de datos principal
- Se migró a SQL Server, pero **NO se migró toda la funcionalidad**
- Algunos módulos aún consultan directamente la base de datos Access

**Ubicación del código problemático:**

```javascript
// Buscar en el código por referencias a Access o conexiones ODBC
// Principalmente en:
// - controllers/emailController.js (configuración SMTP desde Access)
// - Cualquier uso de mssql directo sin Sequelize
```

**Impacto:**

- ⚠️ **Inconsistencia de datos**: Datos duplicados entre Access y SQL Server
- ⚠️ **Mantenimiento complejo**: Dos fuentes de verdad
- ⚠️ **Escalabilidad limitada**: Access no es escalable
- ⚠️ **Riesgo de corrupción**: Access es propenso a corrupción con múltiples usuarios

**Recomendación:**

1. **Corto plazo**: Documentar todas las consultas a Access
2. **Medio plazo**: Migrar datos restantes a SQL Server
3. **Largo plazo**: Eliminar completamente la dependencia de Access

**Tablas afectadas conocidas:**

- `Configuracion_Empresa` (Access) vs `CONFIG_EMPRESA` (SQL Server)
- Configuración de correo SMTP

### 🟡 Consultas SQL Directas (Sin ORM)

> [!WARNING]
> Algunos controladores usan consultas SQL directas con el paquete `mssql` en lugar de Sequelize.

**Problema:**

```javascript
// ❌ MAL - Consulta directa
const pool = await mssql.connect(config);
const result = await pool.request().query("SELECT * FROM USUARIOS");

// ✅ BIEN - Usar Sequelize
const usuarios = await db.USUARIOS.findAll();
```

**Por qué es problemático:**

- No aprovecha las ventajas del ORM (validaciones, relaciones, etc.)
- Código más difícil de mantener
- Riesgo de SQL injection si no se parametriza correctamente
- Inconsistencia en el código

**Archivos afectados:**

- Buscar por `require('mssql')` en controllers

## 🏗️ Decisiones Arquitectónicas

### 1. Separación Frontend/Backend

**Decisión**: Arquitectura desacoplada con API REST

**Razones:**

- ✅ Permite desarrollo independiente de frontend y backend
- ✅ Facilita escalabilidad horizontal
- ✅ Posibilita múltiples clientes (web, móvil futuro)
- ✅ Mejor separación de responsabilidades

**Trade-offs:**

- ❌ Mayor complejidad inicial
- ❌ Necesidad de gestionar CORS
- ❌ Dos servidores en desarrollo

### 2. Sequelize como ORM

**Decisión**: Usar Sequelize para acceso a datos

**Razones:**

- ✅ Abstracción de la base de datos
- ✅ Migraciones y seeders
- ✅ Validaciones a nivel de modelo
- ✅ Relaciones entre modelos

**Trade-offs:**

- ❌ Curva de aprendizaje
- ❌ Queries complejas pueden ser difíciles
- ❌ Overhead de rendimiento en algunos casos

**Configuración específica:**

```javascript
// dbConfig.js
{
  dialect: "mssql",
  timezone: '+00:00',  // UTC
  dialectOptions: {
    options: {
      instanceName: "KONGSERVER",
      encrypt: false,
      trustServerCertificate: true
    }
  },
  logging: false  // Deshabilitado en producción
}
```

### 3. JWT para Autenticación

**Decisión**: JSON Web Tokens (JWT) en lugar de sesiones

**Razones:**

- ✅ Stateless (no requiere almacenamiento en servidor)
- ✅ Escalable horizontalmente
- ✅ Funciona bien con arquitectura REST
- ✅ Incluye información del usuario (empresa, rol)

**Implementación:**

```javascript
// Payload del token
{
  id: usuario.id,
  nombre: usuario.nombre,
  empresa: usuario.id_empresa,
  rol: usuario.rol
}
```

**Seguridad:**

- Token expira en 24 horas
- Refresh token disponible
- Secret key en variable de entorno

### 4. Multi-tenancy por Empresa

**Decisión**: Soft multi-tenancy con `id_empresa`

**Razones:**

- ✅ Una sola base de datos para todas las empresas
- ✅ Más simple de mantener
- ✅ Costos reducidos

**Implementación:**

- Cada tabla tiene `id_empresa`
- Middleware automático filtra por empresa del usuario
- Superadmin puede ver todas las empresas

**Seguridad:**

```javascript
// Siempre filtrar por empresa
const { empresa } = req.user;
const data = await db.TABLA.findAll({
  where: { id_empresa: empresa },
});
```

### 5. Azure Blob Storage para Archivos

**Decisión**: Usar Azure Blob Storage en lugar de sistema de archivos local

**Razones:**

- ✅ Escalable y confiable
- ✅ No depende del servidor local
- ✅ Facilita despliegue en múltiples servidores
- ✅ Backups automáticos

**Trade-offs:**

- ❌ Costo adicional
- ❌ Dependencia de servicio externo
- ❌ Latencia de red

**Uso:**

- Tickets de notas de gasto
- Documentos adjuntos
- Exportaciones de Excel

### 6. Angular Material + Bootstrap

**Decisión**: Combinar Angular Material con Bootstrap

**Razones:**

- ✅ Material para componentes complejos (tablas, modales)
- ✅ Bootstrap para layout y utilidades
- ✅ Diseño consistente

**Trade-offs:**

- ❌ Tamaño del bundle más grande
- ❌ Posibles conflictos de estilos
- ❌ Dos sistemas de diseño

### 7. js-joda para Manejo de Fechas

**Decisión**: Usar js-joda en lugar de Date nativo o moment.js

**Razones:**

- ✅ Inmutable (evita bugs)
- ✅ API clara y consistente
- ✅ Soporte de zonas horarias
- ✅ Compatible entre backend y frontend

**Configuración:**

```javascript
// Zona horaria: Europe/Madrid
import { LocalDateTime, ZoneId } from "@js-joda/core";
const ahora = LocalDateTime.now(ZoneId.of("Europe/Madrid"));
```

> [!IMPORTANT] > **CRÍTICO**: Sequelize con SQL Server y fechas es complicado. Ver [Guía de Fechas con Sequelize](#guía-de-fechas-con-sequelize).

## 📅 Guía de Fechas con Sequelize

### Problema con SQL Server

SQL Server almacena fechas en formato local del servidor, pero Sequelize asume UTC.

**Configuración necesaria:**

```javascript
// En dbConfig.js
{
  timezone: '+00:00',  // Forzar UTC
  dialectOptions: {
    options: {
      useUTC: true
    }
  }
}
```

**En los modelos:**

```javascript
// Definir campos de fecha
{
  fecha: {
    type: DataTypes.DATE,
    get() {
      const rawValue = this.getDataValue('fecha');
      if (!rawValue) return null;

      // Convertir a js-joda
      return LocalDateTime.parse(rawValue.toISOString().slice(0, -1));
    }
  }
}
```

**Recursos útiles:**

- [Sequelize Timezone Issues](https://github.com/sequelize/sequelize/issues/854)
- [js-joda Documentation](https://js-joda.github.io/js-joda/)

## 🔒 Seguridad

### Buenas Prácticas Implementadas

✅ **Autenticación:**

- JWT con expiración
- Passwords hasheados con bcrypt (10 rounds)
- Refresh tokens

✅ **Autorización:**

- Middleware de roles
- Verificación por empresa (multi-tenancy)
- Rutas protegidas

✅ **Validación:**

- Validación de entrada en controllers
- Sanitización de datos

### ⚠️ Áreas de Mejora

❌ **Falta implementar:**

- Rate limiting
- HTTPS obligatorio en producción
- Validación más estricta de inputs
- Logs de auditoría
- Encriptación de datos sensibles en BD

## 🚀 Rendimiento

### Optimizaciones Implementadas

✅ **Backend:**

- Índices en tablas principales
- Paginación en listados grandes
- Caché de configuraciones (cache.js)

✅ **Frontend:**

- Lazy loading de módulos
- Service Worker (PWA)
- OnPush change detection en componentes críticos

### ⚠️ Áreas de Mejora

❌ **Pendiente:**

- Implementar Redis para caché
- Optimizar queries N+1 en Sequelize
- Comprimir respuestas HTTP (gzip)
- CDN para assets estáticos

## 🧪 Testing

> [!WARNING] > **CRÍTICO**: El proyecto actualmente NO tiene tests automatizados.

**Estado actual:**

- ❌ Sin tests unitarios
- ❌ Sin tests de integración
- ❌ Sin tests E2E

**Recomendación urgente:**

1. Implementar tests unitarios para controllers críticos
2. Tests de integración para flujos principales
3. Tests E2E para user journeys críticos

**Stack sugerido:**

- Backend: Jest + Supertest
- Frontend: Jasmine + Karma (ya configurado)

## 📊 Monitoreo y Logs

**Estado actual:**

- ✅ Morgan para logs HTTP (desarrollo)
- ✅ Console.log para errores
- ❌ Sin sistema de logging estructurado
- ❌ Sin monitoreo de errores (Sentry, etc.)

**Recomendación:**

- Implementar Winston para logging estructurado
- Integrar Sentry o similar para tracking de errores
- Monitoreo de rendimiento (New Relic, DataDog)

## 🔄 Migraciones y Versionado

**Base de datos:**

- Sequelize CLI configurado
- Carpeta `migrations/` disponible
- ⚠️ Pocas migraciones documentadas

**Recomendación:**

- Crear migraciones para todos los cambios de schema
- Documentar cambios en CHANGELOG.md
- Versionado semántico (SemVer)

## 📝 Resumen de Deuda Técnica

| Prioridad | Ítem                                | Impacto |
| --------- | ----------------------------------- | ------- |
| 🔴 ALTA   | Eliminar dependencia de Access      | Alto    |
| 🔴 ALTA   | Implementar tests                   | Alto    |
| 🟡 MEDIA  | Migrar queries directas a Sequelize | Medio   |
| 🟡 MEDIA  | Implementar logging estructurado    | Medio   |
| 🟢 BAJA   | Optimizar rendimiento               | Bajo    |
| 🟢 BAJA   | Implementar rate limiting           | Bajo    |

---

**Siguiente**: [Convenciones de Código](../guias/convenciones.md)
