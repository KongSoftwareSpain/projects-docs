# Introducción al Proyecto Fichaje

¡Bienvenido al equipo de desarrollo de **Fichaje**! 👋

Esta guía te ayudará a entender el proyecto y comenzar a contribuir rápidamente.

## 🎯 ¿Qué es Fichaje?

**Fichaje** es una aplicación empresarial completa para la gestión integral de servicios y recursos humanos. Permite a las empresas:

- ⏰ **Control de Asistencia**: Fichaje de entrada/salida con geolocalización
- 📊 **Gestión de Proyectos**: Proyectos, órdenes de trabajo y asignaciones
- 📝 **Partes de Trabajo**: Seguimiento de tiempo por proyecto/actividad
- 💰 **Notas de Gasto**: Gestión de gastos con tickets digitales
- 📄 **Albaranes**: Facturación y cobros
- 🏖️ **Vacaciones**: Solicitudes y aprobaciones
- 📈 **Estadísticas**: Reportes y análisis de datos

## 🏢 Contexto del Negocio

### Multi-empresa (Multi-tenancy)

El sistema está diseñado para dar servicio a **múltiples empresas** desde una única instalación:

- Cada empresa tiene su propia configuración
- Los datos están completamente aislados entre empresas
- Un **superadmin** gestiona todas las empresas
- Cada empresa tiene sus propios **administradores**

### Roles de Usuario (Permisos)

El sistema maneja 3 roles principales de acceso:

#### 1. Superadmin

- Rol interno de nuestra empresa (desarrolladora/gestora).
- Controla el alta y gestión de las empresas clientes.

#### 2. Admin (Administrador de Empresa)

- Acceso al panel de gestión de la empresa.
- **Nota importante**: Solo disponible si la empresa **NO está vinculada al ERP** (o funcionalidad limitada si lo está).

#### 3. Usuario

- Acceso básico a la aplicación (fichaje, partes, etc.).

_(Ya no existen roles específicos de "Manager" o "RRHH")_

### Categorías Laborales

Más allá de los permisos de acceso (Roles), cada usuario tiene una **categoría profesional** que define su operativa diaria:

#### 1. Operario

- Realiza fichajes y partes de trabajo estándar.

#### 2. Técnico

- Tiene permisos adicionales sobre **Órdenes de Trabajo** (OTs).
- Puede realizar acciones técnicas específicas en las OTs asignadas.

#### 3. Administrativo

- Actualmente funcionalmente igual al resto, reservado para uso futuro.

## 🔑 Conceptos Clave

### 1. Fichaje y Asistencia

Los empleados registran su entrada y salida mediante la aplicación:

```
Usuario → Fichar Entrada → CONTROL_ASISTENCIAS (BD)
                         ↓
                    (si parte_auto = true)
                         ↓
                    PARTES_TRABAJO (BD)
```

**Características:**

- Geolocalización obligatoria
- Validación de horarios
- Creación automática de partes (opcional por empresa)

### 2. Proyectos y Órdenes de Trabajo

Jerarquía de trabajo:

```
PROYECTO
  └── ORDEN_TRABAJO (OT)
        ├── PARTES_TRABAJO (tiempo empleados)
        └── CABECERA (albarán/factura)
              ├── DETALLES_DOC (líneas)
              └── COBROS_DOC (pagos)
```

**Flujo típico:**

1. Se crea un **PROYECTO** (ej: "Instalación eléctrica Edificio A")
2. Se generan **ÓRDENES DE TRABAJO** (ej: "OT-001: Cableado planta 1")
3. Los empleados registran **PARTES DE TRABAJO** en las OTs
4. Al finalizar, se genera **ALBARÁN** con materiales y mano de obra
5. Se registran **COBROS** del cliente

### 3. Parte Auto

Configuración importante por empresa (`CONFIG_EMPRESA.parte_auto`):

- **true**: Al fichar entrada/salida, se crea automáticamente un parte de trabajo
- **false**: El empleado debe crear manualmente los partes

**Impacto en el código:**

```javascript
// En asistenciaController.js
const parteAuto = await checkParteAuto(empresa);
if (parteAuto) {
  // Crear parte automáticamente
  await db.PARTES_TRABAJO.create({...});
}
```

### 4. Notificaciones Push (VAPID)

El sistema envía notificaciones push a los navegadores de los usuarios usando el protocolo Web Push con VAPID:

```
Frontend (PWA)
  → Service Worker solicita permiso
  → Obtiene endpoint + claves (p256dh, auth)
  → POST /api/push-browser/subscribe → BD (PushBrowser)

Login  → POST /api/push-browser/reassign  (reasigna suscripción al usuario actual)
Logout → POST /api/push-browser/deactivate (desactiva suscripción)

Desde código backend:
  sendPushToUsers(userId, 'Título', 'Mensaje', '/ruta')
```

**Importante para desarrolladores:** Las suscripciones son **persistentes** (sobreviven al cierre del navegador). La privacidad se garantiza reasignando la suscripción en cada login.

### 5. Fichaje Flutter (App Móvil)

La app Flutter permite fichaje rápido sin JWT, usando API Key por empresa:

```
Flutter App
  → POST /api/flutter-fichaje/fichar
  → Header: X-Flutter-API-Key: <clave_empresa>
  → Body: { "codigo_usuario": "USER001" }
  → Respuesta: { action: "entrada" | "salida" }
```

- Cada empresa tiene su propia API Key (gestionada desde el panel admin web)
- El usuario se identifica por `codigo_usuario` (campo único en USUARIOS)
- Rate limit: 100 peticiones por usuario cada 15 minutos

### 6. VB6-Bridge (Legacy)

Permite a aplicaciones VB6 enviar notificaciones push:

```
VB6 → Shell("bridge_api.exe --usuarios 1,2 --asunto Aviso --cuerpo Mensaje")
      → Lee config de datos.mdb (Access protegido)
      → POST /api/vb6/push con x-vb6-api-key
      → Backend envía push via VAPID
```

### 7. Notas de Gasto

Los empleados pueden:

- Crear notas de gasto
- Añadir líneas de gasto
- Adjuntar tickets (Azure Blob Storage)
- Enviar a aprobación

**Estados:**

- `borrador`: En edición
- `pendiente`: Enviada a aprobación
- `aprobada`: Aprobada
- `rechazada`: Rechazada

## 📚 Módulos Principales

### Backend

| Módulo | Descripción | Archivos clave |
| --- | --- | --- |
| **auth** | Autenticación JWT | `authController.js`, `authMiddleware.js` |
| **asistencia** | Fichajes (web) | `asistenciaController.js`, `asistenciaRoutes.js` |
| **proyectos** | Gestión de proyectos | `proyectosController.js` |
| **partes** | Partes de trabajo | `parteController.js` |
| **albaran** | Albaranes y facturación | `albaranController.js` |
| **nota-gasto** | Notas de gasto | `notaGastoController.js` |
| **vacaciones** | Solicitudes de vacaciones | `vacacionesController.js` |
| **empresa** | Gestión de empresas | `empresaController.js` |
| **flutter-fichaje** | Fichaje desde app móvil (API Key) | `flutterFichajeController.js`, `flutterApiKeyMiddleware.js` |
| **flutter-config** | Gestión API Keys Flutter | `flutterConfigController.js` |
| **push-browser** | Notificaciones push VAPID | `pushBrowserController.js`, `config/push.js` |
| **vb6** | Bridge VB6 → Push | `vb6Controller.js`, `vb6ApiKeyMiddleware.js` |

### Frontend

| Módulo | Descripción | Ubicación |
| --- | --- | --- |
| **fichar-asistencia** | Pantalla de fichaje | `pages/fichar-asistencia/` |
| **page-proyectos** | Listado de proyectos | `pages/page-proyectos/` |
| **page-ote** | Órdenes de trabajo | `pages/page-ote/` |
| **listado-fichaje** | Historial de fichajes | `pages/listado-fichaje/` |
| **admin** | Panel de administración | `pages/admin/` |
| **push.service** | Servicio de push notifications | `services/push/push.service.ts` |

### Componentes externos

| Componente | Descripción | Ubicación |
| --- | --- | --- |
| **vb6-bridge** | Puente Python para apps VB6 legacy | `vb6-bridge/` |
| **Flutter app** | App móvil de fichaje rápido | Repo separado (consume API) |

## 🛠️ Stack Tecnológico

### Backend

- **Node.js** + **Express 5**: API REST
- **Sequelize**: ORM para SQL Server
- **JWT**: Autenticación stateless
- **bcrypt**: Hash de passwords
- **Azure Blob Storage**: Almacenamiento de archivos
- **date-fns**: Manejo de fechas (en transición desde js-joda)

### Frontend

- **Angular 19**: Framework principal
- **Angular Material**: Componentes UI
- **Bootstrap 5**: Layout y utilidades
- **RxJS**: Programación reactiva
- **ngx-charts**: Gráficos
- **FullCalendar**: Calendario de eventos
- **date-fns**: Manejo de fechas (en transición desde js-joda)

### Base de Datos

- **SQL Server**: Base de datos principal
- ⚠️ **Access** (Legacy): Algunas consultas aún usan Access (ver [Decisiones Arquitectónicas](../arquitectura/decisiones.md))

## 🚀 Primeros Pasos

### 1. Leer la Documentación

Orden recomendado:

1. ✅ Esta introducción (estás aquí)
2. 📖 [Configuración del Entorno](entorno.md)
3. 🏗️ [Vista General de la Arquitectura](../arquitectura/overview.md) - Incluye Flutter, VB6-Bridge y Push
4. ⚠️ [Decisiones Arquitectónicas](../arquitectura/decisiones.md) - **MUY IMPORTANTE**
5. 📝 [Convenciones de Código](../guias/convenciones.md)
6. 📲 [Flutter Fichaje API](../../backend-AppServicios/FLUTTER_FICHAJE_API.md) - Si trabajarás con la app móvil
7. 🔔 [Push Notifications](../../backend-AppServicios/PUSH_NOTIFICATIONS_USAGE.md) - Si trabajarás con notificaciones
8. 🔗 [VB6-Bridge](../../vb6-bridge/README.md) - Si trabajarás con la integración legacy

### 2. Configurar tu Entorno

Sigue la guía de [Configuración del Entorno](entorno.md) para:

- Instalar dependencias
- Configurar base de datos
- Configurar variables de entorno
- Ejecutar el proyecto localmente

### 3. Explorar el Código

**Backend - Comienza por:**

1. `server.js` - Punto de entrada
2. `routes/authRoutes.js` - Rutas de autenticación
3. `controllers/asistenciaController.js` - Lógica de fichaje
4. `Model/init-models.js` - Modelos de BD

**Frontend - Comienza por:**

1. `app.routes.ts` - Configuración de rutas
2. `services/auth.service.ts` - Servicio de autenticación
3. `pages/fichar-asistencia/` - Componente de fichaje
4. `interface/usuario.ts` - Interfaces principales

### 4. Hacer tu Primera Contribución

**Sugerencias para empezar:**

1. Corregir un bug pequeño
2. Mejorar documentación de código
3. Añadir validaciones
4. Escribir tests (actualmente no hay)

## ⚠️ Advertencias Importantes

### 🔴 Base de Datos Access (CRÍTICO)

> [!CAUTION]
> El sistema tiene una **dependencia legacy con Access** que coexiste con SQL Server. Lee [Decisiones Arquitectónicas](../arquitectura/decisiones.md#-conexión-paralela-a-base-de-datos-access-legacy) antes de modificar cualquier código relacionado con configuración de empresa o emails.

### 🟡 Manejo de Fechas

El manejo de fechas con Sequelize y SQL Server es complicado:

- Usar **date-fns** para código nuevo (js-joda en código legacy)
- Ver [Guía de Fechas](../arquitectura/decisiones.md#-guía-de-fechas-con-sequelize)

### 🟡 Multi-tenancy

**SIEMPRE** filtrar por empresa:

```javascript
// ✅ BIEN
const { empresa } = req.user;
const datos = await db.TABLA.findAll({
  where: { id_empresa: empresa },
});

// ❌ MAL - Expone datos de todas las empresas
const datos = await db.TABLA.findAll();
```

## 🧪 Testing

> [!WARNING]
> El proyecto actualmente **NO tiene tests automatizados**. Esto es deuda técnica prioritaria.

**Al desarrollar:**

- Prueba manualmente todos los casos
- Documenta los pasos de prueba
- Considera escribir tests para tu código nuevo

## 📞 Recursos y Ayuda

### Documentación Interna

- [Arquitectura](../arquitectura/overview.md)
- [Decisiones Técnicas](../arquitectura/decisiones.md)
- [Convenciones](../guias/convenciones.md)

### Documentación Externa

- [Sequelize Docs](https://sequelize.org/docs/v6/)
- [Angular Docs](https://angular.dev/)
- [Express Docs](https://expressjs.com/)
- [js-joda Docs](https://js-joda.github.io/js-joda/)

### Equipo

- Contacta al equipo técnico interno para dudas
- Revisa el código existente como referencia
- Pregunta antes de hacer cambios grandes

## 🎓 Glosario

| Término | Significado |
| --- | --- |
| **OT** | Orden de Trabajo |
| **Parte** | Parte de trabajo (registro de tiempo) |
| **Fichaje** | Registro de entrada/salida |
| **Parte Auto** | Creación automática de partes al fichar |
| **Multi-tenancy** | Múltiples empresas en un sistema |
| **Superadmin** | Administrador global del sistema |
| **VAPID** | Voluntary Application Server Identification — protocolo para push notifications |
| **VB6-Bridge** | Ejecutable Python que conecta apps VB6 con la API REST |
| **API Key** | Clave de autenticación alternativa a JWT (usada por Flutter y VB6) |
| **codigo_usuario** | Identificador único del usuario para fichaje Flutter (campo en USUARIOS) |
| **PushBrowser** | Tabla que almacena suscripciones push de navegadores |

## ✅ Checklist de Onboarding

- [ ] Leer toda la documentación
- [ ] Configurar entorno de desarrollo
- [ ] Ejecutar backend localmente
- [ ] Ejecutar frontend localmente
- [ ] Explorar la base de datos
- [ ] Hacer login como diferentes roles
- [ ] Probar flujo de fichaje
- [ ] Probar creación de proyecto
- [ ] Revisar código de un módulo completo
- [ ] Entender el flujo de push notifications (subscribe → reassign → send)
- [ ] Revisar cómo funciona el fichaje Flutter (API Key + codigo_usuario)
- [ ] Hacer tu primera contribución

---

**Siguiente**: [Configuración del Entorno](entorno.md)
