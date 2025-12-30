# Decisiones Arquitectónicas y Advertencias

Este documento detalla las decisiones técnicas importantes del proyecto y las advertencias críticas que todo desarrollador debe conocer.

## ⚠️ ADVERTENCIAS CRÍTICAS

### 🔴 Integración con ERP Antiguo en Access

> [!CAUTION] > **CONTEXTO ARQUITECTÓNICO IMPORTANTE**: El sistema SQL Server de esta aplicación se alimenta de datos provenientes de un **ERP antiguo basado en Microsoft Access**.

**Contexto real:**

- La empresa usa un ERP legacy en Access que sigue siendo la fuente principal de los datos
- SQL Server **NO reemplazó** a Access, sino que **se integra** con él
- Los datos fluyen desde el ERP Access hacia SQL Server
- Esta integración es **intencional y necesaria** para el funcionamiento del negocio

**Implicaciones en el diseño:**

> [!IMPORTANT]
> Muchas decisiones de diseño de la base de datos están **condicionadas por el ERP Access**. No te asustes por cómo está montado, es así por necesidad.

**Campos específicos del ERP:**

- `id_origen`: ID del registro en la base de datos Access original
- Nombres de campos: Mantienen nomenclatura del ERP legacy
- Estructura de tablas: Refleja el modelo del ERP Access
- Relaciones: Algunas están condicionadas por el sistema antiguo

**Por qué existe esta integración:**

- ✅ El ERP Access sigue en uso para otras funcionalidades del negocio
- ✅ Migración completa no es viable a corto plazo
- ✅ Necesidad de mantener sincronización de datos
- ✅ Evita duplicación de trabajo en el ERP legacy

**Ubicación del código de integración:**

```javascript
// Buscar en el código por referencias a Access o conexiones ODBC
// Principalmente en:
// - Sincronización de datos desde Access
// - Campos como id_origen que referencian IDs de Access
```

**Recomendación para desarrolladores:**

> [!NOTE] > **NO intentes "arreglar" o "normalizar" la estructura de la base de datos** sin consultar primero. Las decisiones de diseño están atadas al ERP legacy por razones de negocio.

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

**Decisión**: Transición de modelo Single-tenant (Legacy) a Soft Multi-tenancy (Futuro).

> [!NOTE]
> Actualmente coexisten ambos modelos. Ver [Infraestructura](../arquitectura/overview.md#🌍-infraestructura-y-despliegue-topología-actual).

**Razones del cambio a Multi-tenant (`kong1App`):**

- ✅ **Consolidación**: Evitar mantener N servidores para N clientes
- ✅ Una sola base de datos para todas las empresas
- ✅ Más simple de mantener y desplegar
- ✅ Costos reducidos

**Estado Actual (Híbrido):**

- **Legacy**: Servidores dedicados por cliente (`AppFichaje`, `ThrApp`, etc.)
- **Moderno**: Servidor multi-tenant (`kong1App`) alojando múltiples clientes

**Implementación del Multi-tenancy:**

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

### 7. Manejo de Fechas: Transición a date-fns

**Decisión actual**: Migración progresiva de js-joda a date-fns

> [!IMPORTANT] > **ESTADO DE TRANSICIÓN**: El proyecto está en proceso de migración de `js-joda` a `date-fns`. Actualmente coexisten ambas librerías.

**Contexto de la transición:**

- ✅ **Código nuevo**: Usar `date-fns` y `date-fns-tz`
- ⚠️ **Código legacy**: Aún usa `js-joda`
- 🔄 **Migración gradual**: Se está cambiando poco a poco

**Por qué date-fns:**

- ✅ Más ligero y modular
- ✅ Mejor rendimiento
- ✅ API más simple y funcional
- ✅ Mejor soporte de TypeScript
- ✅ Tree-shaking (reduce bundle size)

**Configuración recomendada (código nuevo):**

```javascript
// Backend - date-fns
const { format, parseISO, addDays } = require("date-fns");
const { zonedTimeToUtc, utcToZonedTime } = require("date-fns-tz");

// Zona horaria
const timezone = "Europe/Madrid";
const ahora = utcToZonedTime(new Date(), timezone);
```

```typescript
// Frontend - date-fns
import { format, parseISO, addDays } from "date-fns";
import { zonedTimeToUtc, utcToZonedTime } from "date-fns-tz";
import { es } from "date-fns/locale";

// Formatear fecha
const fechaFormateada = format(new Date(), "dd/MM/yyyy HH:mm", { locale: es });
```

**Código legacy (js-joda):**

```javascript
// Aún encontrarás esto en código antiguo
import { LocalDateTime, ZoneId } from "@js-joda/core";
const ahora = LocalDateTime.now(ZoneId.of("Europe/Madrid"));
```

**Guía para desarrolladores:**

> [!NOTE]
>
> - **Nuevo código**: Usa `date-fns`
> - **Modificando código existente**: Puedes mantener `js-joda` o migrar a `date-fns` si el cambio es significativo
> - **No mezcles**: En un mismo archivo/módulo, usa una sola librería

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

| Prioridad | Ítem                                | Impacto | Nota                  |
| --------- | ----------------------------------- | ------- | --------------------- |
| 🔴 ALTA   | Implementar tests                   | Alto    | Urgente               |
| 🟡 MEDIA  | Migrar queries directas a Sequelize | Medio   | Mejora mantenibilidad |
| 🟡 MEDIA  | Implementar logging estructurado    | Medio   | Mejora debugging      |
| 🟢 BAJA   | Optimizar rendimiento               | Bajo    | No crítico            |
| 🟢 BAJA   | Implementar rate limiting           | Bajo    | Seguridad adicional   |

> [!NOTE]
> La integración con Access **NO es deuda técnica**, es una decisión de negocio necesaria.

---

**Siguiente**: [Convenciones de Código](../guias/convenciones.md)
