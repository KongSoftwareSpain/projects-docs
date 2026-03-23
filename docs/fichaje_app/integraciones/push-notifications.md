# Guía de Uso - Notificaciones Push

## Configuración

### 1. Variable de Entorno

En el archivo `.env`, configura la URL de tu frontend:

```env
FRONTEND_URL="http://localhost:4200"
```

Para producción, cambia a tu dominio real:

```env
FRONTEND_URL="https://tu-dominio.com"
```

## Uso Básico

### Enviar notificación con valores por defecto

```javascript
const { sendPushToUsers } = require("./controllers/pushBrowserController");

// Enviar a un usuario específico
await sendPushToUsers(
  123, // ID del usuario
  "Título", // Título
  "Mensaje de prueba", // Cuerpo
);

// Enviar a múltiples usuarios
await sendPushToUsers(
  [123, 456, 789], // Array de IDs
  "Título",
  "Mensaje",
);

// Enviar a TODOS los usuarios suscritos
await sendPushToUsers(
  null, // null = todos
  "Título",
  "Mensaje",
);
```

### Personalizar URL de destino

```javascript
await sendPushToUsers(
  123,
  "Nueva Vacación",
  "Tu solicitud ha sido aprobada",
  "/vacaciones", // URL al hacer clic
);
```

### Personalizar imágenes y vibración

```javascript
await sendPushToUsers(
  123,
  "Alerta Importante",
  "Requiere tu atención",
  "/alertas",
  {
    icon: "https://ejemplo.com/custom-icon.png",
    badge: "https://ejemplo.com/custom-badge.png",
    vibrate: [200, 100, 200, 100, 200], // Patrón personalizado
  },
);
```

## Formato del Payload

El payload que se envía tiene esta estructura:

```json
{
  "notification": {
    "title": "Título de la notificación",
    "body": "Cuerpo del mensaje",
    "icon": "http://localhost:4200/assets/img/LogoKong-cortada.png",
    "badge": "http://localhost:4200/assets/icons/maskable_icon_x48.png",
    "vibrate": [100, 50, 100],
    "data": {
      "url": "/inicio"
    }
  }
}
```

## Valores por Defecto

Si no especificas las opciones, se usan estos valores:

- **icon**: `${FRONTEND_URL}/assets/img/LogoKong-cortada.png`
- **badge**: `${FRONTEND_URL}/assets/icons/maskable_icon_x48.png`
- **vibrate**: `[100, 50, 100]`
- **url**: `/inicio`

## Ejemplos de Uso en Otros Controllers

### Ejemplo 1: Notificar aprobación de vacaciones

```javascript
// En vacacionesController.js
const { sendPushToUsers } = require("./pushBrowserController");

exports.aprobarVacacion = async (req, res) => {
  // ... lógica de aprobación ...

  await sendPushToUsers(
    vacacion.id_usuario,
    "Vacación Aprobada",
    `Tu solicitud de vacaciones del ${fechaInicio} al ${fechaFin} ha sido aprobada`,
    "/vacaciones",
  );

  res.json({ success: true });
};
```

### Ejemplo 2: Notificar nueva nota de gasto

```javascript
// En notaGastoController.js
const { sendPushToUsers } = require("./pushBrowserController");

exports.crearNotaGasto = async (req, res) => {
  // ... lógica de creación ...

  // Notificar a los autorizadores
  const autorizadores = await obtenerAutorizadores();

  await sendPushToUsers(
    autorizadores.map((a) => a.id),
    "Nueva Nota de Gasto",
    `${usuario.nombre} ha enviado una nota de gasto por ${total}€`,
    "/notas-gasto/pendientes",
  );

  res.json({ success: true });
};
```

### Ejemplo 3: Broadcast a todos los usuarios

```javascript
// En anunciosController.js
const { sendPushToUsers } = require("./pushBrowserController");

exports.enviarAnuncio = async (req, res) => {
  const { titulo, mensaje } = req.body;

  await sendPushToUsers(
    null, // null = todos los usuarios
    titulo,
    mensaje,
    "/anuncios",
  );

  res.json({ message: "Anuncio enviado a todos los usuarios" });
};
```

## Notas Importantes

1. **FRONTEND_URL** debe estar configurado correctamente en `.env`
2. Las rutas de `icon` y `badge` deben ser **URLs absolutas** para que funcionen cuando la app está cerrada
3. Si un usuario no tiene suscripción activa, no recibirá la notificación
4. Las suscripciones caducadas (error 410/404) se desactivan automáticamente

## Seguridad y Ciclo de Vida

Las notificaciones push están diseñadas para ser **persistentes**. Esto significa que seguirán llegando al dispositivo aunque la sesión del navegador caduque o el usuario cierre la app, lo cual es ideal para dispositivos móviles personales.

### Reasignación Automática (Transmisión de Propiedad)

Para garantizar la privacidad, la propiedad de la suscripción cambia automáticamente en el momento del login.

1. **Al Iniciar Sesión (Login)**

Cuando un usuario inicia sesión, la aplicación reasigna la suscripción existente en ese navegador al usuario actual.

```javascript
// En el frontend (ejecutado en LoginComponent tras login exitoso)
await pushService.reassignSubscriptionOnLogin();
```

**¿Cómo funciona?**

- Si el dispositivo era de **Juan** y ahora entra **María**: El backend actualiza la suscripción para que ahora pertenezca a María. Juan deja de recibir notificaciones en este dispositivo inmediatamente.
- Si el usuario es el mismo: Simplemente se asegura de que la suscripción esté activa.

Este enfoque es ideal porque:

- **No interrumpe el servicio**: Las notificaciones siguen llegando aunque el token de 8h caduque.
- **Mantiene la privacidad**: En cuanto entra otra persona, el anterior "dueño" pierde el acceso.

---

## Solución de Problemas

### ✅ Verificar que las notificaciones se envían solo a usuarios específicos

El sistema ahora usa correctamente el operador `IN` de Sequelize para filtrar usuarios:

```javascript
// ✅ CORRECTO - Envía solo al usuario 123
await sendPushToUsers(123, "Título", "Mensaje");

// ✅ CORRECTO - Envía solo a los usuarios 10, 20 y 30
await sendPushToUsers([10, 20, 30], "Título", "Mensaje");

// ✅ CORRECTO - Envía a TODOS los usuarios activos
await sendPushToUsers(null, "Título", "Mensaje");

// ❌ INCORRECTO - Array vacío no enviará a nadie
await sendPushToUsers([], "Título", "Mensaje");
```

### 🔍 Cómo verificar el filtrado

Puedes ejecutar el script de prueba incluido:

```bash
node test-push-filter.js
```

Este script simula diferentes escenarios de filtrado y muestra qué usuarios recibirían las notificaciones.

### 🐛 Debug de consultas SQL

Si necesitas ver la consulta SQL generada, agrega logs temporales:

```javascript
const browsers = await PushBrowser.findAll({
  where: whereClause,
  logging: console.log, // Muestra la consulta SQL
});
```

### 📊 Verificar suscripciones en la BD

Para ver qué usuarios tienen suscripciones activas:

```sql
SELECT id_usuario, COUNT(*) as num_browsers
FROM PushBrowsers
WHERE isActive = 1
GROUP BY id_usuario;
```
