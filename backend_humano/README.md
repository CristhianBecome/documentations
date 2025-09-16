# Backend Humano (BH) - Guía para Revisores

**Bienvenido a la guía del Backend Humano.** Aquí aprenderás cómo revisar los casos que la máquina no puede aprobar automáticamente.

## 🎯 ¿Qué es Backend Humano?

Backend Humano es el **equipo de revisores** que revisa los casos cuando la máquina no está segura. Es como cuando en el banco necesitan que un supervisor revise algo.

### ¿Cómo funciona?

1. **Primero** la máquina revisa el documento automáticamente
2. **Después** decide: ¿Puedo aprobar esto o necesito que un humano lo revise?

### ✅ Casos que la máquina aprueba sola:
- Todo se ve bien y normal
- El documento parece real
- La foto coincide con la persona
- No hay nada sospechoso

### ⚠️ Casos que requieren Revisión Humana (BH):

- **📄 Documento con anomalías** – El sistema no logra confirmar la autenticidad del documento (posible alteración, baja calidad o patrones fuera de lo esperado).

- **👤 Desajuste en biometría facial** – La imagen del documento no coincide con la selfie capturada.

- **📱 Falla en prueba de vida** – No se validó satisfactoriamente que la persona esté presente y sea real.

- **🚨 Historial con incidencias previas** – El usuario presenta alertas o reportes en procesos anteriores.

- **🇨🇴 Excepción en documentos colombianos** – Para cédulas de Colombia aplican validaciones más estrictas; si alguna regla falla, se envía a BH.

- **🔍 Detección de patrones sospechosos** – El sistema identifica señales de riesgo o comportamientos inusuales que generan duda en la validación automática.

## 📱 ¿Qué pasa cuando llega un caso?

1. **Para clientes según configuración del contrato**: Se envía alerta al grupo de Telegram centralizado
2. **Para otros casos**: Aparecen como pendientes en el dashboard
3. **El líder de BH** decide quién tiene acceso a las notificaciones
4. **Revisas** toda la información del caso
5. **Decides** si apruebas o rechazas
6. **Se notifica** al usuario el resultado

## 📋 Lo que necesitas saber

- ✅ **Alertas centralizadas** en grupo de Telegram (solo clientes según configuración del contrato)
- ✅ **Dashboard** para ver casos pendientes
- ✅ **Acceso controlado** por el líder de BH
- ✅ **Reglas claras** sobre cuándo revisar
- ✅ **Se registra** todo lo que haces

## 💡 En palabras simples:

**BH (Backend Humano) es como el "equipo de supervisión"** - cuando la máquina no está 100% segura de que todo está bien, lo manda a que un humano lo revise.

Es como cuando compras algo caro y el cajero llama al supervisor - no es que esté mal, solo necesita una segunda opinión humana.

### ¿Por qué existe?
Para evitar que pasen documentos falsos, personas que no son quienes dicen ser, o cualquier cosa sospechosa.

### ¿Es malo ir a BH?
**No necesariamente** - a veces es solo porque la foto salió borrosa o el documento se ve un poco gastado, pero está bien.

## 🚀 Guías para Revisores

1. [Cuándo revisar un caso](escalation-rules.md)
2. [Tipos de documentos aceptados](document-types.md)
3. [Qué buscar en los documentos](alteration-reasons.md)

