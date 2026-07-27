# Endpoints

La API de Verificación ANI Compliance proporciona los siguientes endpoints principales:

## Autenticación
- **POST** `/auth` - Obtener token de acceso JWT (`grant_type`: `standalone:ani`)

## Verificación
- **POST** `/aniCompliance` - Verificar identidad consultando registros oficiales (`grant_type`: `standalone:ani`)

## URLs base por ambiente

| Ambiente | URL Base |
|----------|----------|
| Producción | `https://api.svi.becomedigital.net/api/v1` |


## Headers requeridos

Todas las solicitudes (excepto `/auth`) deben incluir:

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```