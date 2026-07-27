# Endpoints

La API de Consulta de Migración Colombia proporciona los siguientes endpoints principales:

## Autenticación
- **POST** `/auth` - Obtener token de acceso JWT (`grant_type`: `standalone:migration_co`)

## Consulta de migración
- **POST** `/govchecks/co/migration/compliance` - Consulta de migración Colombia con comparación 1 a 1 de campos (`grant_type`: `standalone:migration_co`)

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
