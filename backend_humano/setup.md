# Configuración del Sistema

## Requisitos Previos

Antes de configurar Backend Humano, asegúrate de tener:

- Acceso a la API de ANI Compliance
- Credenciales de administrador
- Navegador web compatible (Chrome, Firefox, Safari)

## Configuración Inicial

### 1. Acceso al Sistema

1. Navega a la URL del panel de administración
2. Inicia sesión con tus credenciales de administrador
3. Completa la configuración inicial

### 2. Configuración de Integración

Para conectar con ANI Compliance:

```bash
# Configurar endpoint de ANI Compliance
API_ENDPOINT=https://api.svi.becomedigital.net/api/v1
JWT_TOKEN=tu_token_de_acceso
```

### 3. Configuración de Notificaciones

Configura las notificaciones para recibir alertas sobre:

- Verificaciones completadas
- Errores en el sistema
- Actividad sospechosa

## Próximos Pasos

Una vez completada la configuración, puedes:

1. [Acceder al Panel de Administración](admin-panel.md)
2. [Configurar Gestión de Identidades](identity-management.md)
3. [Probar la Integración con APIs](api-integration.md)
