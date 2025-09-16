# Troubleshooting

## Problemas Comunes

### 🔐 Problemas de Autenticación

#### Error: "Token JWT inválido"
**Síntomas:**
- Error 401 Unauthorized
- Mensaje: "Invalid JWT token"

**Soluciones:**
1. Verificar que el token no haya expirado
2. Renovar el token usando el endpoint `/auth`
3. Verificar que el token esté correctamente formateado

```bash
# Verificar token
curl -H "Authorization: Bearer tu_token" \
     https://api.svi.becomedigital.net/api/v1/aniCompliance
```

#### Error: "Credenciales incorrectas"
**Síntomas:**
- Error 401 al obtener token
- Mensaje: "Invalid credentials"

**Soluciones:**
1. Verificar usuario y contraseña
2. Contactar administrador para reset de contraseña
3. Verificar que la cuenta esté activa

### 🌐 Problemas de Conectividad

#### Error: "Timeout de conexión"
**Síntomas:**
- Error 408 Request Timeout
- Conexión lenta o intermitente

**Soluciones:**
1. Verificar conexión a internet
2. Aumentar timeout en configuración
3. Verificar firewall y proxy

```javascript
// Configurar timeout más alto
const config = {
  timeout: 60000, // 60 segundos
  retries: 3
};
```

#### Error: "DNS no resuelve"
**Síntomas:**
- Error de resolución DNS
- No se puede conectar al servidor

**Soluciones:**
1. Verificar configuración DNS
2. Usar IP directa si es necesario
3. Contactar soporte técnico

### 📊 Problemas de Datos

#### Error: "Datos de verificación incompletos"
**Síntomas:**
- Error 400 Bad Request
- Mensaje: "Missing required fields"

**Soluciones:**
1. Verificar que todos los campos requeridos estén presentes
2. Validar formato de datos
3. Revisar documentación de la API

```json
// Ejemplo de request válido
{
  "documentType": "CC",
  "documentNumber": "12345678",
  "firstName": "Juan",
  "lastName": "Pérez",
  "birthDate": "1990-01-01"
}
```

#### Error: "Formato de fecha inválido"
**Síntomas:**
- Error 400 Bad Request
- Mensaje: "Invalid date format"

**Soluciones:**
1. Usar formato ISO 8601: `YYYY-MM-DD`
2. Verificar que la fecha sea válida
3. Convertir formato si es necesario

### 🔄 Problemas de Sincronización

#### Error: "Webhook no recibido"
**Síntomas:**
- Verificaciones completadas pero no actualizadas
- Estado inconsistente entre sistemas

**Soluciones:**
1. Verificar URL del webhook
2. Comprobar que el servidor esté accesible
3. Revisar logs del webhook

```bash
# Verificar webhook
curl -X POST https://tu-servidor.com/webhook \
     -H "Content-Type: application/json" \
     -d '{"test": "webhook"}'
```

#### Error: "Estado desincronizado"
**Síntomas:**
- Estados diferentes entre Backend Humano y ANI Compliance
- Datos inconsistentes

**Soluciones:**
1. Ejecutar sincronización manual
2. Verificar logs de sincronización
3. Contactar soporte para resolución

## Herramientas de Diagnóstico

### 🔍 Verificación de Estado del Sistema

#### Health Check
```bash
# Verificar estado del sistema
curl https://backend-humano.becomedigital.net/health
```

**Response esperado:**
```json
{
  "status": "healthy",
  "services": {
    "database": "connected",
    "ani-compliance": "connected",
    "webhooks": "active"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

#### Verificación de APIs
```bash
# Verificar conectividad con ANI Compliance
curl -H "Authorization: Bearer tu_token" \
     https://api.svi.becomedigital.net/api/v1/health
```

### 📋 Logs y Monitoreo

#### Ver Logs del Sistema
```bash
# Ver logs recientes
tail -f /var/log/backend-humano/app.log

# Buscar errores específicos
grep "ERROR" /var/log/backend-humano/app.log
```

#### Métricas de Rendimiento
```bash
# Ver métricas del sistema
curl https://backend-humano.becomedigital.net/metrics
```

## Escalación de Problemas

### 🚨 Cuándo Contactar Soporte

Contacta al equipo de soporte técnico cuando:

1. **Errores persistentes** después de seguir las soluciones
2. **Problemas de seguridad** o acceso no autorizado
3. **Pérdida de datos** o corrupción
4. **Problemas de rendimiento** críticos
5. **Errores del servidor** (5xx)

### 📞 Información para Soporte

Al contactar soporte, incluye:

1. **Descripción del problema** detallada
2. **Pasos para reproducir** el error
3. **Logs relevantes** y mensajes de error
4. **Timestamp** del problema
5. **Usuario afectado** (si aplica)
6. **Screenshots** o capturas de pantalla

### 📧 Canales de Soporte

- **Email**: soporte@becomedigital.net
- **Slack**: #backend-humano-support
- **Teléfono**: +57 (1) 234-5678
- **Horario**: Lunes a Viernes, 8:00 AM - 6:00 PM (COT)

## Próximos Pasos

- [Configuración del Sistema](setup.md)
- [Panel de Administración](admin-panel.md)
- [Gestión de Identidades](identity-management.md)
