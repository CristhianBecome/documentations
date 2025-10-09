# Endpoints

La API de Verificación KYC de Become Digital proporciona los siguientes grupos de endpoints:

## Autenticación

- **POST** `/auth` - Obtener token de acceso JWT

## KYC - Verificación de Identidad

- **POST** `/newIdentity` - Crear nueva verificación de identidad
- **GET** `/identity/<user_id>` - Consultar resultado de verificación por user_id
- **GET** `/identity` - Listar todas las verificaciones de la empresa
- **POST** `/matches` - Re-verificación mediante cotejo facial
- **POST** `/matches/check` - Consultar resultados de re-verificación
- **GET** `/contract/<contract_id>` - Consultar configuración de contrato

## Gov Checks - Verificaciones Gubernamentales

### Colombia
- **POST** `/aniCompliance` - Verificar cédula de ciudadanía con registros ANI

## Servicios Complementarios

### Telco
- **POST** `/telcoValidation` - Validar número telefónico

### Email
- **POST** `/email` - Validar correo electrónico

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
2. [Crear verificación de identidad](verification-add.md)
3. [Consultar resultados](verification-results.md)

### Verificación KYC
- [Crear verificación](verification-add.md)
- [Consultar resultado individual](verification-results.md)
- [Listar todas las verificaciones](verification-getall.md)
- [Re-verificación (autenticación)](re-verification.md)
- [Resultados de re-verificación](re-verification-results.md)
- [Consultar contrato](contract.md)

### Servicios Adicionales
- [ANI Colombia](ani-compliance.md)
- [Validación Telco](telco.md)
- [Validación Email](email.md)

