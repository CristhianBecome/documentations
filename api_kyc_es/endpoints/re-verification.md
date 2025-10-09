# Re-verificación (Autenticación Biométrica)

Una vez que un usuario ha completado el proceso de **Onboarding** (verificación inicial de identidad), puede autenticarse de forma rápida y segura a través de un **cotejo facial** y una **prueba de vida (liveness detection)**. Este procedimiento elimina la necesidad de presentar nuevamente el documento de identidad en cada transacción.

## POST `/matches`

### Descripción del servicio

Este endpoint permite verificar la identidad de un usuario previamente registrado comparando una nueva selfie con la almacenada durante el onboarding.

#### Dos componentes principales:

1. **Cotejo Facial (Facial Matching)**
   - Se compara la selfie tomada en la transacción actual con la selfie registrada en la etapa de Onboarding
   - Verifica que la persona que realiza la operación es la misma que se dio de alta inicialmente

2. **Prueba de Vida (Liveness Detection)**
   - Se realiza con el SDK en modo **pasivo** (sin movimientos forzados)
   - Garantiza que la selfie provenga de una persona real y no de una fotografía o video pregrabado

### Headers requeridos

```http
Content-Type: multipart/form-data
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del request (FormData)

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|-----------|-------------|
| **user_id** | string | ✅ | Identificador del usuario registrado en el onboarding |
| **contract_id** | string | ✅ | ID del contrato a utilizar |
| **image** | file | ✅ | Foto selfie actual del usuario |
| **liveness** | object | No | Datos de prueba de vida provistos por el SDK (opcional) |

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/matches' \
--header 'Authorization: Bearer <tu_jwt_token>' \
--form 'user_id="usuario_12345_1699123456"' \
--form 'contract_id="1"' \
--form 'image=@"/ruta/selfie_actual.jpg"'
```

### Respuesta de la API

#### ✅ **200 OK - Re-verificación completada**

```json
{
  "company": "YOUR_COMPANY_ACCOUNT_NAME",
  "confidence": 95.4115853309631348,
  "executionId": "f850eaa8-db4e-468f-a5a3-14cffbf975c0",
  "liveness": 99.89696502685547,
  "result": true,
  "user_id": "usuario_12345_1699123456"
}
```

### Descripción de campos de respuesta

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `company` | string | Nombre de la compañía responsable del proceso o cliente que integra el servicio |
| `confidence` | number | Nivel de confianza obtenido en la comparación facial (0-100) |
| `executionId` | string | Identificador único que vincula esta respuesta con la solicitud. Úsalo para consultar resultados |
| `liveness` | number | Resultado de la prueba de vida (0-100). Mayor valor = mayor confianza de que es una persona real |
| `result` | boolean | `true` = Verificación exitosa, `false` = Verificación fallida |
| `user_id` | string | Identificador del usuario para el cual se realizó la validación |

## Interpretación de resultados

### ✅ Autenticación exitosa

```json
{
  "result": true,
  "confidence": 95.5,
  "liveness": 99.8
}
```

**Interpretación:**
- La persona en la selfie coincide con el registro (`result: true`)
- Alta confianza en la comparación (`confidence: 95.5`)
- Alta probabilidad de ser una persona real (`liveness: 99.8`)

### ❌ Autenticación fallida

```json
{
  "result": false,
  "confidence": 2.4115853309631348,
  "liveness": 99.89696502685547
}
```

**Interpretación:**
- La persona NO coincide con el registro (`result: false`)
- Baja confianza en la comparación (`confidence: 2.4`)
- Aunque la prueba de vida es válida, el rostro no corresponde

## Parámetro liveness (opcional)

Si se incluye el parámetro `liveness` con los datos del SDK:

- Se validarán **tanto la identidad como la prueba de vida**
- El resultado incluirá el campo `liveness` con la puntuación

Si se omite el parámetro `liveness`:

- **Solo** se validará la identidad (cotejo facial)
- El campo `liveness` puede ser `null` o no estar presente

## Umbrales recomendados

### Confidence (Confianza de cotejo facial)

| Rango | Interpretación | Recomendación |
|-------|----------------|---------------|
| **90-100** | Coincidencia muy alta | ✅ Aprobar automáticamente |
| **70-89** | Coincidencia media-alta | ⚠️ Revisar o solicitar segunda validación |
| **50-69** | Coincidencia baja | ❌ Rechazar o escalar a revisión manual |
| **0-49** | Sin coincidencia | ❌ Rechazar |

### Liveness (Prueba de vida)

| Rango | Interpretación | Recomendación |
|-------|----------------|---------------|
| **95-100** | Alta probabilidad de persona real | ✅ Aprobar |
| **85-94** | Probabilidad media-alta | ⚠️ Revisar |
| **0-84** | Baja probabilidad o posible fraude | ❌ Rechazar |

> **Nota:** Los umbrales exactos pueden ajustarse según las necesidades de seguridad de tu aplicación.

## Proceso asíncrono

Aunque este endpoint retorna una respuesta inmediata, también puedes consultar el resultado posteriormente usando el `executionId`:

**Ver:** [Resultados de Re-verificación →](re-verification-results.md)

## Consideraciones sobre la imagen

### Requisitos de la selfie

- **Formato:** JPG, JPEG, PNG
- **Resolución:** Mínimo 480x480 píxeles (recomendado: 720x720 o superior)
- **Peso máximo:** 15 MB
- **Iluminación:** Buena iluminación frontal
- **Rostro:** Debe estar completamente visible, sin obstrucciones (gafas de sol, mascarilla, etc.)

### Causas comunes de falla

- ❌ Imagen borrosa o de baja calidad
- ❌ Mala iluminación (muy oscura o con sombras fuertes)
- ❌ Rostro parcialmente oculto
- ❌ Múltiples rostros en la imagen
- ❌ Ningún rostro detectado
- ❌ Uso de foto de una foto (falla en liveness)

## Casos de uso

### 1. Autenticación en transacciones

Validar la identidad del usuario antes de aprobar una transacción financiera sensible.

### 2. Login biométrico

Permitir el acceso a la aplicación mediante reconocimiento facial en lugar de contraseña.

### 3. Confirmación de identidad

Verificar que la persona que realiza una operación es el titular de la cuenta.

### 4. Re-validación periódica

Solicitar autenticación biométrica periódicamente para mantener la sesión activa.

## Manejo de errores

### **401 - Unauthorized**

```json
{
  "msg": "Missing Authorization Header"
}
```

**Causa:** Token faltante o inválido.

### Errores comunes

| Error | Causa | Solución |
|-------|-------|----------|
| No rostro detectado | La imagen no contiene un rostro visible | Solicitar nueva selfie con mejor calidad |
| Múltiples rostros | Hay más de una persona en la imagen | Tomar selfie individual |
| Baja calidad | Imagen borrosa o pixelada | Mejorar calidad de captura |
| user_id no encontrado | No existe onboarding previo para ese user_id | Verificar que el usuario completó el onboarding |

## Ventajas de la re-verificación

- ⚡ **Rápida:** No requiere enviar documentos
- 🔒 **Segura:** Combina cotejo facial y prueba de vida
- 👍 **Mejor UX:** Experiencia más fluida para el usuario
- 💰 **Económica:** Menor costo que una verificación completa

## Siguientes pasos

- [Consultar resultados de re-verificación →](re-verification-results.md)
- [Ver verificación inicial (onboarding) →](verification-add.md)
- [Consultar resultados de verificación →](verification-results.md)

