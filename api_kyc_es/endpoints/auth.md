# Autenticación JWT

Endpoint para obtener el token de acceso JWT requerido para consumir un recurso de la API. Debes indicar el `grant_type` correspondiente al endpoint que vas a llamar; el valor exacto se documenta en la página de cada recurso.

## POST `/auth`

### Headers requeridos

```http
Content-Type: application/json
```

### Parámetros del request

| Campo | Tipo | Obligatorio | Descripción |
|-------|------|-------------|-------------|
| `client_id` | string | Sí | Identificador del cliente proporcionado por Become Digital |
| `client_secret` | string | Sí | Secreto del cliente proporcionado por Become Digital |
| `grant_type` | string | Sí | Alcance del token según el recurso a consumir. Consulta la documentación del endpoint destino |

```json
{
  "client_id": "tu_client_id",
  "client_secret": "tu_client_secret",
  "grant_type": "<grant_type_del_recurso>"
}
```

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/auth' \
--header 'Content-Type: application/json' \
--data '{
    "client_id": "demo_client_123",
    "client_secret": "demo_secret_456_ABCDEFGH",
    "grant_type": "<grant_type_del_recurso>"
}'
```

### Respuestas de la API

#### ✅ **200 - Autenticación exitosa**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "exp": 1703123456,
  "iat": 1703119856
}
```

#### ⚠️ **200 - Credenciales inactivas**
```json
{
  "msg": "credential inactive"
}
```

### Manejo de errores

#### **400 - Bad Request**

**Solicitud no es JSON:**
```json
{ "msg": "Missing JSON in request" }
```

**Falta client_id:**
```json
{ "msg": "Missing client_id parameter" }
```

**Falta client_secret:**
```json
{ "msg": "Missing client_secret parameter" }
```

**grant_type inválido:**
```json
{ "msg": "Invalid grant_type" }
```

#### **401 - Unauthorized**

**Credenciales inválidas:**
```json
{ "msg": "Invalid login details" }
```

#### **404 - Not Found**

**Empresa no encontrada:**
```json
{ "error": "Not Found" }
```

### Proceso de validación

La API sigue este flujo de validación:

1. **Validación de formato JSON** → Error 400 si la solicitud no es JSON válido
2. **Validación de parámetros** → Error 400 si faltan `client_id`, `client_secret` o `grant_type`
3. **Validación de grant_type** → Error 400 si el `grant_type` no es reconocido
4. **Validación de cliente** → Error 401 si el client_id no existe
5. **Validación de estado activo** → Respuesta 200 si las credenciales están inactivas
6. **Validación de empresa** → Error 404 si la empresa no existe
7. **Validación de contraseña** → Error 401 si el client_secret es incorrecto
8. **Autenticación exitosa** → Respuesta 200 con token JWT acotado al `grant_type` solicitado

### Información importante

- El token JWT contiene información del cliente, empresa y el alcance (`grant_type`) autorizado
- Los campos `exp` e `iat` son timestamps Unix para fecha de expiración y emisión
- El `grant_type` válido para cada recurso se indica en la documentación de ese endpoint
- Solicita un token distinto para cada recurso; un token emitido para un `grant_type` no sirve para otro
- Guarda el `access_token` para incluirlo en las solicitudes al endpoint asociado a ese `grant_type`