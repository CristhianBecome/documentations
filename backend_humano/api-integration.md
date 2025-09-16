# Integración con APIs

## Descripción General

Backend Humano se integra con múltiples APIs para proporcionar un sistema completo de gestión de identidades. La integración principal es con ANI Compliance, pero también soporta otras APIs de verificación.

## Integración con ANI Compliance

### Configuración

#### Variables de Entorno
```bash
# ANI Compliance API
ANI_API_BASE_URL=https://api.svi.becomedigital.net/api/v1
ANI_API_TOKEN=tu_jwt_token_aqui
ANI_API_TIMEOUT=30000
```

#### Headers Requeridos
```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Endpoints Utilizados

#### 1. Verificación de Identidad
```http
POST /aniCompliance
```

**Request:**
```json
{
  "documentType": "CC",
  "documentNumber": "12345678",
  "firstName": "Juan",
  "lastName": "Pérez",
  "birthDate": "1990-01-01"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "verificationId": "ver_12345",
    "status": "verified",
    "confidence": 0.95,
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

#### 2. Consulta de Estado
```http
GET /aniCompliance/{verificationId}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "verificationId": "ver_12345",
    "status": "completed",
    "result": "approved",
    "details": {
      "nameMatch": true,
      "documentValid": true,
      "birthDateMatch": true
    }
  }
}
```

### Webhooks

#### Configuración de Webhooks

Backend Humano puede recibir notificaciones de ANI Compliance:

```json
{
  "webhook_url": "https://backend-humano.becomedigital.net/webhooks/ani-compliance",
  "events": [
    "verification.completed",
    "verification.failed",
    "verification.pending"
  ]
}
```

#### Procesamiento de Webhooks

```javascript
// Ejemplo de procesamiento de webhook
app.post('/webhooks/ani-compliance', (req, res) => {
  const { event, data } = req.body;
  
  switch (event) {
    case 'verification.completed':
      await processVerificationCompleted(data);
      break;
    case 'verification.failed':
      await processVerificationFailed(data);
      break;
    default:
      console.log('Evento no manejado:', event);
  }
  
  res.status(200).json({ received: true });
});
```

## Otras Integraciones

### APIs de Verificación Adicionales

#### Verificación de Documentos
```http
POST /api/document-verification
```

#### Verificación Biométrica
```http
POST /api/biometric-verification
```

#### Verificación de Dirección
```http
POST /api/address-verification
```

### APIs de Terceros

#### Servicios de Correo
- **SendGrid**: Notificaciones por email
- **Twilio**: Notificaciones SMS

#### Servicios de Almacenamiento
- **AWS S3**: Almacenamiento de documentos
- **Google Cloud Storage**: Backup de datos

## Manejo de Errores

### Códigos de Error Comunes

| Código | Descripción | Acción |
|--------|-------------|---------|
| 400 | Bad Request | Verificar parámetros |
| 401 | Unauthorized | Renovar token |
| 403 | Forbidden | Verificar permisos |
| 404 | Not Found | Verificar endpoint |
| 500 | Internal Server Error | Contactar soporte |

### Retry Logic

```javascript
// Ejemplo de retry con backoff exponencial
async function callAPIWithRetry(apiCall, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await apiCall();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      const delay = Math.pow(2, attempt) * 1000; // 2s, 4s, 8s
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

## Monitoreo y Logs

### Métricas Importantes

- **Tasa de Éxito**: Porcentaje de verificaciones exitosas
- **Tiempo de Respuesta**: Latencia promedio de las APIs
- **Errores por Endpoint**: Frecuencia de errores por API
- **Uso de Webhooks**: Número de webhooks procesados

### Logs Estructurados

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "info",
  "service": "backend-humano",
  "operation": "verification.completed",
  "identity_id": "12345",
  "api": "ani-compliance",
  "duration_ms": 1250,
  "success": true
}
```

## Próximos Pasos

- [Troubleshooting](troubleshooting.md)
- [Configuración del Sistema](setup.md)
