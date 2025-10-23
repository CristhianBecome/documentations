# Re-verificación (Autenticación Biométrica)

Una vez que un usuario ha completado el proceso de **Onboarding** (verificación inicial de identidad), puede autenticarse de forma rápida y segura a través de un **cotejo facial** y una **prueba de vida (liveness detection)**. Este procedimiento elimina la necesidad de presentar nuevamente el documento de identidad en cada transacción.

## Proceso de re-verificación

1. **Obtener selfie:** Captura una nueva selfie del usuario
2. **Enviar validación:** Utiliza el endpoint `/matches` con la imagen y datos del usuario
3. **Procesar resultado:** Interpreta la respuesta según tus umbrales de confianza

## Flujo de re-verificación

```
Cliente → [Captura selfie] → [POST /matches con imagen] → API
API → [Compara con selfie registrada] → [Valida liveness] → Retorna resultado
Cliente → [Procesa resultado] → [Aplica lógica de negocio]
```

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

### Parámetros del Request

La petición debe enviarse como `multipart/form-data` e incluir campos de texto y archivos.

#### ✅ Campos OBLIGATORIOS

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `user_id` | string | Identificador del usuario registrado en el onboarding |
| `contract_id` | string | ID del contrato a utilizar |
| `image` | file | Foto selfie actual del usuario |

#### 📱 Campos OPCIONALES

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `liveness` | object | Datos de prueba de vida provistos por el SDK |

### Ejemplos de request

**Re-verificación básica (solo cotejo facial):**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/matches' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--form 'user_id="usuario_12345_1699123456"' \
--form 'contract_id="1"' \
--form 'image=@"selfie_actual.jpg"'
```

**Re-verificación con liveness (cotejo facial + prueba de vida):**

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/matches' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--form 'user_id="usuario_12345_1699123456"' \
--form 'contract_id="1"' \
--form 'image=@"selfie_actual.jpg"' \
--form 'liveness="{\"score\": 0.95, \"passed\": true}"'
```

### Respuestas de la API

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

## Consideraciones de seguridad

- 🔒 **Almacena de forma segura** las selfies y datos biométricos
- ⏰ **Los executionId expiran** después de un tiempo determinado
- 🔄 **Implementa validación** de que el `user_id` pertenece a tu contrato
- 🔑 **Nunca expongas** las selfies en logs o URLs públicas
- 📝 **Registra todas las re-verificaciones** para auditoría de seguridad
- 🛡️ **Valida umbrales de confianza** según políticas de riesgo

## Uso de la imagen en las solicitudes

La imagen debe enviarse como archivo en el campo `image` de la petición multipart/form-data:

```bash
--form 'image=@"selfie_actual.jpg"'
```

Todas las solicitudes al endpoint `/matches` deben incluir este archivo obligatorio.

## Proceso asíncrono

Aunque este endpoint retorna una respuesta inmediata, también puedes consultar el resultado posteriormente usando el `executionId`:

**Ver:** [Resultados de Re-verificación →](re-verification-results.md)

### Requisitos de calidad

#### Imágenes (image)
- **Formatos:** `.jpg`, `.jpeg`, `.png`
- **Resolución:** Mínimo 480x480 píxeles (recomendado: 720x720 o superior)
- **Peso máximo:** 15 MB
- **Rostro centrado, buena iluminación, solo una persona**
- **Sin obstrucciones:** Sin gafas de sol, mascarilla, etc.

#### user_id recomendado
- **Sin caracteres especiales:** Solo letras, números, guiones (`-`) y guiones bajos (`_`)
- **Longitud máxima:** 50 caracteres
- **Ejemplos válidos:** `user-12345-abc`, `550e8400-e29b-41d4`, `usuario_789_1699123456`
- **Ejemplos inválidos:** `user@123#invalid!`, `juan.perez@gmail.com`

## Casos de uso

### 1. Autenticación en transacciones

Validar la identidad del usuario antes de aprobar una transacción financiera sensible.

### 2. Login biométrico

Permitir el acceso a la aplicación mediante reconocimiento facial en lugar de contraseña.

### 3. Confirmación de identidad

Verificar que la persona que realiza una operación es el titular de la cuenta.

### 4. Re-validación periódica

Solicitar autenticación biométrica periódicamente para mantener la sesión activa.

### Errores comunes

**400 - Campos inválidos:**
- `"Falta el 'user_id'"` - Campo obligatorio faltante
- `"Falta el 'contract_id'"` - Campo obligatorio faltante
- `"Falta el archivo 'image'"` - Imagen selfie requerida
- `"user_id no encontrado"` - No existe onboarding previo para ese user_id

**400 - Archivos:**
- `"No se detectó rostro en la imagen"` - La imagen no contiene un rostro visible
- `"Múltiples rostros detectados"` - Hay más de una persona en la imagen
- `"Imagen de baja calidad"` - Imagen borrosa o pixelada
- `"Formato de imagen no válido"` - Usar JPG, JPEG o PNG

**401 - Autenticación:**
- `"Missing Authorization Header"` - Falta token JWT
- `"Token has expired"` - Token expirado, renovar

**Errores de contrato:**
- `"No se encontró el contrato"` - contract_id inválido
- `"Usuario no registrado"` - user_id no existe en el sistema

## Mejores prácticas

**Calidad de archivos:**
- Mejor calidad de imagen = Procesamiento más rápido
- Comprimir antes de enviar (máximo 15MB por archivo)
- Verificar iluminación y nitidez antes de subir

**Seguridad:**
- Renovar tokens JWT regularmente
- No exponer tokens en logs o URLs
- Transmitir únicamente por HTTPS
- Validar umbrales de confianza según políticas de seguridad

**Performance:**
- Reutilizar el mismo token JWT para múltiples requests
- Implementar timeouts adecuados (60-90 segundos)
- Validar datos antes de enviar para evitar errores
- Configurar umbrales de confianza apropiados

**Manejo de errores:**
- Implementar reintentos automáticos para errores 500
- Registrar respuestas de error para debugging
- Manejar casos de baja confianza con flujos alternativos
- Implementar notificaciones de fallo para el usuario

## Ventajas de la re-verificación

- ⚡ **Rápida:** No requiere enviar documentos
- 🔒 **Segura:** Combina cotejo facial y prueba de vida
- 👍 **Mejor UX:** Experiencia más fluida para el usuario
- 💰 **Económica:** Menor costo que una verificación completa

## Siguientes pasos

- [Consultar resultados de re-verificación →](re-verification-results.md)
- [Ver verificación inicial (onboarding) →](verification-add.md)
- [Consultar resultados de verificación →](verification-results.md)

