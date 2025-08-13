# Endpoints

La API de Verificación ANI Compliance cuenta con los siguientes endpoints principales:

## Autenticación
- **POST** `/auth` - Obtener token de acceso JWT

## Verificación
- **POST** `/aniCompliance` - Verificar identidad con registros oficiales

## Base URLs

| Ambiente | URL Base |
|----------|----------|
| Producción | `https://api.svi.becomedigital.net/api/v1` |


## Headers requeridos

Todas las requests (excepto `/auth`) deben incluir:

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>