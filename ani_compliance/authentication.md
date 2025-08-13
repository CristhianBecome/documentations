# Autenticación

Para acceder a la API de Verificación ANI Compliance, es necesario autenticarse utilizando un token de acceso. Este endpoint de autenticación valida las credenciales del cliente y retorna un token JWT que debe ser incluido en todas las solicitudes posteriores.

## Flujo de autenticación

1. **Obtener credenciales:** Solicita tu `client_id` y `client_secret` al equipo de soporte
2. **Generar token:** Usa el endpoint `/auth` para obtener un JWT
3. **Usar token:** Incluye el JWT en el header `Authorization` de todas las requests

## Seguridad

- ⚠️ **Nunca expongas** el `client_secret` en código frontend
- 🔒 **Almacena de forma segura** los tokens de acceso
- ⏰ **Los tokens expiran** cada 1 hora, renuévalos automáticamente
- 🔄 **Implementa retry logic** para renovación automática

## Siguientes pasos

- [Ver endpoint de autenticación →](endpoints/auth.md)
- [Realizar primera verificación →](endpoints/verification.md)