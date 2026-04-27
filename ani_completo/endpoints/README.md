# Endpoints

La API ANI (Registraduría) expone los siguientes endpoints principales:

## Autenticación
- **POST** `/auth` - Obtener token de acceso JWT

## Verificación
- **POST** `/aniValidation` - Consultar datos de cédula en registros oficiales (ANI / Registraduría)

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
