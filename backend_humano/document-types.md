# Tipos de Documentos Aceptados

## ¿Qué es esto?

Esta guía te explica **qué tipos de documentos** acepta el sistema y **cómo los interpreta**. Es importante que sepas esto para entender por qué algunos documentos van a revisión.

## 📊 Resumen

| Categoría | Cantidad | Para qué sirve |
|-----------|----------|----------------|
| **Documentos de Identidad** | 11 | Identificación oficial |
| **Licencias de Conducir** | 4 | Permiso para conducir |
| **Pasaportes** | 5 | Documento de viaje |
| **Total** | **20** | **20 tipos diferentes** |

## 🆔 Documentos de Identidad Nacional

| Código | Descripción | País | Para qué sirve |
|--------|-------------|------|----------------|
| `TYPE_ID` | Documento de Identidad General | Internacional | Identificación oficial estándar |
| `TYPE_MINORS_ID` | Documento de Identidad de Menores | Internacional | Identificación para menores de edad |
| `TYPE_ALIEN_ID` | Documento de Identidad de Extranjero | Internacional | Identificación para extranjeros |
| `TYPE_RESIDENT_ID` | Documento de Identidad de Residente | Internacional | Identificación para residentes |
| `TYPE_RESIDENCE_PERMIT` | Permiso de Residencia | Internacional | Permiso oficial de residencia |
| `TYPE_TEMPORARY_RESIDENCE_PERMIT` | Permiso de Residencia Temporal | Internacional | Permiso temporal de residencia |
| `TYPE_CITIZENSHIP_CERTIFICATE` | Certificado de Ciudadanía | Internacional | Certificado oficial de ciudadanía |
| `TYPE_MULTIPURPOSE_ID` | Documento de Identidad Multiuso | Internacional | Documento con múltiples propósitos |
| `TYPE_VOTER_ID` | Documento de Identidad Electoral | Internacional | Identificación para votación |
| `TYPE_PROOF_OF_AGE_CARD` | Tarjeta de Prueba de Edad | Internacional | Documento que prueba la edad |
| `TYPE_GREEN_CARD` | Tarjeta Verde (Green Card) | Estados Unidos | Permiso de residencia permanente en EE.UU. |

## 🚗 Licencias de Conducir

| Código | Descripción | País | Para qué sirve |
|--------|-------------|------|----------------|
| `TYPE_DRIVER_CARD` | Tarjeta de Conductor | Internacional | Licencia de conducir estándar |
| `TYPE_DRIVING_PRIVILEGE_CARD` | Tarjeta de Privilegio de Conducir | Estados Unidos | Permiso especial para conducir |
| `TYPE_DL` | Licencia de Conducir (Driver's License) | Internacional | Licencia de conducir tradicional |
| `TYPE_DL_PUBLIC_SERVICES_CARD` | Tarjeta de Servicios Públicos con DL | Internacional | Licencia con servicios públicos integrados |

## 🛂 Pasaportes

| Código | Descripción | País | Para qué sirve |
|--------|-------------|------|----------------|
| `TYPE_PASSPORT_CARD` | Tarjeta de Pasaporte | Estados Unidos | Pasaporte en formato de tarjeta |
| `TYPE_CONSULAR_PASSPORT` | Pasaporte Consular | Internacional | Pasaporte emitido por consulado |
| `TYPE_MINORS_PASSPORT` | Pasaporte de Menores | Internacional | Pasaporte específico para menores de edad |
| `TYPE_ALIEN_PASSPORT` | Pasaporte de Extranjero | Internacional | Pasaporte para extranjeros |
| `TYPE_PASSPORT` | Pasaporte Estándar | Internacional | Pasaporte tradicional estándar |

## 🇨🇴 Documentos Colombianos Específicos

Aplica a las versiones **new** y **old** de cada tipo.

### Cédula de Ciudadanía (CC)
- **Código**: `TYPE_ID` (cuando país = Colombia)
- **Para**: Ciudadanos colombianos mayores de 18 años
- **Variantes**: documento new y old

### Cédula de Extranjería (CE)
- **Código**: `TYPE_ALIEN_ID` (cuando país = Colombia)
- **Para**: Extranjeros residentes en Colombia
- **Variantes**: documento new y old

### Tarjeta de Identidad (TI)
- **Código**: `TYPE_ID` (cuando país = Colombia y edad < 18) **o** `TYPE_MINORS_ID`
- **Para**: Ciudadanos colombianos menores de 18 años
- **Nota**: El sistema puede detectarla como `TYPE_ID` o como `TYPE_MINORS_ID`
- **Variantes**: documento new y old

## 🔍 ¿Por qué es importante saber esto?

### Para revisores de BH:
1. **Entender el contexto** - Saber qué tipo de documento estás revisando
2. **Aplicar reglas correctas** - Algunas reglas solo aplican para documentos colombianos
3. **Identificar problemas** - Saber si el tipo de documento es correcto
4. **Tomar decisiones** - Entender por qué el sistema envió el caso a revisión

### Errores comunes que verás:
- **`tipo incorrecto`** - El usuario seleccionó el tipo de documento equivocado
- **`tipo de documento incorrecto`** - El sistema no reconoce el tipo
- **`Tipo de documento no está en la lista de tipos aceptados del contrato`** - Tipo no permitido
- **`Tipo de documento no especificado`** - No se especificó el tipo

## Próximos Pasos

- [Cuándo revisar un caso](escalation-rules.md)
- [Razones de pending y asignación BH](alteration-reasons.md)