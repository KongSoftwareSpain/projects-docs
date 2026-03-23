# Flutter API Key Authentication

## Descripción

El endpoint de fichaje de Flutter utiliza autenticación mediante API Key para identificar la empresa y asegurar que solo las aplicaciones autorizadas puedan realizar fichajes.

## Configuración

### 1. Generar API Key para una Empresa

Para generar un API key para una empresa, utiliza el endpoint de administración:

```http
POST /api/flutter-config/flutter-api-key/generate/:id_empresa
Authorization: Bearer <JWT_TOKEN>
```

**Respuesta exitosa:**

```json
{
  "success": true,
  "message": "API key generada exitosamente",
  "data": {
    "empresa": "Nombre de la Empresa",
    "api_key": "a1b2c3d4e5f6..."
  }
}
```

### 2. Configurar API Key en la App Flutter

En la aplicación Flutter, configura el API key en los headers de todas las peticiones:

```dart
final response = await http.post(
  Uri.parse('https://tu-servidor.com/api/flutter-fichaje/fichar'),
  headers: {
    'Content-Type': 'application/json',
    'X-Flutter-API-Key': 'a1b2c3d4e5f6...',
  },
  body: jsonEncode({
    'codigo_usuario': 'USER001',
  }),
);
```

## Endpoints de Administración

Todos los endpoints de administración requieren autenticación JWT.

### Obtener API Key Actual

```http
GET /api/flutter-config/flutter-api-key/:id_empresa
Authorization: Bearer <JWT_TOKEN>
```

**Respuesta:**

```json
{
  "success": true,
  "data": {
    "empresa": "Nombre de la Empresa",
    "api_key": "a1b2c3d4e5f6..." | null,
    "has_api_key": true | false
  }
}
```

### Generar Nuevo API Key

```http
POST /api/flutter-config/flutter-api-key/generate/:id_empresa
Authorization: Bearer <JWT_TOKEN>
```

Solo funciona si la empresa no tiene un API key existente.

### Regenerar API Key

```http
POST /api/flutter-config/flutter-api-key/regenerate/:id_empresa
Authorization: Bearer <JWT_TOKEN>
```

Genera un nuevo API key e invalida el anterior.

**Respuesta:**

```json
{
  "success": true,
  "message": "API key regenerada exitosamente. La clave anterior ha sido invalidada",
  "data": {
    "empresa": "Nombre de la Empresa",
    "api_key": "nuevo_api_key...",
    "had_previous_key": true
  }
}
```

### Revocar API Key

```http
DELETE /api/flutter-config/flutter-api-key/revoke/:id_empresa
Authorization: Bearer <JWT_TOKEN>
```

Elimina el API key de la empresa, deshabilitando el acceso desde Flutter.

## Endpoint de Fichaje

### Fichar (Check-in/Check-out)

```http
POST /api/flutter-fichaje/fichar
X-Flutter-API-Key: <API_KEY>
Content-Type: application/json

{
  "codigo_usuario": "USER001"
}
```

**Validaciones:**

1. El header `X-Flutter-API-Key` debe estar presente
2. El API key debe ser válido y pertenecer a una empresa
3. El `codigo_usuario` debe pertenecer a la empresa autenticada
4. El usuario debe tener `fichaje_activo = true`

**Respuesta exitosa:**

```json
{
  "success": true,
  "action": "entrada" | "salida",
  "usuario": {
    "id": 123,
    "nomapes": "Juan Pérez"
  },
  "fichaje": {
    "id": 456,
    "fecha": "2026-02-02",
    "hora_entrada": "2026-02-02T08:00:00.000Z",
    "hora_salida": null | "2026-02-02T17:00:00.000Z"
  }
}
```

**Errores posibles:**

- `401`: API key no proporcionada o inválida
- `404`: Usuario no encontrado o no pertenece a la empresa
- `403`: Usuario sin permiso para fichar
- `429`: Demasiadas solicitudes (rate limiting)

## Seguridad

- **API Key único por empresa**: Cada empresa tiene su propio API key
- **Validación de empresa**: El usuario debe pertenecer a la empresa del API key
- **Rate limiting**: 100 peticiones por usuario cada 15 minutos
- **HTTPS requerido**: Todas las peticiones deben usar HTTPS en producción

## Migración

Para aplicar los cambios en la base de datos:

```bash
npx sequelize-cli db:migrate
```

Esto añadirá el campo `flutter_api_key` a la tabla `CONFIG_EMPRESA`.
