# API CURP México — GovChecks

**Documentación de la API standalone de validación de CURP mexicano.** Permite integrar la consulta de identidad frente a los registros oficiales de RENAPO (Registro Nacional de Población) en aplicaciones con autenticación JWT.

Al enviar el CURP y el contrato, la API retorna el resultado de la consulta según los registros oficiales mexicanos. Existen dos versiones de respuesta (`v1` y `v2`) con contratos HTTP distintos.

## Características principales

- Consulta frente a registros oficiales de RENAPO
- Dos versiones de respuesta (`v1` y `v2`)
- Respuestas en tiempo real
- Autenticación mediante tokens JWT
- Validación de formato CURP
- Integración sencilla

## URLs base

- **Producción:** `https://api.svi.becomedigital.net/api/v1`

## Primeros pasos

1. [Obtener credenciales de acceso](authentication.md)
2. [Realizar validación CURP](endpoints/curp.md)

## Requisitos del contrato

El contrato debe tener habilitada la validación CURP (`can_mexican_curp`).
