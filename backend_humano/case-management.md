# Gestión de Casos en Backend Humano

## Descripción General

La gestión de casos en Backend Humano permite a los revisores humanos procesar eficientemente las verificaciones que requieren revisión manual. Cada caso incluye toda la información necesaria para tomar una decisión informada.

## 📋 Tipos de Casos

### 🔍 Casos de Documento
**Descripción**: Documentos que requieren verificación manual
- **Documentos borrosos** o de baja calidad
- **Elementos de seguridad** cuestionables
- **Marcas de agua** inconsistentes
- **Tipografía** que no coincide con estándares

### 👤 Casos de Identidad
**Descripción**: Discrepancias en la identidad del usuario
- **Fotos que no coinciden** con el documento
- **Diferencias de edad** aparente
- **Cambios drásticos** en apariencia
- **Múltiples identidades** asociadas

### 📱 Casos de Liveness
**Descripción**: Pruebas de vida que fallaron
- **Videos estáticos** o pre-grabados
- **Movimientos no naturales**
- **Calidad insuficiente** del video
- **Detección de deepfakes**

### 🇨🇴 Casos Regulatorios
**Descripción**: Verificaciones requeridas por normativas
- **Usuarios colombianos** con documentos CC
- **Cumplimiento** con regulaciones locales
- **Validación adicional** requerida
- **Verificación con Registraduría**

## 🎯 Estados de Casos

### 📥 Pendiente
- **Descripción**: Caso asignado, esperando revisión
- **Acciones**: Revisar, asignar, transferir
- **Tiempo límite**: 24 horas

### 🔍 En Revisión
- **Descripción**: Caso siendo revisado por un revisor
- **Acciones**: Aprobar, rechazar, solicitar info
- **Tiempo límite**: 4 horas

### ⏳ Esperando Información
- **Descripción**: Se solicitó información adicional al usuario
- **Acciones**: Revisar respuesta, continuar proceso
- **Tiempo límite**: 72 horas

### ✅ Aprobado
- **Descripción**: Caso aprobado por revisor humano
- **Acciones**: Ver historial, generar reporte
- **Estado final**: Completado

### ❌ Rechazado
- **Descripción**: Caso rechazado por revisor humano
- **Acciones**: Ver historial, generar reporte
- **Estado final**: Completado

## 🔄 Flujo de Trabajo

### 1. Recepción del Caso
```
Caso Escalado → Notificación → Asignación → Revisión
```

### 2. Proceso de Revisión
```
Revisar Datos → Analizar Evidencia → Tomar Decisión → Notificar Usuario
```

### 3. Seguimiento
```
Decisión Tomada → Actualizar Estado → Generar Reporte → Archivar
```

## 🛠️ Herramientas de Revisión

### Panel de Revisión
- **Vista de documentos**: Zoom, rotación, filtros
- **Comparación facial**: Lado a lado con métricas
- **Análisis de liveness**: Reproducción de video con análisis
- **Historial del usuario**: Verificaciones anteriores

### Herramientas de Análisis
- **Detector de deepfakes**: Análisis automático de videos
- **Verificador de documentos**: Validación de elementos de seguridad
- **Comparador facial**: Algoritmos de similitud
- **Análisis de calidad**: Métricas de imagen y video

## 📊 Métricas de Rendimiento

### KPIs del Revisor
- **Casos procesados por día**: Productividad individual
- **Tiempo promedio de revisión**: Eficiencia
- **Tasa de precisión**: Decisiones correctas
- **Satisfacción del usuario**: Rating post-decisión

### KPIs del Sistema
- **Tiempo de resolución**: Promedio de casos en BH
- **Tasa de aprobación**: % de casos aprobados
- **Tasa de rechazo**: % de casos rechazados
- **Casos escalados**: Volumen diario/semanal

## 🚨 Alertas y Notificaciones

### Alertas Críticas
- **Casos de alta prioridad**: Documentos falsos detectados
- **Múltiples intentos**: Usuarios con patrón sospechoso
- **Alertas de seguridad**: Detección de fraude
- **Casos vencidos**: Tiempo límite excedido

### Notificaciones Regulares
- **Nuevos casos**: Asignación automática
- **Recordatorios**: Casos pendientes
- **Resúmenes**: Reportes diarios/semanales
- **Actualizaciones**: Cambios en el sistema

## 📱 Interfaz Móvil

### WhatsApp/Telegram Bot
```
🤖 BH Bot: Nuevo caso asignado
📋 ID: BH-2024-001234
👤 Usuario: Juan Pérez
📄 Tipo: Documento sospechoso
⏰ Prioridad: Alta
🔗 Ver caso: [Enlace]
```

### Comandos Disponibles
- `/casos` - Ver casos asignados
- `/estadisticas` - Métricas personales
- `/ayuda` - Comandos disponibles
- `/estado` - Estado del sistema

## 🔐 Seguridad y Auditoría

### Logs de Actividad
- **Acceso a casos**: Quién accedió y cuándo
- **Decisiones tomadas**: Aprobación/rechazo con justificación
- **Tiempo de revisión**: Duración de cada caso
- **Cambios de estado**: Historial completo

### Permisos y Roles
- **Revisor Junior**: Casos de baja prioridad
- **Revisor Senior**: Todos los casos
- **Supervisor**: Revisión de decisiones
- **Administrador**: Configuración del sistema

## 📈 Reportes y Analytics

### Reportes Diarios
- **Casos procesados**: Por revisor y total
- **Tiempo promedio**: De resolución
- **Tasas de aprobación**: Por tipo de caso
- **Alertas generadas**: Críticas y regulares

### Reportes Semanales
- **Tendencias**: Patrones de escalamiento
- **Efectividad**: De reglas automáticas
- **Rendimiento**: Del equipo de revisores
- **Mejoras**: Oportunidades identificadas

## Próximos Pasos

- [Panel de Revisión](admin-panel.md)
- [Reglas de Escalamiento](escalation-rules.md)
- [Integración con APIs](api-integration.md)
