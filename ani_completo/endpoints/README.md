# Endpoints

La API ANI (Registraduría) expone los siguientes endpoints principales:

## Autenticación
- **POST** `/auth` - Obtener token de acceso JWT (`grant_type`: `standalone:ani`)

## Verificación
- **POST** `/aniValidation` - Consultar datos de cédula en registros oficiales (ANI / Registraduría). Bearer con scope `full_access`. Si el token trae `grant_type`, `standalone:ani` o `standalone:govchecks`.
- **POST** [`/aniValidationSB`](sandbox.md) - Sandbox. Catálogo fijo de documentos. No consulta registros en vivo.

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
