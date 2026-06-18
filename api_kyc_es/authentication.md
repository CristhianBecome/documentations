# Autenticación

Para acceder a la API de Verificación KYC de Become Digital, es necesario autenticarse utilizando un token de acceso JWT. El endpoint de autenticación valida las credenciales del cliente y el `grant_type` solicitado, y retorna un token con alcance limitado al recurso indicado. Ese token debe incluirse en las solicitudes al endpoint correspondiente.

## Proceso de autenticación

1. **Obtener credenciales:** Solicita tu `client_id` y `client_secret` al equipo de soporte de Become Digital
2. **Identificar el recurso:** Consulta la documentación del endpoint que vas a consumir para conocer el `grant_type` requerido
3. **Generar token:** Utiliza el endpoint `/auth` enviando credenciales y el `grant_type` específico de ese recurso
4. **Usar token:** Incluye el JWT en el header `Authorization: Bearer <token>` de las solicitudes a ese recurso

> El `grant_type` de cada endpoint se documenta en la página del recurso correspondiente. Solicita un token distinto para cada recurso que vayas a consumir.

## Flujo de autenticación

```
Cliente → [Consulta grant_type del endpoint] → [POST /auth con credenciales + grant_type] → API
API → [Valida credenciales y grant_type] → Retorna JWT
Cliente → [Usa JWT en headers] → Accede al endpoint protegido
```

## Consideraciones de seguridad

- ⚠️ **Nunca expongas** el `client_secret` en código del lado del cliente
- 🔒 **Almacena de forma segura** los tokens de acceso
- ⏰ **Los tokens expiran** por defecto después de **1 hora**
- 🔄 **Implementa renovación automática** de tokens al detectar expiración
- 🔑 **Restablecimiento de contraseñas** debe solicitarse al soporte técnico

## Tiempo de expiración del token

El token JWT tiene un tiempo de expiración predeterminado de **1 hora**, pero este límite puede ser modificado bajo petición al soporte técnico.

> **Nota importante:** Una vez **expirado** el **token**, se debe realizar nuevamente la solicitud al endpoint `/auth` con las mismas credenciales y el **mismo `grant_type`** del recurso que vas a consumir.

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

Todas las solicitudes a los endpoints protegidos (excepto `/auth`) deben incluir este header. El token debe haberse obtenido con el `grant_type` indicado en la documentación de cada recurso.

## Siguientes pasos

- [Ver endpoint de autenticación →](endpoints/auth.md)
- [Realizar primera verificación →](endpoints/verification-add.md)
- [Consultar resultados →](endpoints/verification-results.md)

