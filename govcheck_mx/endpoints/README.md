# Endpoints

La API CURP México (GovChecks) expone los siguientes endpoints principales:

## Autenticación
- **POST** `/auth` - Obtener token de acceso JWT (`grant_type`: `standalone:govchecks`)

## Validación CURP
- **POST** `/govchecks/mx/curp` - Validación CURP México (v1) (`grant_type`: `standalone:govchecks`)
- **POST** `/govchecks/mx/curp/v2` - Validación CURP México (v2) (`grant_type`: `standalone:govchecks`)

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

## Requisitos

- **Grant type:** `standalone:govchecks`
- **Contrato:** debe tener habilitada la validación CURP (`can_mexican_curp`)
