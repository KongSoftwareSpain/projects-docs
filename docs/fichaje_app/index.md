# Documentación del Proyecto Fichaje

Bienvenido a la documentación del sistema de gestión de fichajes, proyectos y servicios empresariales.

## 📋 Índice General

### 🏗️ Arquitectura

- [Vista General de la Arquitectura](arquitectura/overview.md) - Descripción completa de la arquitectura del sistema
- [Infraestructura y Despliegue](arquitectura/infraestructura.md) - Topología de servidores y bases de datos
- [Decisiones Arquitectónicas](arquitectura/decisiones.md) - Decisiones técnicas importantes y advertencias

### 📱 Integraciones Externas

- [Flutter Fichaje API](../backend-AppServicios/FLUTTER_FICHAJE_API.md) - API de fichaje para la app móvil Flutter
- [Flutter API Key Auth](../backend-AppServicios/FLUTTER_API_KEY_AUTH.md) - Gestión de API Keys para Flutter
- [VB6-Bridge](../vb6-bridge/README.md) - Puente Python para enviar push desde aplicaciones VB6
- [Push Notifications](../backend-AppServicios/PUSH_NOTIFICATIONS_USAGE.md) - Guía de uso de notificaciones push (VAPID)

### 📖 Guías

- [Convenciones de Código](guias/convenciones.md) - Estándares y convenciones del proyecto

### 🚀 Onboarding

- [Introducción al Proyecto](onboarding/intro.md) - Guía de inicio para nuevos desarrolladores
- [Configuración del Entorno](onboarding/entorno.md) - Setup del entorno de desarrollo

## 🎯 Descripción General del Proyecto

**Fichaje** es una aplicación empresarial completa para la gestión de:

- ⏰ Control de asistencia y fichajes de empleados
- 📊 Gestión de proyectos y órdenes de trabajo
- 📝 Partes de trabajo y seguimiento de actividades
- 💰 Notas de gasto
- 📄 Albaranes y facturación
- 🏖️ Solicitudes de vacaciones
- 📈 Estadísticas y reportes
- 🔔 Notificaciones push en tiempo real (VAPID/Web Push)
- 📲 App móvil Flutter para fichaje rápido (sin JWT)
- 🔗 Puente VB6-Bridge para integración con aplicaciones legacy

## 🛠️ Stack Tecnológico

### Backend

- **Runtime**: Node.js
- **Framework**: Express 5
- **ORM**: Sequelize
- **Base de Datos**: SQL Server (MSSQL)
- **Autenticación**: JWT (JSON Web Tokens) + API Keys (Flutter/VB6)
- **Push Notifications**: Web Push con VAPID (librería `web-push`)
- **Almacenamiento**: Azure Blob Storage

### Frontend

- **Framework**: Angular 19
- **UI Components**: Angular Material + Bootstrap 5
- **Gráficos**: ngx-charts, FullCalendar
- **Gestión de Estado**: RxJS + Services
- **PWA**: Service Worker habilitado (incluye push notifications)

### App Móvil

- **Framework**: Flutter/Dart
- **Autenticación**: API Key por empresa (header `X-Flutter-API-Key`)
- **Funcionalidad**: Fichaje rápido entrada/salida

### Integración Legacy

- **VB6-Bridge**: Script Python compilado a EXE con PyInstaller
- **Comunicación**: VB6 → bridge_api.exe → API REST → Web Push
- **Configuración**: Almacenada en Access (datos.mdb)

## 📁 Estructura del Repositorio

```
Fichaje/
├── backend-AppServicios/     # API REST del backend
│   ├── controllers/          # Lógica de negocio
│   ├── routes/              # Definición de rutas
│   ├── Model/               # Modelos Sequelize
│   ├── middleware/          # Middlewares (auth, roles, Flutter, VB6)
│   ├── config/              # Configuraciones (DB, push VAPID, FTP)
│   └── utils/               # Utilidades y helpers
│
├── front-AppServicios/      # Aplicación Angular (PWA)
│   └── src/
│       └── app/
│           ├── components/  # Componentes reutilizables
│           ├── pages/       # Páginas principales
│           ├── services/    # Servicios HTTP (incluye push.service.ts)
│           ├── guards/      # Guards de navegación
│           ├── interceptors/# Interceptores HTTP
│           └── interface/   # Interfaces TypeScript
│
├── vb6-bridge/              # Puente Python para apps VB6
│   ├── bridge_api.py        # Script principal
│   ├── requirements.txt     # Dependencias Python
│   └── README.md            # Documentación del bridge
│
└── documentacion/           # Esta documentación
```

## 🔗 Enlaces Rápidos

- **Repositorio**: [GitHub](https://github.com/tu-repo/fichaje)
- **Backend API**: `http://localhost:3000`
- **Frontend Dev**: `http://localhost:4200`

## 📞 Contacto y Soporte

Para preguntas o problemas, contacta al equipo de desarrollo interno.

---

**Última actualización**: Marzo 2026
