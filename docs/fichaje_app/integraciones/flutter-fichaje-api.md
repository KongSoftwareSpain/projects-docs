# Flutter Fichaje API - Documentación

## Descripción General

Este documento describe el nuevo punto de entrada para la aplicación Flutter que permite realizar fichajes (check-in/check-out) sin autenticación JWT. El sistema utiliza un `codigo_usuario` único para identificar a cada usuario.

## Authentication

This endpoint uses **API Key authentication** instead of JWT. Each company has a unique API key that must be included in the request headers.

### Required Header

```http
X-Flutter-API-Key: <your_company_api_key>
```

To obtain or manage API keys, see [Flutter API Key Auth](./flutter-api-key-auth.md).

## Endpoint

### POST `/api/flutter-fichaje/fichar`

Realiza automáticamente el fichaje de entrada o salida según el estado actual del usuario.

#### Request

**Headers:**

```
Content-Type: application/json
```

**Body:**

```json
{
  "codigo_usuario": "USER001"
}
```

**Parámetros:**

- `codigo_usuario` (string, requerido): Código único del usuario. Debe existir en la tabla USUARIOS.

#### Response

**Success (200 OK):**

Entrada:

```json
{
  "success": true,
  "action": "entrada",
  "usuario": {
    "id": 123,
    "nomapes": "Juan Pérez"
  },
  "fichaje": {
    "id": 456,
    "fecha": "2026-01-31",
    "hora_entrada": "2026-01-31T08:00:00.000Z",
    "hora_salida": null
  }
}
```

Salida:

```json
{
  "success": true,
  "action": "salida",
  "usuario": {
    "id": 123,
    "nomapes": "Juan Pérez"
  },
  "fichaje": {
    "id": 456,
    "fecha": "2026-01-31",
    "hora_entrada": "2026-01-31T08:00:00.000Z",
    "hora_salida": "2026-01-31T17:00:00.000Z"
  }
}
```

**Error Responses:**

400 Bad Request - Falta codigo_usuario:

```json
{
  "success": false,
  "message": "codigo_usuario es requerido"
}
```

404 Not Found - Usuario no encontrado:

```json
{
  "success": false,
  "message": "Usuario no encontrado"
}
```

403 Forbidden - Fichaje desactivado:

```json
{
  "success": false,
  "message": "Permiso para fichar desactivado"
}
```

429 Too Many Requests - Rate limit excedido:

```json
{
  "success": false,
  "message": "Demasiadas solicitudes. Por favor, intenta más tarde.",
  "resetTime": "2026-01-31T12:15:00.000Z"
}
```

500 Internal Server Error:

```json
{
  "success": false,
  "message": "Error del servidor"
}
```

## Rate Limiting

El endpoint está protegido por rate limiting para prevenir abuso:

- **Ventana de tiempo**: 15 minutos
- **Máximo de requests**: 100 por `codigo_usuario`
- **Headers de respuesta**:
  - `X-RateLimit-Limit`: Número máximo de requests permitidos
  - `X-RateLimit-Remaining`: Requests restantes en la ventana actual
  - `X-RateLimit-Reset`: Timestamp ISO 8601 cuando se resetea el límite

## Lógica de Fichaje Automático

El sistema determina automáticamente si debe hacer entrada o salida:

1. **Primera vez del día** → Crea entrada
2. **Ya hay entrada sin salida** → Registra salida
3. **Ya hay entrada con salida** → Crea nueva entrada

## Configuración de Base de Datos

### Campo `codigo_usuario`

Se ha añadido un nuevo campo a la tabla `USUARIOS`:

```sql
ALTER TABLE USUARIOS ADD codigo_usuario VARCHAR(100) NULL;
CREATE UNIQUE INDEX IX_USUARIOS_CODIGO_USUARIO ON USUARIOS(codigo_usuario);
```

**Características:**

- Tipo: VARCHAR(100)
- Nullable: Sí (para no romper registros existentes)
- Unique: Sí
- Indexed: Sí

### Migración

Para aplicar los cambios en la base de datos, ejecutar:

```bash
npx sequelize-cli db:migrate
```

Archivo de migración: `migrations/20260131124400-add-codigo-usuario-to-usuarios.js`

## Ejemplo de Uso (Flutter/Dart)

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<void> ficharUsuario(String codigoUsuario) async {
  final url = Uri.parse('${baseUrl}/api/flutter-fichaje/fichar');

  try {
    final response = await http.post(
      url,
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({'codigo_usuario': codigoUsuario}),
    );

    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      print('Fichaje exitoso: ${data['action']}');
      print('Usuario: ${data['usuario']['nomapes']}');
    } else if (response.statusCode == 429) {
      final data = jsonDecode(response.body);
      print('Rate limit excedido. Resetea en: ${data['resetTime']}');
    } else {
      print('Error: ${response.body}');
    }
  } catch (e) {
    print('Error de conexión: $e');
  }
}
```

## Seguridad

- ✅ **Rate limiting** implementado para prevenir abuso
- ✅ **Validación** de existencia de usuario
- ✅ **Verificación** de permisos de fichaje
- ⚠️ **Sin autenticación JWT** - El `codigo_usuario` debe mantenerse privado
- ⚠️ **Recomendación**: Implementar HTTPS en producción

## Archivos Modificados/Creados

### Nuevos Archivos

- `migrations/20260131124400-add-codigo-usuario-to-usuarios.js`
- `middleware/flutterRateLimitMiddleware.js`
- `controllers/flutterFichajeController.js`
- `routes/flutterFichajeRoutes.js`

### Archivos Modificados

- `Model/USUARIOS.js` - Añadido campo `codigo_usuario`
- `server.js` - Registradas nuevas rutas

## Notas Importantes

1. **No se modificó código existente** - Toda la funcionalidad es nueva
2. **Los endpoints JWT siguen funcionando** normalmente
3. **El rate limiting es por `codigo_usuario`**, no por IP
4. **La limpieza automática** del rate limiter se ejecuta cada 5 minutos
