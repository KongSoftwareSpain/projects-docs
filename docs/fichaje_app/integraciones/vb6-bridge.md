# VB6 Bridge — Notificaciones Push

Puente Python compilado a EXE que permite a una aplicación **VB6** enviar **notificaciones push** a usuarios concretos a través de la API Node.js en Azure.

## Arquitectura

```
VB6 App
↓  (Shell + args)
bridge_api.exe  ←→  datos.mdb (Access: tabla configuracion_api)
↓  (HTTPS + x-vb6-api-key)
POST /api/vb6/push
↓
Node.js → WebPush VAPID → Navegadores de los usuarios
```

---

## Configuración previa

### 1. Generar la API Key desde el panel SuperAdmin

La API Key VB6 se genera **desde el panel SuperAdmin** de la aplicación web:

1. Accede como **superadmin** a la aplicación web
2. Selecciona la empresa para la que necesitas el bridge
3. En la sección **API Keys**, localiza la tarjeta **VB6**
4. Pulsa **Generar** para crear una nueva clave
5. **Copia la clave generada** — la necesitarás para el paso siguiente

> **Importante**: Cada empresa tiene su propia API Key VB6. Si regeneras la clave, la anterior queda invalidada inmediatamente y el bridge dejará de funcionar hasta que actualices el Access.

### 2. Crear la base de datos Access `datos.mdb`

El bridge lee las credenciales de un archivo Access protegido con contraseña. Crea un archivo `datos.mdb` con una tabla llamada **`configuracion_api`** con esta estructura:

| Campo     | Tipo  | Descripción                                                      |
| --------- | ----- | ---------------------------------------------------------------- |
| `Id`      | Autonumérico | Clave primaria                                              |
| `Api_Key` | Texto | La API Key VB6 generada desde el panel SuperAdmin                |
| `Api_Url` | Texto | URL base de la API en Azure, **con barra final**                 |

**Ejemplo de registro:**

| Id | Api_Key                                                          | Api_Url                                                                  |
| -- | ---------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 1  | `a3f82e1d9c4b7f2a0e5d8c1b6a4f9e2d7c3b8a5f1e4d7c2b9a6f3e0d8c5b2` | `https://fichaje-comercial-hpf2hxa0bhasgrib9.spaincentral-01.azurewebsites.net/` |

Ponle contraseña al archivo `.mdb`:
- En Access: **Herramientas → Seguridad → Contraseña de base de datos**

### 3. Ajustar rutas en `bridge_api.py`

Edita estas dos constantes al principio del archivo:

```python
MDB_PATH     = r".\datos.mdb"         # Ruta real del archivo .mdb
MDB_PASSWORD = "PASSWORD_ACCESS"       # Contraseña del .mdb
```

> **Nota**: La ruta `.\datos.mdb` es relativa al directorio donde se encuentra el EXE. Si prefieres una ruta absoluta: `r"C:\app\datos.mdb"`.

---

## Flujo completo (paso a paso)

```
1. SuperAdmin genera API Key VB6 para la empresa
        ↓
2. Se crea datos.mdb con tabla configuracion_api
   ┌──────────────────────────────────────────────────────┐
   │ Api_Key = (la clave generada)                        │
   │ Api_Url = https://tu-backend.azurewebsites.net/      │
   └──────────────────────────────────────────────────────┘
        ↓
3. Se compila bridge_api.py → bridge_api.exe
        ↓
4. Se despliega en la máquina del cliente:
   C:\app\bridge_api.exe
   C:\app\datos.mdb
        ↓
5. La app VB6 ejecuta:
   bridge_api.exe --usuarios 1,2 --asunto "Aviso" --cuerpo "Mensaje"
        ↓
6. El bridge lee Api_Key y Api_Url del Access
        ↓
7. POST /api/vb6/push con header x-vb6-api-key
        ↓
8. El backend valida la clave contra CONFIG_EMPRESA.vb6_api_key
        ↓
9. WebPush VAPID → Navegador del usuario
```

---

## Instalación (entorno de desarrollo)

Requiere Python 3.9+ y Microsoft Access Database Engine instalado.

```bash
pip install -r requirements.txt
```

### Probar sin compilar

```bash
python bridge_api.py --usuarios 1,2 --asunto "Aviso" --cuerpo "Mensaje de prueba"
python bridge_api.py --usuarios 5  --asunto "Alerta" --cuerpo "Revisad el parte" --url "/partes"
```

Salida esperada:

```json
{ "success": true, "ok": true, "enviados": 2 }
```

---

## Compilar a EXE con PyInstaller

```bash
pyinstaller --onefile bridge_api.py
```

El ejecutable queda en `dist\bridge_api.exe`. Distribúyelo junto al `datos.mdb` en la máquina del cliente.

> **Nota**: El EXE requiere que **Microsoft Access Database Engine** esté instalado en la máquina del cliente. Descarga: [Microsoft Access Database Engine 2016](https://www.microsoft.com/en-us/download/details.aspx?id=54920)

---

## Uso desde VB6

```vb
Private Function EnviarNotificacion(usuarios As String, asunto As String, cuerpo As String) As String
    Dim oShell As Object
    Set oShell = CreateObject("WScript.Shell")

    Dim cmdLine As String
    cmdLine = "C:\app\bridge_api.exe --usuarios " & usuarios & _
              " --asunto """ & asunto & """" & _
              " --cuerpo """ & cuerpo & """"

    Dim oExec As Object
    Set oExec = oShell.Exec(cmdLine)

    ' Esperar a que termine
    Do While oExec.Status = 0
        DoEvents
    Loop

    EnviarNotificacion = oExec.StdOut.ReadAll
End Function

' Ejemplo de llamada:
' resultado = EnviarNotificacion("1,2,3", "Aviso importante", "Revisad el parte del día")
' If InStr(resultado, """ok"": true") > 0 Then MsgBox "Notificación enviada"
```

---

## Manejo de errores

El EXE siempre imprime JSON por stdout:

| Resultado      | Salida                                                        |
| -------------- | ------------------------------------------------------------- |
| Éxito          | `{"success": true, "ok": true, "enviados": 2}`                |
| Error Access   | `{"ok": false, "error": "Error al leer Access: ..."}`         |
| Error red      | `{"ok": false, "error": "No se pudo conectar a la API: ..."}` |
| Error HTTP 401 | `{"ok": false, "error": "Error HTTP 401 de la API: ..."}`     |
| Timeout        | `{"ok": false, "error": "Tiempo de espera agotado..."}`       |

**Error 401 — Causas comunes:**
- La `Api_Key` en el Access no coincide con la generada en el panel SuperAdmin
- Se regeneró la clave desde el panel pero no se actualizó el Access del cliente
- La clave fue revocada

---

## Seguridad

- La clave `Api_Key` se lee del Access protegido con contraseña → nunca está hardcodeada en el EXE.
- Cada empresa tiene su propia API Key (almacenada en `CONFIG_EMPRESA.vb6_api_key`).
- El EXE usa HTTPS (Azure proporciona TLS automáticamente).
- El backend valida la clave en cada petición y la asocia a la empresa correspondiente.
- Se recomienda además restringir IPs en Azure Portal → Web App → Networking → Access Restrictions.

---

## Gestión de API Keys

Las API Keys VB6 se gestionan desde el **panel SuperAdmin** de la aplicación web:

| Acción     | Descripción                                                  |
| ---------- | ------------------------------------------------------------ |
| **Generar**    | Crea una nueva API Key para una empresa sin clave previa     |
| **Regenerar**  | Invalida la clave anterior y genera una nueva                |
| **Revocar**    | Elimina la clave — el bridge deja de funcionar               |
| **Copiar**     | Copia la clave al portapapeles para pegarla en el Access     |

> Tras regenerar o revocar, **actualiza inmediatamente** el campo `Api_Key` en el `datos.mdb` del cliente.
