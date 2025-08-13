# Autenticación JWT

Endpoint para obtener el token de acceso requerido para todas las operaciones de la API.

## POST `/auth`

### Request

**Headers:**
```http
Content-Type: application/json
```

**Body:**
```json
{
  "client_id": "tu_client_id",
  "client_secret": "tu_client_secret"
}
```

### Ejemplo de Request

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/auth' \
--header 'Content-Type: application/json' \
--data '{
    "client_id": "demo_client_123",
    "client_secret": "demo_secret_456_ABCDEFGH"
}'
```

### Response

*Agregar ejemplo de respuesta*

### Errores comunes

| Código | Descripción |
|--------|-------------|
| 401 | Credenciales inválidas |
| 400 | Formato de request inválido |
| 429 | Demasiadas requests |