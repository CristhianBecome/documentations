# Webhooks - Configuración por contrato

Manual para configurar la URL de webhook y opciones adicionales (headers y parámetros) a nivel de contrato.

Cuando una verificación se completa, el sistema envía un `POST` a la URL configurada. La configuración opcional se guarda en la columna `webhook_config` (JSONB) del contrato.

Los webhooks permiten recibir notificaciones automáticas al completar una verificación, sin necesidad de polling constante.

> **Importante:** El webhook es una **notificación**. Después de recibirlo, **debes consultar** [GET /identity](verification-results.md) para obtener los datos completos de la verificación.

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| `webhook_url` | URL destino del webhook (columna existente en el contrato) |
| `webhook_config` | Objeto JSON con headers y parámetros opcionales |
| Evento disparador | `verification_completed` (al completar una verificación) |

Prefijo base: `/api/v1`

Autenticación: `Authorization: Bearer <token>`

Solo permite operar sobre contratos de la **compañía del token** (`company_id` del JWT).

### Permisos requeridos (grant types)

| Método | Grant types |
|--------|-------------|
| `GET` | `verification:get` |
| `PUT` | `verification:update` |

---

## Estructura de `webhook_config`

```json
{
  "headers": {
    "Authorization": "Bearer mi-token",
    "user": "456789"
  },
  "params": {
    "query": {
      "tenant": "acme",
      "partner_id": "abc123"
    },
    "body": {
      "environment": "production",
      "source": "become"
    }
  }
}
```

| Sección | Dónde se aplica en el envío |
|---------|----------------------------|
| `headers` | Headers HTTP adicionales (se fusionan con los defaults) |
| `params.query` | Query string en la URL (`?tenant=acme&partner_id=abc123`) |
| `params.body` | Campos extra fusionados en el JSON del body |

Si `webhook_config` es `null` o está vacío, el envío usa solo la configuración base (sin headers ni parámetros personalizados).

### Límites y validaciones

- Máximo **20 pares** clave-valor por sección (`headers`, `params.query`, `params.body`).
- Claves y valores deben ser **texto** (`string`), máximo **256 caracteres** cada uno.
- La `webhook_url` debe ser `http` o `https`, sin credenciales embebidas y sin apuntar a hosts internos (política SSRF).
- Headers bloqueados (no se envían): `Host`, `Content-Length`, `Transfer-Encoding`, `Connection`, `Keep-Alive`, `Proxy-Authorization`, `TE`, `Trailer`, `Upgrade`.

### Requisitos de la URL receptora

- Método `POST`, protocolo HTTPS recomendado
- Accesible públicamente
- Responder con código `200 OK` en menos de 30 segundos

---

## Payload enviado al completar verificación

El webhook se envía **únicamente cuando el proceso pasa a estado `completed`**.

### Sin `webhook_config` (o `null`)

```http
POST https://hooks.client.com/become
Content-Type: application/json
User-Agent: BecomeDigital-Webhook/1.0

{
  "event": "verification_completed",
  "contract_id": 123,
  "user_id": "uuid-del-usuario"
}
```

### Con configuración de ejemplo

```json
{
  "headers": { "user": "456789" },
  "params": {
    "query": { "tenant": "acme" },
    "body": { "environment": "production" }
  }
}
```

Resultado:

```http
POST https://hooks.client.com/become?tenant=acme
Content-Type: application/json
User-Agent: BecomeDigital-Webhook/1.0
user: 456789

{
  "event": "verification_completed",
  "contract_id": 123,
  "user_id": "uuid-del-usuario",
  "environment": "production"
}
```

**Campos base del payload:**

| Campo | Descripción |
|-------|-------------|
| `event` | Siempre `"verification_completed"` |
| `contract_id` | ID del contrato |
| `user_id` | Identificador único del usuario |

> **Nota:** Descripción completa de los campos de resultado en [GET /identity](verification-results.md).

---

## Flujo con webhook

1. Recibir webhook → Notificación de completado
2. Consultar `GET /identity/<user_id>` → Datos completos (**obligatorio**)
3. Procesar y almacenar

**Si el webhook falla:**

- No hay reintentos automáticos
- Usar [polling](verification-results.md#polling-requestconsulta-periódica) como respaldo
- Implementar monitoreo de webhooks

---

## GET `/contract/<contract_id>/webhook`

Consulta la configuración de webhook de un contrato propio.

**Grant type:** `verification:get`

### Headers requeridos

```http
Authorization: Bearer <tu_jwt_token>
```

### Ejemplo de solicitud

```bash
curl -s -X GET "https://api.svi.becomedigital.net/api/v1/contract/100/webhook" \
  -H "Authorization: Bearer $TOKEN"
```

### Respuesta 200

```json
{
  "contract_id": 100,
  "webhook_url": "https://hooks.client.com/become",
  "webhook_config": {
    "headers": { "user": "456789" },
    "params": {
      "query": { "tenant": "acme" },
      "body": { "environment": "production" }
    }
  }
}
```

- Si no hay URL real configurada (`aws` o vacía), `webhook_url` viene como `null`.
- Si no hay config extra, `webhook_config` viene como `null`.

### Errores

| HTTP | Causa |
|------|-------|
| 401 | Token ausente o inválido |
| 403 | Sin grant type o sin acceso al contrato |
| 404 | Contrato no existe o no pertenece a la compañía |

---

## PUT `/contract/<contract_id>/webhook`

Actualiza la URL y/o la configuración opcional del webhook.

**Grant type:** `verification:update`

**Body:** al menos uno de `webhook_url` o `webhook_config` es requerido.

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Ejemplo de solicitud

```bash
curl -s -X PUT "https://api.svi.becomedigital.net/api/v1/contract/100/webhook" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "webhook_url": "https://hooks.client.com/become",
    "webhook_config": {
      "headers": {
        "Authorization": "Bearer secret-token",
        "user": "456789"
      },
      "params": {
        "query": { "tenant": "acme" },
        "body": { "environment": "production" }
      }
    }
  }'
```

### Respuesta 200

```json
{
  "message": "Webhook configurado correctamente"
}
```

### Actualización parcial

Si envías solo una parte de `webhook_config`, se hace **merge** con lo existente:

- Enviar solo `headers` → actualiza headers, conserva `params` previos.
- Enviar solo `params.query` → actualiza query, conserva headers y `params.body` previos.

### Limpiar configuración

```json
{
  "webhook_config": null
}
```

Elimina toda la config opcional (headers y params). La `webhook_url` no se modifica si no la envías.

### Solo actualizar la URL

```json
{
  "webhook_url": "https://nueva-url.client.com/hook"
}
```

### Solo actualizar headers

```json
{
  "webhook_config": {
    "headers": { "user": "456789" }
  }
}
```

### Errores

| HTTP | Causa |
|------|-------|
| 400 | Falta `webhook_url` y `webhook_config`, URL no permitida, o `webhook_config` inválido |
| 401 | Token ausente o inválido |
| 403 | Sin grant type o sin acceso al contrato |
| 404 | Contrato no existe o no pertenece a la compañía |

---

## Mejores prácticas

- Configurar webhook en lugar de polling constante
- Validar el origen de los webhooks (token en header, parámetros propios, etc.)
- Responder `200 OK` de forma rápida y procesar de forma asíncrona en tu lado
- Tener polling como respaldo si el webhook falla
- No exponer secretos en logs; usar HTTPS

## Siguientes pasos

- [Crear verificación de identidad →](verification-add.md)
- [Consultar resultados de verificación →](verification-results.md)
