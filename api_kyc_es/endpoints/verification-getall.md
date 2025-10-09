# Verificación de Identidad - Listar Todas

Este endpoint devuelve todas las verificaciones pertenecientes a la empresa (cliente) que está actualmente autenticada.

## GET `/identity`

### Headers requeridos

```http
Authorization: Bearer <tu_jwt_token>
```

### Parámetros de consulta (query params)

| Parámetro | Tipo | Requerido | Descripción | Valor por defecto |
|-----------|------|-----------|-------------|-------------------|
| `page` | number | No | Número de página a consultar | 1 |
| `per_page` | number | No | Cantidad de resultados por página | 10 |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/identity?page=1&per_page=10' \
--header 'Authorization: Bearer <tu_jwt_token>'
```

### Respuesta de la API

#### ✅ **200 OK - Listado exitoso**

```json
{
  "data": [
    {
      "documentType": "national-id",
      "fullName": null,
      "documentID": null,
      "verification_status": "pending",
      "created_at": "Fri, 24 Jul 2020 19:10:59 GMT"
    },
    {
      "documentType": "national-id",
      "fullName": null,
      "documentID": null,
      "verification_status": "error",
      "created_at": "Sat, 25 Jul 2020 15:40:00 GMT"
    },
    {
      "documentType": "national-id",
      "fullName": "JUAN CARLOS GARCIA LOPEZ",
      "documentID": "12345678",
      "verification_status": "completed",
      "created_at": "Sat, 25 Jul 2020 15:40:00 GMT"
    },
    {
      "documentType": "driving-license",
      "fullName": "MARIA FERNANDA RODRIGUEZ",
      "documentID": "87654321",
      "verification_status": "completed",
      "created_at": "Sat, 25 Jul 2020 19:50:24 GMT"
    },
    {
      "documentType": "passport",
      "fullName": "PEDRO MARTINEZ SANCHEZ",
      "documentID": "AB123456",
      "verification_status": "completed",
      "created_at": "Sat, 25 Jul 2020 19:50:24 GMT"
    }
  ]
}
```

## Descripción de campos de respuesta

Cada elemento en el array `data` contiene:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `documentType` | string | Tipo de documento verificado: `national-id`, `passport`, `driving-license`, `proof-of-residency` |
| `fullName` | string/null | Nombre completo extraído del documento. `null` si aún no se ha procesado |
| `documentID` | string/null | Número de documento extraído. `null` si aún no se ha procesado |
| `verification_status` | string | Estado de la verificación: `pending`, `completed`, `error` |
| `created_at` | string | Fecha y hora de creación de la verificación |

## Interpretación de estados

### Estados de verificación

| Estado | Descripción |
|--------|-------------|
| `pending` | La verificación está en proceso de validación |
| `completed` | La verificación se completó exitosamente |
| `error` | Ocurrió un error durante el proceso de verificación |

### Valores null

Cuando `fullName` y/o `documentID` son `null`:

- **Si `verification_status` es `pending`:** Los datos aún no se han extraído (OCR en proceso)
- **Si `verification_status` es `error`:** No se pudieron extraer los datos del documento
- **Si `verification_status` es `completed`:** Es poco común, pero puede indicar que el documento no contenía estos datos legibles

## Paginación

### Parámetros de paginación

El endpoint soporta paginación mediante query parameters:

```
GET /identity?page=2&per_page=20
```

- **page:** Número de página (comienza en 1)
- **per_page:** Cantidad de resultados por página (recomendado: 10-50)

### Ejemplo de uso con paginación

```bash
# Primera página con 20 resultados
curl --location 'https://api.svi.becomedigital.net/api/v1/identity?page=1&per_page=20' \
--header 'Authorization: Bearer <tu_jwt_token>'

# Segunda página con 20 resultados
curl --location 'https://api.svi.becomedigital.net/api/v1/identity?page=2&per_page=20' \
--header 'Authorization: Bearer <tu_jwt_token>'
```

## Manejo de errores

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

**Causa:** No se incluyó el token JWT en el header o el token es inválido/expirado.

**Solución:** 
1. Verifica que incluiste el header `Authorization: Bearer <token>`
2. Si el token expiró, genera uno nuevo en el endpoint [/auth](auth.md)

## Casos de uso

### 1. Dashboard de verificaciones

Consultar periódicamente este endpoint para mostrar el estado de todas las verificaciones en un panel administrativo.

### 2. Auditoría y reportes

Obtener listado completo de verificaciones realizadas en un período específico.

### 3. Monitoreo de procesos pendientes

Identificar verificaciones que están en estado `pending` por mucho tiempo o que fallaron con `error`.

### 4. Búsqueda de verificaciones

Aunque este endpoint no soporta filtros nativos, puedes obtener todas las verificaciones y filtrarlas del lado del cliente.

## Ejemplo de respuesta vacía

Si no hay verificaciones para la empresa:

```json
{
  "data": []
}
```

## Buenas prácticas

1. **Usa paginación:** No consultes todas las verificaciones en una sola request si tienes muchos registros
2. **Cachea resultados:** Si no necesitas datos en tiempo real, considera cachear la respuesta
3. **Filtra del lado del cliente:** Aplica filtros por estado, fecha o tipo de documento después de recibir la respuesta
4. **Manejo de errores:** Implementa lógica de reintento para tokens expirados

## Información adicional

### Ordenamiento

Los resultados se retornan ordenados por fecha de creación (`created_at`), de más reciente a más antiguo.

### Límites

No hay un límite documentado para el parámetro `per_page`, pero se recomienda no exceder 100 resultados por página para mantener tiempos de respuesta óptimos.

## Siguientes pasos

- [Consultar detalle de una verificación →](verification-results.md)
- [Crear nueva verificación →](verification-add.md)
- [Consultar configuración de contrato →](contract.md)

