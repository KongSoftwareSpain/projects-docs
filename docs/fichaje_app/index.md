# Documentación del Proyecto Fichaje

Bienvenido a la documentación del sistema de gestión de fichajes, proyectos y servicios empresariales.

## 📋 Índice General

### 🏗️ Arquitectura

- [Vista General de la Arquitectura](arquitectura/overview.md) - Descripción completa de la arquitectura del sistema
- [Infraestructura y Despliegue](arquitectura/infraestructura.md) - Topología de servidores y bases de datos
- [Decisiones Arquitectónicas](arquitectura/decisiones.md) - Decisiones técnicas importantes y advertencias

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

## 🛠️ Stack Tecnológico

### Backend

- **Runtime**: Node.js
- **Framework**: Express 5
- **ORM**: Sequelize
- **Base de Datos**: SQL Server (MSSQL)
- **Autenticación**: JWT (JSON Web Tokens)
- **Almacenamiento**: Azure Blob Storage

### Frontend

- **Framework**: Angular 19
- **UI Components**: Angular Material + Bootstrap 5
- **Gráficos**: ngx-charts, FullCalendar
- **Gestión de Estado**: RxJS + Services
- **PWA**: Service Worker habilitado

## 📁 Estructura del Repositorio

```
Fichaje/
├── backend-AppServicios/     # API REST del backend
│   ├── controllers/          # Lógica de negocio
│   ├── routes/              # Definición de rutas
│   ├── Model/               # Modelos Sequelize
│   ├── middleware/          # Middlewares (auth, roles, etc.)
│   ├── config/              # Configuraciones
│   └── utils/               # Utilidades y helpers
│
├── front-AppServicios/      # Aplicación Angular
│   └── src/
│       └── app/
│           ├── components/  # Componentes reutilizables
│           ├── pages/       # Páginas principales
│           ├── services/    # Servicios HTTP
│           ├── guards/      # Guards de navegación
│           ├── interceptors/# Interceptores HTTP
│           └── interface/   # Interfaces TypeScript
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

**Última actualización**: Diciembre 2025
