# Autenticación

Para acceder a la API AML Watchlists de Become Digital, es necesario autenticarse utilizando un token de acceso JWT. El endpoint de autenticación valida las credenciales del cliente y retorna un token que debe incluirse en todas las solicitudes posteriores.

## Proceso de autenticación

1. **Obtener credenciales:** Solicita tu `client_id` y `client_secret` al equipo de soporte de Become Digital
2. **Generar token:** Utiliza el endpoint `/auth` para obtener un token JWT
3. **Usar token:** Incluye el JWT en el header `Authorization: Bearer <token>` de todas las solicitudes

## Flujo de autenticación

```
Cliente → [POST /auth con credenciales] → API
API → [Valida credenciales] → Retorna JWT
Cliente → [Usa JWT en headers] → Accede a endpoints protegidos
```

## Consideraciones de seguridad

- ⚠️ **Nunca expongas** el `client_secret` en código del lado del cliente
- 🔒 **Almacena de forma segura** los tokens de acceso
- ⏰ **Los tokens expiran** por defecto después de **1 hora**
- 🔄 **Implementa renovación automática** de tokens al detectar expiración
- 🔑 **Restablecimiento de contraseñas** debe solicitarse al soporte técnico

## Tiempo de expiración del token

El token JWT tiene un tiempo de expiración predeterminado de **1 hora**, pero este límite puede ser modificado bajo petición al soporte técnico.

> **Nota importante:** Una vez **expirado** el **token**, se debe realizar nuevamente la solicitud al endpoint `/auth` con las mismas credenciales para su **renovación**.

## Entornos disponibles

Las credenciales de acceso (`client_id` y `client_secret`) son proporcionadas por el equipo de Become Digital para:

- **Entorno de pruebas (Testing)**
- **Entorno de producción (Production)**

Asegúrate de usar las credenciales correctas según el entorno en el que estés trabajando.

## Uso del token en las solicitudes

El token debe incluirse en el encabezado `Authorization` en las solicitudes posteriores:

```http
Authorization: Bearer <tu_jwt_token>
```

Todas las solicitudes a los endpoints protegidos (excepto `/auth`) deben incluir este header.

## Siguientes pasos

- [Ver endpoint de autenticación →](endpoints/auth.md)
- [Consultar listas AML Watchlists →](endpoints/watchlists.md)

