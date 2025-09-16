# Tipos de Documentos Aceptados

## Descripción General

Este documento detalla todos los tipos de documentos que acepta el sistema de verificación, organizados por categorías y con sus códigos específicos.

## 📊 Resumen por Categorías

| Categoría | Cantidad | Códigos |
|-----------|----------|---------|
| **Documentos de Identidad Nacional** | 10 | `TYPE_ID`, `TYPE_ALIEN_ID`, `TYPE_RESIDENT_ID`, etc. |
| **Licencias de Conducir** | 4 | `TYPE_DRIVER_CARD`, `TYPE_DRIVING_PRIVILEGE_CARD`, etc. |
| **Pasaportes** | 5 | `TYPE_PASSPORT_CARD`, `TYPE_CONSULAR_PASSPORT`, etc. |
| **Total** | **19** | **19 tipos diferentes** |

## 🆔 Documentos de Identidad Nacional (national-id)

### Documentos Generales
| Código | Descripción | País | Uso Principal |
|--------|-------------|------|---------------|
| `TYPE_ID` | Documento de Identidad General | Internacional | Identificación oficial estándar |
| `TYPE_ALIEN_ID` | Documento de Identidad de Extranjero | Internacional | Identificación para extranjeros |
| `TYPE_RESIDENT_ID` | Documento de Identidad de Residente | Internacional | Identificación para residentes |

### Permisos de Residencia
| Código | Descripción | País | Uso Principal |
|--------|-------------|------|---------------|
| `TYPE_RESIDENCE_PERMIT` | Permiso de Residencia | Internacional | Permiso oficial de residencia |
| `TYPE_TEMPORARY_RESIDENCE_PERMIT` | Permiso de Residencia Temporal | Internacional | Permiso temporal de residencia |

### Certificados y Documentos Especiales
| Código | Descripción | País | Uso Principal |
|--------|-------------|------|---------------|
| `TYPE_CITIZENSHIP_CERTIFICATE` | Certificado de Ciudadanía | Internacional | Certificado oficial de ciudadanía |
| `TYPE_MULTIPURPOSE_ID` | Documento de Identidad Multiuso | Internacional | Documento con múltiples propósitos |
| `TYPE_VOTER_ID` | Documento de Identidad Electoral | Internacional | Identificación para votación |
| `TYPE_PROOF_OF_AGE_CARD` | Tarjeta de Prueba de Edad | Internacional | Documento que prueba la edad |
| `TYPE_GREEN_CARD` | Tarjeta Verde (Green Card) | Estados Unidos | Permiso de residencia permanente en EE.UU. |

## 🚗 Licencias de Conducir (driving-license)

| Código | Descripción | País | Uso Principal |
|--------|-------------|------|---------------|
| `TYPE_DRIVER_CARD` | Tarjeta de Conductor | Internacional | Licencia de conducir estándar |
| `TYPE_DRIVING_PRIVILEGE_CARD` | Tarjeta de Privilegio de Conducir | Estados Unidos | Permiso especial para conducir |
| `TYPE_DL` | Licencia de Conducir (Driver's License) | Internacional | Licencia de conducir tradicional |
| `TYPE_DL_PUBLIC_SERVICES_CARD` | Tarjeta de Servicios Públicos con DL | Internacional | Licencia con servicios públicos integrados |

## 🛂 Pasaportes (passport)

| Código | Descripción | País | Uso Principal |
|--------|-------------|------|---------------|
| `TYPE_PASSPORT_CARD` | Tarjeta de Pasaporte | Estados Unidos | Pasaporte en formato de tarjeta |
| `TYPE_CONSULAR_PASSPORT` | Pasaporte Consular | Internacional | Pasaporte emitido por consulado |
| `TYPE_MINORS_PASSPORT` | Pasaporte de Menores | Internacional | Pasaporte específico para menores de edad |
| `TYPE_ALIEN_PASSPORT` | Pasaporte de Extranjero | Internacional | Pasaporte para extranjeros |
| `TYPE_PASSPORT` | Pasaporte Estándar | Internacional | Pasaporte tradicional estándar |

## 🇨🇴 Documentos Colombianos Específicos

### Cédula de Ciudadanía (CC)
- **Código**: `TYPE_ID` (cuando país = Colombia)
- **Versiones soportadas**:
  - CC versión 2000
  - CC versión 2010
  - CC versión 2020
- **Elementos de seguridad**:
  - Hologramas
  - Microtexto
  - Marcas de agua
  - Código de barras

### Cédula de Extranjería (CE)
- **Código**: `TYPE_ALIEN_ID` (cuando país = Colombia)
- **Versiones soportadas**:
  - CE versión 2010
  - CE versión 2020
- **Elementos de seguridad**:
  - Hologramas
  - Microtexto
  - Marcas de agua

### Tarjeta de Identidad (TI)
- **Código**: `TYPE_ID` (cuando país = Colombia y edad < 18)
- **Versiones soportadas**:
  - TI versión 2010
  - TI versión 2020
- **Elementos de seguridad**:
  - Hologramas
  - Microtexto
  - Marcas de agua

## 🔍 Validación de Tipos de Documento

### Criterios de Validación
1. **Tipo de documento** debe estar en la lista de tipos aceptados
2. **País** debe coincidir con el tipo de documento
3. **Formato** debe ser reconocido por el sistema
4. **Elementos de seguridad** deben estar presentes

### Errores Comunes
- **`tipo incorrecto`** - Tipo de documento incorrecto
- **`tipo de documento incorrecto`** - Tipo de documento incorrecto
- **`Tipo de documento no está en la lista de tipos aceptados del contrato`** - Tipo no aceptado por el contrato
- **`Tipo de documento no especificado`** - No se especificó el tipo de documento


## Próximos Pasos

- [Razones de Alteración y Templates](alteration-reasons.md)
- [Reglas de Escalamiento](escalation-rules.md)
- [Gestión de Casos](case-management.md)
