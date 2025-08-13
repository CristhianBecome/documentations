# API de Verificación ANI Compliance

**Bienvenido a la documentación oficial de la API de Verificación ANI Compliance.** Esta API permite a los desarrolladores integrar fácilmente la verificación de identidad en sus aplicaciones mediante consulta directa a la Registraduría Nacional del Estado Civil de Colombia. Al proporcionar los datos personales del usuario, la API retorna una confirmación de veracidad basada en los registros oficiales colombianos. La API está diseñada para ayudar a las empresas a cumplir con los requisitos de validación de identidad y normativas de compliance locales.

## Características principales

- ✅ Verificación directa con registros oficiales
- ✅ Respuesta en tiempo real
- ✅ Autenticación segura con JWT
- ✅ Cumplimiento normativo colombiano
- ✅ Fácil integración

## URLs Base

- **Producción:** `https://api.svi.becomedigital.net/api/v1`
- **Desarrollo:** `https://api.dev.svi.becomedigital.net/api/v1`

## Comenzar

1. [Obtener credenciales de acceso](authentication.md)
2. [Realizar primera verificación](endpoints/verification.md)
3. [Manejar respuestas y errores](error-codes.md)

---

# SUMMARY.md

# Table of contents

* [Introducción](README.md)
* [Autenticación](authentication.md)
* [Endpoints](endpoints/README.md)
  * [Autenticación JWT](endpoints/auth.md)
  * [Verificación ANI](endpoints/verification.md)
* [Códigos de Error](error-codes.md)
* [Ejemplos de Integración](examples.md)

---

# authentication.md

# Autenticación

Para acceder a la API de Verificación ANI Compliance, es necesario autenticarse utilizando un token de acceso. Este endpoint de autenticación valida las credenciales del cliente y retorna un token JWT que debe ser incluido en todas las solicitudes posteriores.

## Flujo de autenticación

1. **Obtener credenciales:** Solicita tu `client_id` y `client_secret` al equipo de soporte
2. **Generar token:** Usa el endpoint `/auth` para obtener un JWT
3. **Usar token:** Incluye el JWT en el header `Authorization` de todas las requests

## Seguridad

- ⚠️ **Nunca expongas** el `client_secret` en código frontend
- 🔒 **Almacena de forma segura** los tokens de acceso
- ⏰ **Los tokens expiran** cada 1 hora, renuévalos automáticamente
- 🔄 **Implementa retry logic** para renovación automática

## Siguientes pasos

- [Ver endpoint de autenticación →](endpoints/auth.md)
- [Realizar primera verificación →](endpoints/verification.md)

---

# endpoints/README.md

# Endpoints

La API de Verificación ANI Compliance cuenta con los siguientes endpoints principales:

## Autenticación
- **POST** `/auth` - Obtener token de acceso JWT

## Verificación
- **POST** `/aniCompliance` - Verificar identidad con registros oficiales

## Base URLs

| Ambiente | URL Base |
|----------|----------|
| Producción | `https://api.svi.becomedigital.net/api/v1` |
| Desarrollo | `https://api.dev.svi.becomedigital.net/api/v1` |

## Headers requeridos

Todas las requests (excepto `/auth`) deben incluir:

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

---

# endpoints/auth.md

# Autenticación JWT

Endpoint para obtener el token de acceso requerido para todas las operaciones de la API.

## POST `/auth`

### Request

**Headers:**
```http
Content-Type: application/json
```

**Body:**
```json
{
  "client_id": "tu_client_id",
  "client_secret": "tu_client_secret"
}
```

### Ejemplo de Request

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/auth' \
--header 'Content-Type: application/json' \
--data '{
    "client_id": "demo_client_123",
    "client_secret": "demo_secret_456_ABCDEFGH"
}'
```

### Response Exitoso (200)

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJkZW1vX2NsaWVudF8xMjMiLCJpYXQiOjE2ODg1NzQ0MDAsImV4cCI6MTY4ODU3ODAwMH0.demo_signature",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### Errores comunes

| Código | Descripción |
|--------|-------------|
| 401 | Credenciales inválidas |
| 400 | Formato de request inválido |
| 429 | Demasiadas requests |

### Ejemplo de uso del token

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/aniCompliance' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...' \
--data '{ ... }'
```

---

# endpoints/verification.md

# Verificación ANI Compliance

Endpoint principal para verificar la identidad de una persona contra los registros oficiales de la Registraduría Nacional del Estado Civil de Colombia.

## POST `/aniCompliance`

### Request

**Headers:**
```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

**Body:**
```json
{
  "document_number": 12345678,
  "contract_id": 100,
  "departamentoExpedicion": "VALLE",
  "fechaExpedicion": "15/03/1990",
  "municipioExpedicion": "CALI",
  "primerApellido": "GARCIA",
  "segundoNombre": "MARIA",
  "primerNombre": "JUAN",
  "segundoApellido": "LOPEZ"
}
```

### Parámetros

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `document_number` | number | ✅ | Número de cédula de ciudadanía |
| `contract_id` | number | ✅ | ID del contrato asignado |
| `departamentoExpedicion` | string | ✅ | Departamento de expedición |
| `fechaExpedicion` | string | ✅ | Fecha de expedición (DD/MM/YYYY) |
| `municipioExpedicion` | string | ✅ | Municipio de expedición |
| `primerApellido` | string | ✅ | Primer apellido |
| `segundoNombre` | string | ❌ | Segundo nombre (opcional) |
| `primerNombre` | string | ✅ | Primer nombre |
| `segundoApellido` | string | ❌ | Segundo apellido (opcional) |

### Ejemplo de Request

```bash
curl --location 'https://api.dev.svi.becomedigital.net/api/v1/aniCompliance' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer tu_jwt_token_aqui' \
--data '{
    "document_number": 12345678,
    "contract_id": 100,
    "departamentoExpedicion": "VALLE",
    "fechaExpedicion": "15/03/1990",
    "municipioExpedicion": "CALI",
    "primerApellido": "GARCIA",
    "segundoNombre": "MARIA",
    "primerNombre": "JUAN",
    "segundoApellido": "LOPEZ"
}'
```

### Response

*Pendiente: Agregar ejemplos de respuesta cuando estén disponibles*

---

# error-codes.md

# Códigos de Error

La API utiliza códigos de estado HTTP estándar para indicar el éxito o fallo de las requests.

## Códigos de Estado

### 2xx - Éxito
| Código | Descripción |
|--------|-------------|
| 200 | Request procesado exitosamente |
| 201 | Recurso creado exitosamente |

### 4xx - Errores del Cliente
| Código | Descripción |
|--------|-------------|
| 400 | Bad Request - Formato de request inválido |
| 401 | Unauthorized - Token inválido o expirado |
| 403 | Forbidden - Sin permisos para el recurso |
| 404 | Not Found - Endpoint no encontrado |
| 422 | Unprocessable Entity - Datos de entrada inválidos |
| 429 | Too Many Requests - Rate limit excedido |

### 5xx - Errores del Servidor
| Código | Descripción |
|--------|-------------|
| 500 | Internal Server Error - Error interno del servidor |
| 502 | Bad Gateway - Error de gateway |
| 503 | Service Unavailable - Servicio temporalmente no disponible |

## Formato de Error

Todos los errores retornan un objeto JSON con la siguiente estructura:

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "El número de documento es requerido",
    "details": "El campo 'document_number' no puede estar vacío"
  }
}
```

## Errores Comunes

### Autenticación
- **TOKEN_EXPIRED**: El token JWT ha expirado
- **INVALID_CREDENTIALS**: Client ID o secret inválidos
- **MISSING_TOKEN**: Token de autorización no proporcionado

### Validación
- **MISSING_REQUIRED_FIELD**: Campo requerido faltante
- **INVALID_FORMAT**: Formato de datos incorrecto
- **INVALID_DATE**: Fecha en formato incorrecto

### Rate Limiting
- **RATE_LIMIT_EXCEEDED**: Se ha excedido el límite de requests por minuto

---

# examples.md

# Ejemplos de Integración

Ejemplos prácticos de cómo integrar la API de Verificación ANI Compliance en diferentes lenguajes de programación.

## JavaScript/Node.js

```javascript
const axios = require('axios');

class ANIComplianceAPI {
  constructor(clientId, clientSecret, baseURL = 'https://api.svi.becomedigital.net/api/v1') {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.baseURL = baseURL;
    this.token = null;
  }

  async authenticate() {
    try {
      const response = await axios.post(`${this.baseURL}/auth`, {
        client_id: this.clientId,
        client_secret: this.clientSecret
      });
      
      this.token = response.data.access_token;
      return this.token;
    } catch (error) {
      throw new Error(`Autenticación fallida: ${error.message}`);
    }
  }

  async verifyIdentity(personData) {
    if (!this.token) {
      await this.authenticate();
    }

    try {
      const response = await axios.post(`${this.baseURL}/aniCompliance`, personData, {
        headers: {
          'Authorization': `Bearer ${this.token}`,
          'Content-Type': 'application/json'
        }
      });
      
      return response.data;
    } catch (error) {
      throw new Error(`Verificación fallida: ${error.message}`);
    }
  }
}

// Uso
const api = new ANIComplianceAPI('tu_client_id', 'tu_client_secret');

const personData = {
  document_number: 12345678,
  contract_id: 100,
  departamentoExpedicion: "VALLE",
  fechaExpedicion: "15/03/1990",
  municipioExpedicion: "CALI",
  primerApellido: "GARCIA",
  segundoNombre: "MARIA",
  primerNombre: "JUAN",
  segundoApellido: "LOPEZ"
};

api.verifyIdentity(personData)
  .then(result => console.log('Verificación exitosa:', result))
  .catch(error => console.error('Error:', error));
```

## Python

```python
import requests
import json

class ANIComplianceAPI:
    def __init__(self, client_id, client_secret, base_url='https://api.svi.becomedigital.net/api/v1'):
        self.client_id = client_id
        self.client_secret = client_secret
        self.base_url = base_url
        self.token = None

    def authenticate(self):
        url = f"{self.base_url}/auth"
        payload = {
            "client_id": self.client_id,
            "client_secret": self.client_secret
        }
        
        response = requests.post(url, json=payload)
        
        if response.status_code == 200:
            self.token = response.json()['access_token']
            return self.token
        else:
            raise Exception(f"Autenticación fallida: {response.text}")

    def verify_identity(self, person_data):
        if not self.token:
            self.authenticate()
        
        url = f"{self.base_url}/aniCompliance"
        headers = {
            'Authorization': f'Bearer {self.token}',
            'Content-Type': 'application/json'
        }
        
        response = requests.post(url, json=person_data, headers=headers)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Verificación fallida: {response.text}")

# Uso
api = ANIComplianceAPI('tu_client_id', 'tu_client_secret')

person_data = {
    "document_number": 12345678,
    "contract_id": 100,
    "departamentoExpedicion": "VALLE",
    "fechaExpedicion": "15/03/1990",
    "municipioExpedicion": "CALI",
    "primerApellido": "GARCIA",
    "segundoNombre": "MARIA",
    "primerNombre": "JUAN",
    "segundoApellido": "LOPEZ"
}

try:
    result = api.verify_identity(person_data)
    print("Verificación exitosa:", result)
except Exception as e:
    print("Error:", e)
```

## PHP

```php
<?php

class ANIComplianceAPI {
    private $clientId;
    private $clientSecret;
    private $baseURL;
    private $token;

    public function __construct($clientId, $clientSecret, $baseURL = 'https://api.svi.becomedigital.net/api/v1') {
        $this->clientId = $clientId;
        $this->clientSecret = $clientSecret;
        $this->baseURL = $baseURL;
    }

    public function authenticate() {
        $url = $this->baseURL . '/auth';
        $data = [
            'client_id' => $this->clientId,
            'client_secret' => $this->clientSecret
        ];

        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);

        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        if ($httpCode === 200) {
            $result = json_decode($response, true);
            $this->token = $result['access_token'];
            return $this->token;
        } else {
            throw new Exception("Autenticación fallida: " . $response);
        }
    }

    public function verifyIdentity($personData) {
        if (!$this->token) {
            $this->authenticate();
        }

        $url = $this->baseURL . '/aniCompliance';

        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($personData));
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, [
            'Content-Type: application/json',
            'Authorization: Bearer ' . $this->token
        ]);

        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        if ($httpCode === 200) {
            return json_decode($response, true);
        } else {
            throw new Exception("Verificación fallida: " . $response);
        }
    }
}

// Uso
$api = new ANIComplianceAPI('tu_client_id', 'tu_client_secret');

$personData = [
    'document_number' => 12345678,
    'contract_id' => 100,
    'departamentoExpedicion' => 'VALLE',
    'fechaExpedicion' => '15/03/1990',
    'municipioExpedicion' => 'CALI',
    'primerApellido' => 'GARCIA',
    'segundoNombre' => 'MARIA',
    'primerNombre' => 'JUAN',
    'segundoApellido' => 'LOPEZ'
];

try {
    $result = $api->verifyIdentity($personData);
    echo "Verificación exitosa: " . json_encode($result, JSON_PRETTY_PRINT);
} catch (Exception $e) {
    echo "Error: " . $e->getMessage();
}
?>
```

---

# .gitbook.yaml

root: ./
structure:
  readme: README.md
  summary: SUMMARY.md