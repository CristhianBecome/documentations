# Razones de Alteración y Templates

## Descripción General

Este documento detalla las razones específicas de alteración que pueden causar que un caso sea enviado al Backend Humano, así como los templates de documentos reconocidos por el sistema.

## 🚨 Razones de Alteración (AlterationReason)

### 🔴 Razones de Alteración - Flag en Rojo

#### 🇨🇴 Registraduría (ANI = Registraduría)
1. **`ani_status`** - Problema con el estado en la Registraduría
2. **`ani_names_missing`** - Nombres faltantes en la Registraduría
3. **`ani_names`** - Problema con los nombres en la Registraduría

#### 🚫 Lista Negra
4. **`black_list`** - Usuario está en lista negra

#### 📄 Validación de Documento
5. **`document_validation_number`** - Validación del número de documento falló

#### 🔍 Revisión Manual
6. **`Screen Check`** - Sospecha de pantalla
7. **`Photocopy Check`** - Sospecha de fotocopia

#### 🤖 Modelo AI
8. **`Model AI`** - Modelo de IA sospecha alteración en un 90%

#### 📊 Data Match
9. **`Data Match`** - No coincide información de frente con reverso (puede deberse a falla en OCR)

## 📄 Razones de Template (Rev. tipo documento)

### 🔴 Razones de Template - Flag en Rojo

#### 📋 Problemas de OCR (2)
1. **`datos OCR vacíos`** - Los datos del OCR están vacíos
2. **`falla identificación de campos`** - No se pudieron identificar los campos del documento

#### 🌍 Problemas de País (4)
3. **`país no coincide`** - El país del documento no coincide
4. **`país no es Colombia`** - El país no es Colombia
5. **`País del documento no coincide con el declarado por el usuario`** - País del documento no coincide con el declarado
6. **`País del documento no está en la lista de países aceptados del contrato`** - País no aceptado por el contrato

#### 📄 Problemas de Tipo de Documento (3)
7. **`tipo incorrecto`** - Tipo de documento incorrecto
8. **`tipo de documento incorrecto`** - Tipo de documento incorrecto
9. **`Tipo de documento no está en la lista de tipos aceptados del contrato`** - Tipo de documento no aceptado por el contrato

#### 🔧 Problemas de Template/Formato (4)
10. **`template no reconocido`** - El template del documento no es reconocido
11. **`documento no reconocido(permiso de protección temporal)`** - Documento no reconocido por permiso temporal
12. **`Template del documento no válido - formato no reconocido`** - Template no válido, formato no reconocido
13. **`Error en procesamiento OCR - falla en análisis del documento`** - Error en procesamiento OCR

#### 📊 Problemas de Data Match (1)
14. **`Data match fallido - información del documento inconsistente`** - Data match fallido, información inconsistente

#### ⚠️ Problemas de Validación (2)
15. **`Tipo de documento no especificado`** - No se especificó el tipo de documento
16. **`Tipo de documento no reconocido`** - El tipo de documento no es reconocido por el sistema


## 📊 Resumen por Categorías

### 🔴 Razones de Alteración (9)
- **Registraduría (3 tipos)** - ANI = Registraduría
- **Blacklist (1 tipo)**
- **Validación de documento (1 tipo)**
- **Revisión manual (2 tipos)**
- **Model AI (1 tipo)**
- **Data Match (1 tipo)**

### 📄 Razones de Template (16)
- **Problemas de OCR (2)**
- **Problemas de país (4)**
- **Problemas de tipo de documento (3)**
- **Problemas de template/formato (4)**
- **Problemas de data match (1)**
- **Problemas de validación (2)**

### 📊 **Total: 25 razones diferentes**

## 🔍 Códigos de Error

### Códigos de Alteración (AlterationReason)

| Código | Descripción | Prioridad |
|--------|-------------|-----------|
| `ani_status` | Problema con el estado en la Registraduría | Alta |
| `ani_names_missing` | Nombres faltantes en la Registraduría | Alta |
| `ani_names` | Problema con los nombres en la Registraduría | Alta |
| `black_list` | Usuario está en lista negra | Alta |
| `document_validation_number` | Validación del número de documento falló | Media |
| `Screen Check` | Sospecha de pantalla | Media |
| `Photocopy Check` | Sospecha de fotocopia | Media |
| `Model AI` | Modelo de IA sospecha alteración en un 90% | Alta |
| `Data Match` | No coincide información de frente con reverso | Media |

### Códigos de Template (TemplateReason)

| Código | Descripción | Prioridad |
|--------|-------------|-----------|
| `datos OCR vacíos` | Los datos del OCR están vacíos | Baja |
| `falla identificación de campos` | No se pudieron identificar los campos | Baja |
| `país no coincide` | El país del documento no coincide | Media |
| `país no es Colombia` | El país no es Colombia | Media |
| `País del documento no coincide con el declarado por el usuario` | País no coincide con el declarado | Media |
| `País del documento no está en la lista de países aceptados del contrato` | País no aceptado por el contrato | Media |
| `tipo incorrecto` | Tipo de documento incorrecto | Media |
| `tipo de documento incorrecto` | Tipo de documento incorrecto | Media |
| `Tipo de documento no está en la lista de tipos aceptados del contrato` | Tipo no aceptado por el contrato | Media |
| `template no reconocido` | El template del documento no es reconocido | Alta |
| `documento no reconocido(permiso de protección temporal)` | Documento no reconocido por permiso temporal | Media |
| `Template del documento no válido - formato no reconocido` | Template no válido, formato no reconocido | Alta |
| `Data match fallido - información del documento inconsistente` | Data match fallido, información inconsistente | Media |
| `Tipo de documento no especificado` | No se especificó el tipo de documento | Baja |
| `Tipo de documento no reconocido` | El tipo de documento no es reconocido por el sistema | Media |
| `Error en procesamiento OCR - falla en análisis del documento` | Error en procesamiento OCR | Baja |


