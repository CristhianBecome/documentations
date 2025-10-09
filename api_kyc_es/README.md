# API de Verificación KYC - Become Digital

**Bienvenido a la documentación oficial de la API de Verificación KYC de Become Digital.** Esta API le permite verificar la identidad de sus usuarios de manera digital y remota de forma fácil y completamente interoperable. La API está diseñada para que su empresa cumpla con regulaciones internacionales, locales y todo lo relacionado con KYC (Know Your Customer) / AML (Anti-Money Laundering).

## Características principales

- ✅ Verificación de documentos de identidad (cédulas, pasaportes, licencias de conducir)
- ✅ Prueba de vida (liveness detection) y cotejo facial
- ✅ Validación de números telefónicos con Telco
- ✅ Verificación de correos electrónicos
- ✅ Consultas a registros gubernamentales (ANI Colombia)
- ✅ Detección de alteraciones y fraudes en documentos
- ✅ Autenticación segura mediante tokens JWT
- ✅ Proceso de verificación asíncrono
- ✅ Integración mediante SDK o API directa

## URLs base

- **Producción:** `https://api.svi.becomedigital.net/api/v1`

## Métodos de Verificación

Ofrecemos dos maneras de ejecutar el flujo de verificación:

1. **SDKs** para integración en dispositivos móviles o web
2. **API** para realizar llamados directos al servicio

## Verificación en 3 Pasos Sencillos

1. **Autenticación:** Acceso seguro al servicio mediante credenciales
2. **Envío:** Carga de los documentos o información necesaria
3. **Consulta de resultados:** Obtención de los resultados de validación de forma asíncrona

> **Nota importante:** El proceso de validación es **asíncrono**, lo que significa que la respuesta del endpoint de consulta de resultados cambiará según avance la validación.

## Formas de Identificación Soportadas

- **Identificación Nacional** (cédulas de ciudadanía, DNI, INE, etc.)
- **Pasaporte**
- **Licencia de Conducir**

## Servicios Adicionales

### Verificaciones Gubernamentales
- **ANI Colombia:** Validación de cédulas de ciudadanía con registros de la Registraduría Nacional

### Validaciones Complementarias
- **Telco:** Verificación de números telefónicos, detección de cambios de SIM, análisis de riesgo
- **Email:** Validación de correos electrónicos, detección de spam traps y riesgo de fraude

## Casos de Uso

Nuestra API está diseñada para ayudar a las empresas a:

- ✅ Cumplir con las normativas y regulaciones de KYC/AML
- ✅ Facilitar la validación de identidad digital de sus usuarios
- ✅ Prevenir fraudes y suplantación de identidad
- ✅ Automatizar procesos de onboarding de clientes
- ✅ Re-autenticación de usuarios sin necesidad de documentos

## Primeros pasos

1. [Obtener credenciales de acceso](authentication.md)
2. [Autenticarse y obtener token JWT](endpoints/auth.md)
3. [Realizar primera verificación de identidad](endpoints/verification-add.md)
4. [Consultar resultados de verificación](endpoints/verification-results.md)

## Soporte

Para obtener credenciales de acceso o asistencia técnica, contacte al equipo de soporte de Become Digital.

Visite nuestro sitio web: [https://becomedigital.ai/](https://becomedigital.ai/)

