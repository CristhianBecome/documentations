# Autenticación JWT

Endpoint para obtener el token de acceso requerido para todas las operaciones de la API.

## POST `/auth`

### Headers requeridos

```http
Content-Type: application/json
```

### Parámetros del request

```json
{
  "client_id": "tu_client_id",
  "client_secret": "tu_client_secret"
}
```

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/auth' \
--header 'Content-Type: application/json' \
--data '{
    "client_id": "demo_client_123",
    "client_secret": "demo_secret_456_ABCDEFGH"
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
2. **Validación de parámetros** → Error 400 si faltan client_id o client_secret
3. **Validación de cliente** → Error 401 si el client_id no existe
4. **Validación de estado activo** → Respuesta 200 si las credenciales están inactivas
5. **Validación de empresa** → Error 404 si la empresa no existe
6. **Validación de contraseña** → Error 401 si el client_secret es incorrecto
7. **Autenticación exitosa** → Respuesta 200 con token JWT

### Información importante

- El token JWT contiene información del cliente, empresa y permisos necesarios
- Los campos `exp` e `iat` son timestamps Unix para fecha de expiración y emisión
- Guarda el `access_token` para incluirlo en las solicitudes posteriores a la API