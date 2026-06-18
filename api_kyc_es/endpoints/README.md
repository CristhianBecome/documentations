# Endpoints

La API de Verificación KYC de Become Digital proporciona los siguientes grupos de endpoints:

## Autenticación

- **POST** `/auth` - Obtener token de acceso JWT

## KYC - Verificación de Identidad

- **POST** `/newIdentity` - Crear nueva verificación de identidad (`grant_type`: `verification:create`)
- **GET** `/identity/<user_id>` - Consultar resultado de verificación por user_id (`grant_type`: `verification:get`)
- **POST** `/matches` - Re-verificación mediante cotejo facial (`grant_type`: `reverification:create`)
- **POST** `/matches/check` - Consultar resultados de re-verificación (`grant_type`: `reverification:create`)
- **GET** `/contract/<contract_id>` - Consultar configuración de contrato

## Gov Checks - Verificaciones Gubernamentales

### Colombia
- **POST** `/aniCompliance` - Verificar cédula de ciudadanía con registros ANI

## Servicios Complementarios

### Telco
- **POST** `/telcoValidation` - Validar número telefónico

### Email
- **POST** `/email` - Validar correo electrónico

## URL base

`https://api.svi.becomedigital.net/api/v1`

Producción y sandbox comparten esta misma URL base y rutas (`/auth`, `/newIdentity`, `/identity/…`, `/matches`, `/matches/check`). En sandbox usas **credenciales de prueba** y sufijos `-TEST-*` en `user_id` o `executionId`. Los escenarios sandbox de `/newIdentity` y `/matches` son **solo para API REST directa** (no SDK). Detalle en [Ambiente Sandbox](../sandbox.md).

## Headers requeridos

Todas las solicitudes (excepto `/auth`) deben incluir:

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

Para endpoints que envían archivos (como `/newIdentity` y `/matches`), usar:

```http
Content-Type: multipart/form-data
Authorization: Bearer <tu_jwt_token>
```

## Códigos de respuesta HTTP comunes

| Código | Descripción |
|--------|-------------|
| **200** | Solicitud exitosa |
| **201** | Recurso creado exitosamente |
| **202** | Solicitud aceptada, procesamiento pendiente |
| **400** | Error en la solicitud (parámetros faltantes o inválidos) |
| **401** | No autorizado (token faltante o inválido) |
| **404** | Recurso no encontrado |
| **412** | Precondición fallida (validación de datos) |
| **500** | Error interno del servidor |

## Navegación

### Primeros Pasos
1. [Autenticación JWT](auth.md)
2. [Ambiente Sandbox](../sandbox.md) (pruebas con `-TEST-1` / `-TEST-2` / `-TEST-3`)
3. [Crear verificación de identidad](verification-add.md)
4. [Consultar resultados](verification-results.md)

### Verificación KYC
- [Crear verificación](verification-add.md)
- [Consultar resultado individual](verification-results.md)
- [Re-verificación (autenticación)](re-verification.md)
- [Resultados de re-verificación](re-verification-results.md)
- [Consultar contrato](contract.md)

### Servicios Adicionales
- [ANI Colombia](ani-compliance.md)
- [Validación Telco](telco.md)
- [Validación Email](email.md)

