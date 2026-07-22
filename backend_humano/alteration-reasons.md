# Razones de pending y asignación BH

Inventario de razones por las que una identidad queda en `pending` y puede asignarse a un revisor BH.

## Flujo relevante

Al completar el flow, `ensure_completion` ejecuta `rules_validation`. Si las reglas fallan y está **fuera de horario desatendido**, la identidad permanece `pending` y se asigna con `assign_verification_equitably`.

---

## 🚨 Razones de Alteración (`alteration_reasons`)

### 🇨🇴 Registraduría (ANI)

| Razón | Origen |
|---|---|
| `ani_status` | ANI: estado inválido |
| `ani_names_missing` | ANI: nombres faltantes |
| `ani_names` | ANI: nombres no coinciden |

### 🛂 Migración

| Razón | Origen |
|---|---|
| `migration_status` | Migración: estado inválido |
| `migration_expiration_date` | Migración: fecha de expiración |
| `migration_names_missing` | Migración: nombres faltantes |
| `migration_names` | Migración: nombres no coinciden |

### 🇪🇨 Ecuador (registro civil)

| Razón | Origen |
|---|---|
| `ec_registry_status` | Ecuador: estado de registro |
| `ec_registry_death` | Ecuador: fallecido |
| `ec_names_missing` | Ecuador: nombres faltantes |
| `ec_names` | Ecuador: nombres no coinciden |

### 🚫 Lista negra

| Razón | Origen |
|---|---|
| `black_list` | Lista negra |
| `black_list2` | Lista negra 2 |

### 📄 Validación de documento y checks

| Razón | Origen |
|---|---|
| `document_validation_number` | Validación de número de documento |
| `Screen Check` | Fallo screen check |
| `Photocopy Check` | Fallo photocopy check |
| `Model AI` | Modelo IA: rechazo automático (score 0–10) |
| `Data Match` | `dataMatchResult` no es SUCCESS/NOT_PERFORMED |

### 🔧 Overrides y revisión BH

| Razón | Origen |
|---|---|
| `alterado por template` | Override: template falló → fuerza `alteration=False` (companies 1, 163, 186, 164) |
| `Manual_revision` | Revisión manual del BH (al completar) |

---

## 📄 Razones de Template (`template_reasons`)

### Problemas de OCR / procesamiento

| Razón | Origen |
|---|---|
| `Error en procesamiento OCR - falla en análisis del documento` | Error OCR |
| `datos OCR vacíos` | OCR sin datos |
| `falla identificación de campos` | `FIELD_IDENTIFICATION_FAILED` |
| `error en la solicitud de procesamiento` | `RequestException` |
| `imagen del lado incorrecto` | `SCANNING_WRONG_SIDE` |

### País

| Razón | Origen |
|---|---|
| `país no coincide` | País OCR ≠ declarado (CO) |
| `país no es Colombia` | Cédula con país ≠ CO |
| `País del documento no coincide con el declarado por el usuario` | Caso general |
| `País del documento no está en la lista de países aceptados del contrato` | País no permitido |

### Tipo de documento

| Razón | Origen |
|---|---|
| `tipo incorrecto` | No es `TYPE_ALIEN_ID` |
| `tipo de documento incorrecto` | No es `TYPE_ID` |
| `Tipo de documento no especificado` | Sin document type |
| `Tipo de documento no reconocido` | Tipo no reconocido |
| `Tipo de documento no está en la lista de tipos aceptados del contrato` | Tipo no permitido |

### Template / formato

| Razón | Origen |
|---|---|
| `template no reconocido` | Template check fallido (CO) |
| `template back no coincide con el front` | Front/back prediction distinta |
| `documento no reconocido(permiso de protección temporal)` | PPT no detectado |
| `Template del documento no válido - formato no reconocido` | Template inválido |

### Data match, expiración y revisión BH

| Razón | Origen |
|---|---|
| `Data match fallido - información del documento inconsistente` | Data match fallido (template flow) |
| `documento expirado` | Documento vencido |
| `Manual_revision` | Revisión manual del BH |

---

## ⚠️ Casos pending sin razón escrita

Fallan `rules_validation` y dejan `pending` + asignación, pero **no** crean fila en `AlterationReason` / `TemplateReason`:

1. **Zona manual del modelo IA** (score **10–82**) — pending sin razón `Model AI`
   - **0–10**: rechazo automático
   - **10–82**: BH
   - **82–100**: aprobación automática
2. **Sin resultado del modelo AI** en Colombia con `can_model_alteration`
3. **Template prediction `"new"`** (cédula nueva colombiana) — fuerza pending siempre
4. **Face match** fuera del rango auto-complete (score > 30 y falla threshold / score inválido)
5. **Liveness** faltante cuando el contrato exige video
6. **`processingStatus`** en `FIELD_IDENTIFICATION_FAILED` / `LOW_CONFIDENCE` / `MANDATORY_FIELD_MISSING` (a veces sin `TemplateReason`)
7. **One-to-N fallido** (company 5)
8. **Contrato `reviewed` sin `requires_partial_validation`** (nunca auto-completa)
9. **OCR incompleto** (sin `full_name` / `document_number` / `birth`)

---

## ✅ Razones que auto-completan (no quedan pending asignadas)

Aunque se guardan, disparan auto-complete en `rules_validation` y **no** se asignan a BH:

### Alteration

- `black_list`
- `black_list2`

### Template

- `documento expirado`
- `Error en procesamiento OCR - falla en análisis del documento`

### Combos template

- `País del documento no coincide con el declarado por el usuario` + `País del documento no está en la lista de países aceptados del contrato`
- `País del documento no está en la lista de países aceptados del contrato` + `Tipo de documento no especificado`

### Otros

- Face match score **0–30** (sin template `"new"` de cédula nueva colombiana)
- Modelo IA score **0–10**: rechazo automático (excepto company 5)
- Modelo IA score **82–100**: aprobación automática

---

## 📊 Resumen

| Categoría | Cantidad |
|---|---|
| AlterationReason | 20 |
| TemplateReason | 21 |
| Pending sin razón escrita | 9 |
| **Total códigos / casos documentados** | **50** |

---

## Referencias de código

- `app/v2/verification/verification_utils/utils.py` — `ensure_completion`, `rules_validation`, `has_auto_complete_*`
- `app/webhook/routes.py` — razones de alteration (Screen/Photocopy/Model AI/Data Match, black_list, document_validation_number)
- `app/webhook/utils2.py` — TemplateReason y ANI / black_list2
- `app/v2/gov_checks/MIGRATION/utils.py` — migration_*
- `app/v2/gov_checks/EC/utils.py` — ec_*
