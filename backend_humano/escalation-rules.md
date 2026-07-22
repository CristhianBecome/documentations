# Cuándo Revisar un Caso

## ¿Qué es esto?

Esta guía te explica **cuándo** un caso llega a tu mesa para que lo revises. La máquina decide automáticamente cuándo necesita tu ayuda.

## Flujo técnico (resumen)

Al completar el flow, `ensure_completion` ejecuta `rules_validation`. Si las reglas fallan y está **fuera de horario desatendido**, la identidad permanece `pending` y se asigna con `assign_verification_equitably`.

El inventario completo de códigos está en [Razones de pending y asignación BH](alteration-reasons.md).

---

## 🚨 Cuándo llega un caso a tu mesa

### 🇨🇴 Regla 1: Documentos Colombianos - Zona manual del modelo IA
**¿Cuándo?** Score del modelo IA entre **10 y 82** (zona BH).

| Score | Resultado |
|---|---|
| **0 – 10** | Rechazo automático (no llega a BH) |
| **10 – 82** | `pending` → asignación BH |
| **82 – 100** | Aprobación automática (no llega a BH) |

- **Ejemplo**: Score 55 → va a BH
- **Importante**: En zona 10–82 queda `pending` **sin** escribir razón `Model AI` en `AlterationReason`
- **Solo para**: Cédulas colombianas (CC, CE, TI) — versiones **new** y **old**

### 🇨🇴 Regla 2: Documentos Colombianos - La máquina no pudo revisar
**¿Cuándo?** Colombia con `can_model_alteration` y **sin resultado** del modelo AI.
- **Ejemplo**: Error en el sistema o máquina no disponible
- **Importante**: Queda `pending` **sin** razón escrita
- **Solo para**: Cédulas colombianas (CC, CE, TI) — versiones **new** y **old**

### 🤖 Regla 3: Modelo AI — rechazo automático (score 0–10)
**¿Cuándo?** Score del modelo IA en **0–10**.
- **Efecto**: Rechazo automático (puede guardar razón `Model AI`); **no** queda pending asignada a BH
- **Excepción**: Company 5 (zona TTA de rechazo automático no aplica igual)
- **Para**: Documentos donde aplica el modelo

### 📄 Regla 4: El documento / template no se reconoce
**¿Cuándo?** Fallan validaciones de template, país, tipo u OCR. Incluye:
- Datos OCR vacíos, falla de campos, error de procesamiento
- País o tipo de documento incorrecto / no permitido
- Template no reconocido o inválido
- Front/back con predicción distinta (`template back no coincide con el front`)
- Imagen del lado incorrecto (`SCANNING_WRONG_SIDE`)
- **Template prediction `"new"`** (cédula nueva colombiana) — fuerza `pending` **siempre** (sin razón escrita necesariamente)
- **`processingStatus`** en `FIELD_IDENTIFICATION_FAILED` / `LOW_CONFIDENCE` / `MANDATORY_FIELD_MISSING` (a veces sin `TemplateReason`)
- **Para**: Todos los tipos de documento

### 👤 Regla 5: La foto no coincide (face match)
**¿Cuándo?** Face match fuera del rango auto-complete: score **> 30** y falla threshold, o score inválido.
- **Ejemplo**: La persona en el documento no se parece a la selfie
- **Importante**: Queda `pending` **sin** razón en Alteration/Template
- **Nota**: Score **0–30** (sin template `"new"` de cédula nueva colombiana) **auto-completa** y no llega a BH
- **Para**: Todos los tipos de documento

### 📱 Regla 6: No pasó la prueba de vida (liveness)
**¿Cuándo?** Falta liveness cuando el contrato exige video.
- **Ejemplo**: No se pudo verificar que es una persona real
- **Importante**: Queda `pending` **sin** razón escrita
- **Para**: Contratos que exigen video

### 🏛️ Regla 7: Checks gubernamentales y otras razones escritas
**¿Cuándo?** Cualquiera de estas razones en `AlterationReason` / `TemplateReason` (salvo las que auto-completan):

**Alteration**
- Registraduría (ANI): `ani_status`, `ani_names_missing`, `ani_names`
- Migración: `migration_status`, `migration_expiration_date`, `migration_names_missing`, `migration_names`
- Ecuador: `ec_registry_status`, `ec_registry_death`, `ec_names_missing`, `ec_names`
- `document_validation_number`, `Screen Check`, `Photocopy Check`, `Data Match`
- `alterado por template` (companies 1, 163, 186, 164)
- `Manual_revision` (al completar revisión BH)

**Template** (ejemplos frecuentes)
- País / tipo / template inválidos
- Data match fallido en template flow
- `Manual_revision`

Ver tabla completa en [alteration-reasons.md](alteration-reasons.md).

### 🏢 Regla 8: One-to-N fallido (company 5)
**¿Cuándo?** El sistema detecta match one-to-N fallido.
- **Importante**: Queda `pending` **sin** razón escrita
- **Para**: Company 5

### 📋 Regla 9: OCR incompleto
**¿Cuándo?** No hay `full_name` / `document_number` / `birth` en OCR.
- **Importante**: Queda `pending` **sin** razón escrita
- **Para**: Todos los tipos de documento

### 📝 Regla 10: Contrato `reviewed` sin validación parcial
**¿Cuándo?** Contrato configurado como `reviewed` **sin** `requires_partial_validation`.
- **Efecto**: **Nunca** auto-completa; siempre puede quedar `pending` + asignación
- **Importante**: No escribe razón en Alteration/Template por este solo hecho

---

## ⚠️ Casos pending sin razón escrita (checklist)

Estos fallan `rules_validation` y se asignan a BH, pero **no** crean fila en `AlterationReason` / `TemplateReason`:

1. Zona manual del modelo IA (score **10–82**)
2. Sin resultado del modelo AI (CO + `can_model_alteration`)
3. Template prediction `"new"` (cédula nueva colombiana)
4. Face match fuera de auto-complete (score > 30 / inválido)
5. Liveness faltante (contrato exige video)
6. `processingStatus` crítico (a veces sin TemplateReason)
7. One-to-N fallido (company 5)
8. Contrato `reviewed` sin `requires_partial_validation`
9. OCR incompleto (sin nombre / documento / birth)

---

## ✅ Qué NO llega a BH (auto-complete)

Aunque a veces se guardan razones, disparan auto-complete en `rules_validation` y **no** quedan pending asignadas:

| Tipo | Caso |
|---|---|
| Alteration | `black_list`, `black_list2` |
| Template | `documento expirado` |
| Template | `Error en procesamiento OCR - falla en análisis del documento` |
| Combo template | País no coincide + país no aceptado por contrato |
| Combo template | País no aceptado + tipo no especificado |
| Face match | Score **0–30** (sin template `"new"` de cédula nueva colombiana) |
| Modelo IA | Score **0–10**: rechazo automático (excepto company 5) |
| Modelo IA | Score **82–100**: aprobación automática |

---

## 📊 Resumen de Prioridades

### 🔴 Alta Prioridad (Revisar primero)
- **Regla 1–2**: Modelo IA zona manual (10–82) o sin resultado
- **Regla 4**: Documento / template no reconocido (incl. `"new"` = cédula nueva colombiana)
- **Regla 7**: Checks gov (ANI / Migración / Ecuador) y flags de alteración
- **Regla 8**: One-to-N (company 5)

### 🟡 Media Prioridad (Revisar después)
- **Regla 5**: Face match
- **Regla 6**: Liveness
- **Regla 10**: Contrato `reviewed` (revisión completa por diseño)

### 🟢 Baja Prioridad (Revisar al final)
- **Regla 9**: OCR incompleto (problemas técnicos de lectura)

---

## Próximos Pasos

- [Tipos de documentos aceptados](document-types.md)
- [Razones de pending y asignación BH](alteration-reasons.md)
