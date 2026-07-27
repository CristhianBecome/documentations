# Endpoints

La API ANI (Registraduría) expone los siguientes endpoints principales:

## Autenticación
- **POST** `/auth` - Obtener token de acceso JWT (`grant_type`: `standalone:ani`)

## Verificación
- **POST** `/aniValidation` - Consultar datos de cédula en registros oficiales (ANI / Registraduría) (`grant_type`: `standalone:ani`)

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
