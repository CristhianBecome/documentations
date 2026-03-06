# Autenticación

Para acceder a la API de Consulta de Migración Colombia, es necesario autenticarse utilizando un token de acceso. El endpoint de autenticación valida las credenciales del cliente y retorna un token JWT que debe incluirse en todas las solicitudes posteriores.

## Proceso de autenticación

1. **Obtener credenciales:** Solicita tu `client_id` y `client_secret` al equipo de soporte
2. **Generar token:** Utiliza el endpoint `/auth` para obtener un token JWT
3. **Usar token:** Incluye el JWT en el header `Authorization` de todas las solicitudes

## Consideraciones de seguridad

- ⚠️ **Nunca expongas** el `client_secret` en código del lado del cliente
- 🔒 **Almacena de forma segura** los tokens de acceso
- ⏰ **Los tokens expiran** cada hora, implementa renovación automática
- 🔄 **Implementa lógica de reintento** para la renovación automática de tokens

## Siguientes pasos

- [Ver endpoint de autenticación →](endpoints/auth.md)
- [Realizar consulta de migración →](endpoints/migration-compliance.md)
