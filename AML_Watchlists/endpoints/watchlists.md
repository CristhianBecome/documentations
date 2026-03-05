# Verificación AML Watchlists

Endpoint para consultar si una persona se encuentra en listas restrictivas AML.

## POST `/watchlists`

### Headers requeridos

```http
Content-Type: application/json
Authorization: Bearer <tu_jwt_token>
```

### Parámetros del request

```json
{
  "documentNumber": "70041053",
  "contractId": 173,
  "fullName": "alvaro uribe velez",
  "country": "COLOMBIA",
  "birth": "1985-09-13"
}
```

### Ejemplo de solicitud

```bash
curl --location 'https://api.svi.becomedigital.net/api/v1/watchlists' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <tu_jwt_token>' \
--data '{
    "documentNumber": "70041053",
    "contractId": 173,
    "fullName": "alvaro uribe velez",
    "country": "COLOMBIA",
    "birth": "1985-09-13"
}'
```

---

## Respuesta exitosa

### HTTP 200 OK

```json
{
  "data": {
    "check": false,
    "data": {
      "block": true,
      "datos_amlnews": [
        {
          "buscado": "alvaro uribe velez",
          "fecha": "October 2025",
          "news_paper": "TRT Español",
          "title": "La justicia colombiana revoca la condena al expresidente Álvaro Uribe por soborno y fraude procesal",
          "url": "https://news.google.com/..."
        }
      ],
      "datos_listas": "3699,2776,1381,...",
      "datos_pro": null,
      "datos_procuraduria": [],
      "datos_ramajudicial": null,
      "datos_registraduria": false,
      "datos_tsti": [
        {
          "_version_": 1858816276772683777,
          "alias": [""],
          "categoria": ["Peps Colombia"],
          "ciudadania": ["Colombiana"],
          "compania": ["Senado de la Republica"],
          "detalle": ["Ex Senador de la República de Colombia..."],
          "estado": null,
          "estado1": null,
          "estado2": null,
          "estado3": null,
          "etiqueta": {
            "color": "#5bfa05",
            "nombre": "PEPS"
          },
          "id": "1314734",
          "id_relacion_lista": ["93"],
          "lista": ["4223"],
          "n_identificacion": ["70041053"],
          "n_identificacion2": ["70041053"],
          "nombre_apellido": ["ALVARO URIBE VELEZ"],
          "nombre_relacion_lista": ["PEPS Colombia"],
          "pais": ["COLOMBIA"],
          "pasaporte": [""],
          "pasaporte2": [""],
          "relacionado": "[]",
          "url": "http://www.senado.gov.co/el-senado/senadores"
        }
      ],
      "datos_twitter": null,
      "doc_id": "70041053",
      "item_no": 2,
      "nombre": "alvaro uribe velez"
    },
    "error": null
  }
}
```

### Descripción de campos de la respuesta

| Campo | Tipo | Descripción |
| ------- | ------ | ----------- |
| `data.check` | boolean | Indica si la verificación fue positiva sin coincidencias en listas restrictivas. `false` significa que se encontraron coincidencias. |
| `data.error` | string \| null | Mensaje de error si ocurrió alguno durante la consulta. `null` si no hubo errores. |
| `data.data.block` | boolean | Indica si la persona debe ser bloqueada. `true` si aparece en listas restrictivas. |
| `data.data.nombre` | string | Nombre consultado. |
| `data.data.doc_id` | string | Número de documento consultado. |
| `data.data.item_no` | number | Número de ítem en el proceso de verificación. |
| `data.data.datos_amlnews` | array | Noticias encontradas relacionadas con la persona consultada. |
| `data.data.datos_amlnews[].buscado` | string | Nombre que se buscó en la fuente de noticias. |
| `data.data.datos_amlnews[].fecha` | string | Fecha aproximada de la noticia. |
| `data.data.datos_amlnews[].news_paper` | string | Medio de comunicación que publicó la noticia. |
| `data.data.datos_amlnews[].title` | string | Título de la noticia. |
| `data.data.datos_amlnews[].url` | string | URL de la noticia. |
| `data.data.datos_listas` | string | IDs de listas restrictivas donde aparece la persona, separados por coma. |
| `data.data.datos_pro` | array \| null | Registros de procesos adicionales. Array vacío o `null` si no hay coincidencias. |
| `data.data.datos_procuraduria` | array | Registros encontrados en la Procuraduría General de la Nación. Array vacío si no hay coincidencias. |
| `data.data.datos_ramajudicial` | array \| null | Registros encontrados en la Rama Judicial. Array vacío o `null` si no hay coincidencias. |
| `data.data.datos_registraduria` | boolean | Indica si hay registro en la Registraduría Nacional. `false` si no hay coincidencias. |
| `data.data.datos_tsti` | array | Registros encontrados en listas TSTI (personas expuestas políticamente y otras listas especializadas). |
| `data.data.datos_tsti[].id` | string | Identificador único del registro en la lista. |
| `data.data.datos_tsti[].nombre_apellido` | array | Nombre completo de la persona. |
| `data.data.datos_tsti[].n_identificacion` | array | Número(s) de identificación registrados. |
| `data.data.datos_tsti[].categoria` | array | Categoría de la lista donde aparece (ej. `"Peps Colombia"`). |
| `data.data.datos_tsti[].ciudadania` | array | Ciudadanía registrada. |
| `data.data.datos_tsti[].compania` | array | Organización o entidad con la que se relaciona. |
| `data.data.datos_tsti[].detalle` | array | Descripción detallada de la vinculación de la persona con la lista. |
| `data.data.datos_tsti[].etiqueta` | object | Etiqueta visual asignada al tipo de lista. |
| `data.data.datos_tsti[].etiqueta.color` | string | Color hexadecimal de la etiqueta. |
| `data.data.datos_tsti[].etiqueta.nombre` | string | Nombre de la etiqueta (ej. `"PEPS"`). |
| `data.data.datos_tsti[].lista` | array | ID(s) de la lista donde aparece el registro. |
| `data.data.datos_tsti[].nombre_relacion_lista` | array | Nombre de la lista de relación. |
| `data.data.datos_tsti[].pais` | array | País de origen del registro. |
| `data.data.datos_tsti[].relacionado` | string | JSON con entidades relacionadas. |
| `data.data.datos_tsti[].url` | string | URL de la fuente del registro. |
| `data.data.datos_twitter` | object \| null | Datos de redes sociales relacionados. `null` si no aplica. |

---

## Respuesta sin incidencias

Cuando la persona consultada **no aparece** en ninguna lista restrictiva, `check` es `true` y `block` es `false`. Todos los campos de resultados retornan vacíos.

### HTTP 200 OK — Sin incidencias

```json
{
  "data": {
    "check": true,
    "data": {
      "block": false,
      "datos_amlnews": [],
      "datos_listas": "3699,2776,1381,...",
      "datos_pro": [],
      "datos_procuraduria": [],
      "datos_ramajudicial": [],
      "datos_registraduria": false,
      "datos_tsti": [],
      "datos_twitter": null,
      "doc_id": "1116276856",
      "item_no": 2,
      "nombre": "cristhian rendon sanchez"
    },
    "error": null
  }
}
```

---

## Respuestas de error

### 400 Bad Request

**Faltan datos o body vacío:**

```json
{
  "msg": "faltan algunos datos, por favor completalos",
  "status": 400
}
```

Con detalle de campos faltantes:

```json
{
  "msg": "faltan algunos datos, por favor completalos",
  "missing_keys": ["documentNumber"],
  "status": 400
}
```

**Contrato no encontrado:**

```json
{
  "msg": "No se encontro el contrato para la compañia especificada",
  "status": 400
}
```

**Formato de fecha inválido:**

```json
{
  "msg": "La fecha no está en el formato correcto (YYYY-MM-DD)",
  "status": 400
}
```

**Error interno no controlado:**

```json
{
  "msg": "Internal server error",
  "status": 400
}
```

### 401 / 422 Unauthorized / Unprocessable Entity

Retornados por el decorador `@jwt_required()` cuando el token JWT es inválido, está ausente o mal formado. El formato de respuesta es el estándar de `flask_jwt_extended`.
