# Gestión de Identidades

## Descripción General

La gestión de identidades en Backend Humano permite administrar de manera centralizada todas las identidades verificadas a través de nuestros servicios de verificación.

## Tipos de Identidades

### 🔐 Identidades Verificadas
- **Estado**: Verificadas y aprobadas
- **Acceso**: Completo a todos los servicios
- **Renovación**: Según políticas de la empresa

### ⏳ Identidades Pendientes
- **Estado**: En proceso de verificación
- **Acceso**: Limitado hasta completar verificación
- **Tiempo**: Procesamiento automático o manual

### ❌ Identidades Rechazadas
- **Estado**: Verificación fallida
- **Acceso**: Sin acceso a servicios
- **Acción**: Requiere nueva verificación

## Operaciones Disponibles

### Ver Identidades

#### Lista Principal
```
GET /api/identities
```

**Parámetros de Filtrado:**
- `status`: Estado de la identidad (verified, pending, rejected)
- `date_from`: Fecha de inicio
- `date_to`: Fecha de fin
- `search`: Búsqueda por nombre o documento

#### Detalles de Identidad
```
GET /api/identities/{id}
```

**Información Incluida:**
- Datos personales
- Historial de verificaciones
- Documentos asociados
- Log de actividades

### Actualizar Identidad

#### Modificar Información
```
PUT /api/identities/{id}
```

**Campos Modificables:**
- Información de contacto
- Estado de la identidad
- Notas administrativas

#### Cambiar Estado
```
PATCH /api/identities/{id}/status
```

**Estados Disponibles:**
- `active`: Identidad activa
- `suspended`: Identidad suspendida
- `archived`: Identidad archivada

### Eliminar Identidad

#### Eliminación Lógica
```
DELETE /api/identities/{id}
```

**Consideraciones:**
- Se mantiene el historial para auditoría
- No se puede recuperar fácilmente
- Requiere permisos especiales

## Integración con ANI Compliance

### Sincronización Automática

Las identidades se sincronizan automáticamente con ANI Compliance:

1. **Verificación Completada**: Se crea automáticamente la identidad
2. **Actualización de Estado**: Cambios se reflejan en tiempo real
3. **Eliminación**: Se notifica a ANI Compliance

### Webhooks

Backend Humano recibe notificaciones de ANI Compliance:

```json
{
  "event": "verification.completed",
  "identity_id": "12345",
  "user_data": {
    "name": "Juan Pérez",
    "document": "12345678",
    "email": "juan@example.com"
  },
  "verification_result": "approved"
}
```

## Seguridad y Auditoría

### Log de Actividades

Todas las operaciones se registran:

- **Usuario**: Quién realizó la acción
- **Acción**: Qué se hizo
- **Timestamp**: Cuándo ocurrió
- **IP**: Desde dónde se realizó
- **Resultado**: Éxito o fallo

### Permisos

#### Roles Disponibles
- **Admin**: Acceso completo
- **Manager**: Gestión de identidades
- **Viewer**: Solo lectura
- **Auditor**: Acceso a logs y reportes

## Próximos Pasos

- [Integración con APIs](api-integration.md)
- [Troubleshooting](troubleshooting.md)
