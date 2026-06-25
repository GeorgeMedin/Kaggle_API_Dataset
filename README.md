# API REST-like para Consulta de Datasets CSV

API HTTP estilo REST-like construida con **Google Apps Script** para registrar, listar, describir y consultar datasets CSV a partir de URLs públicas.

> Proyecto creado con fines de **estudio y práctica**, especialmente para aprender a consumir APIs desde herramientas y lenguajes como:
>
> - Python
> - TypeScript / JavaScript
> - Excel
> - Power BI
> - Postman

---

## Base URL

```txt
{{base_url}}
```

Ejemplo real:

```txt
https://script.google.com/macros/s/AKfycbxjF06rPJX5iJ0sDRu3HZsysXG2m7gD2ybL6rtFYXjEF84KZ4PdMSUy5uOz1VocWlz7BQ/exec
```

---

# Tabla de contenido

- [Características](#características)
- [Arquitectura general](#arquitectura-general)
- [Autenticación](#autenticación)
- [Endpoints](#endpoints)
  - [1. Generar API Key](#1-generar-api-key)
  - [2. Registrar dataset](#2-registrar-dataset)
  - [3. Listar datasets](#3-listar-datasets)
  - [4. Describir columnas de un dataset](#4-describir-columnas-de-un-dataset)
  - [5. Consultar dataset](#5-consultar-dataset)
  - [6. Exportar resultado como CSV](#6-exportar-resultado-como-csv)
- [Parámetros de consulta soportados](#parámetros-de-consulta-soportados)
- [Ejemplos de uso](#ejemplos-de-uso)
  - [Postman](#postman)
  - [Python](#python)
  - [TypeScript / JavaScript](#typescript--javascript)
  - [Power BI (Power Query)](#power-bi-power-query)
  - [Excel](#excel)
- [Estructura de datos interna](#estructura-de-datos-interna)
- [Formato de respuestas](#formato-de-respuestas)
- [Limitaciones actuales](#limitaciones-actuales)
- [Compatibilidad legacy](#compatibilidad-legacy)
- [Notas técnicas](#notas-técnicas)
- [Recomendaciones de uso](#recomendaciones-de-uso)
- [Ejemplos completos](#ejemplos-completos)
- [Autor](#autor)
- [Licencia](#licencia)

---

# Características

- Generación de **API Key** a partir de usuario y contraseña.
- Registro de datasets CSV públicos por URL.
- Listado de datasets registrados.
- Descripción automática de columnas:
  - nombre
  - tipo inferido
  - cantidad de nulos
- Consulta de datasets con soporte para:
  - filtros
  - selección de columnas
  - ordenamiento
  - límite de filas
  - agrupación con `SUM(...)`
- Exportación del resultado como **CSV**
- Compatible con consumo vía `GET`

---

# Arquitectura general

La API usa dos hojas dentro de un Google Spreadsheet:

- **users**
- **data**

El flujo general es:

1. El usuario obtiene sus credenciales por otro proceso externo.
2. Usa `user` + `pass` para generar una `api_key`.
3. Usa la `api_key` para acceder al resto de acciones.
4. Registra datasets CSV públicos.
5. Consulta o describe datasets registrados.

---

# Autenticación

La única acción pública es la generación de API Key.

## Flujo

### Paso 1: obtener credenciales
Debes contar previamente con:

- `user` o `email`
- `password`

### Paso 2: generar API Key
Con esas credenciales puedes solicitar una API key.

### Paso 3: usar la API Key
A partir de ese momento, debes enviar:

```txt
api_key=TU_API_KEY
```

en las acciones protegidas.

---

# Endpoints

## 1. Generar API Key

Genera una API key válida a partir de un usuario/email y contraseña.

### Endpoint

```txt
{{base_url}}?action=create_key&user=TU_USUARIO_O_EMAIL&pass=TU_PASSWORD
```

### Parámetros

- `action=create_key`
- `user`: username o email
- `pass`: contraseña

### Ejemplo

```txt
{{base_url}}?action=create_key&user=jorge@email.com&pass=123456
```

### Respuesta exitosa

```json
{
  "ok": true,
  "message": "API key generada correctamente",
  "data": {
    "user": "jorge",
    "email": "jorge@email.com",
    "api_key": "AbC123xYz456LmN789Qw"
  }
}
```

### Respuesta con error

```json
{
  "ok": false,
  "error": "Unauthorized",
  "message": "Usuario o contraseña inválidos"
}
```

### Compatibilidad legacy

También soporta:

```txt
{{base_url}}?action=getkey&user=TU_USUARIO&pass=TU_PASSWORD
```

---

## 2. Registrar dataset

Registra o actualiza un dataset CSV a partir de una URL pública.

### Endpoint

```txt
{{base_url}}?action=register_dataset&api_key=TU_API_KEY&url=URL_DEL_CSV
```

### Parámetros

- `action=register_dataset`
- `api_key`: llave válida
- `url`: URL pública del CSV

### Ejemplo

```txt
{{base_url}}?action=register_dataset&api_key=TU_API_KEY&url=https://raw.githubusercontent.com/usuario/repositorio/main/archivo.csv
```

### Respuesta si el dataset es nuevo

```json
{
  "ok": true,
  "message": "Dataset registrado",
  "data": {
    "name": "archivo.csv",
    "url": "https://raw.githubusercontent.com/usuario/repositorio/main/archivo.csv"
  }
}
```

### Respuesta si el dataset ya existía

```json
{
  "ok": true,
  "message": "Dataset actualizado",
  "data": {
    "name": "archivo.csv",
    "url": "https://raw.githubusercontent.com/usuario/repositorio/main/archivo.csv"
  }
}
```

### Error de validación

```json
{
  "ok": false,
  "error": "Validation Error",
  "message": "La URL no apunta a un archivo CSV"
}
```

### Compatibilidad legacy

También soporta:

```txt
{{base_url}}?api_key=TU_API_KEY&register_data=URL_DEL_CSV
```

---

## 3. Listar datasets

Devuelve todos los datasets registrados.

### Endpoint

```txt
{{base_url}}?action=list_datasets&api_key=TU_API_KEY
```

### Parámetros

- `action=list_datasets`
- `api_key`: llave válida

### Ejemplo

```txt
{{base_url}}?action=list_datasets&api_key=TU_API_KEY
```

### Respuesta

```json
{
  "ok": true,
  "count": 2,
  "data": [
    {
      "id": "e8f21f9c-4bfe-4b95-81fd-xxxxxxxxxxxx",
      "url": "https://raw.githubusercontent.com/usuario/repo/main/ventas.csv",
      "name": "ventas.csv"
    },
    {
      "id": "4bada3d0-c7c0-4a3d-bf9e-xxxxxxxxxxxx",
      "url": "https://raw.githubusercontent.com/usuario/repo/main/clientes.csv",
      "name": "clientes.csv"
    }
  ]
}
```

### Compatibilidad legacy

También soporta:

```txt
{{base_url}}?api_key=TU_API_KEY&history_data
```

---

## 4. Describir columnas de un dataset

Analiza el dataset y devuelve metadatos básicos de sus columnas.

### Endpoint

```txt
{{base_url}}?action=describe_dataset&api_key=TU_API_KEY&dataset=archivo.csv
```

### Parámetros

- `action=describe_dataset`
- `api_key`: llave válida
- `dataset`: nombre exacto del archivo registrado

### Ejemplo

```txt
{{base_url}}?action=describe_dataset&api_key=TU_API_KEY&dataset=ventas.csv
```

### Respuesta

```json
{
  "ok": true,
  "count": 4,
  "source": "https://raw.githubusercontent.com/usuario/repo/main/ventas.csv",
  "data": [
    {
      "column": "Category",
      "type": "string",
      "nulls": 0
    },
    {
      "column": "Price",
      "type": "number",
      "nulls": 2
    },
    {
      "column": "Quantity",
      "type": "number",
      "nulls": 0
    },
    {
      "column": "Country",
      "type": "string",
      "nulls": 1
    }
  ]
}
```

### Compatibilidad legacy

También soporta:

```txt
{{base_url}}?api_key=TU_API_KEY&dataset=ventas.csv&details-column
```

---

## 5. Consultar dataset

Consulta un dataset registrado y permite aplicar filtros, campos, ordenamiento, agrupación y límite.

### Endpoint base

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=archivo.csv
```

### Parámetros mínimos

- `action=query_dataset`
- `api_key`: llave válida
- `dataset`: nombre del dataset registrado

### Ejemplo simple

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv
```

### Respuesta

```json
{
  "ok": true,
  "count": 100,
  "source": "https://raw.githubusercontent.com/usuario/repo/main/ventas.csv",
  "data": [
    {
      "Product": "A",
      "Category": "Snacks",
      "Price": "10",
      "Country": "Colombia"
    }
  ]
}
```

---

## 6. Exportar resultado como CSV

Puedes exportar el resultado de una consulta en formato CSV.

### Endpoint

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&export=csv
```

### Ejemplo

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&fields=Product,Price&limit=10&export=csv
```

### Compatibilidad legacy

También soporta:

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&d-csv
```

---

# Parámetros de consulta soportados

Estos parámetros se usan principalmente con `action=query_dataset`.

---

## `filter`

Permite filtrar por igualdad exacta.

### Sintaxis

```txt
filter=columna:valor
```

### Múltiples condiciones

```txt
filter=Country:Colombia,Category:Snacks
```

### Ejemplo

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&filter=Country:Colombia
```

> Las condiciones se evalúan con lógica **AND**.

---

## `fields`

Permite seleccionar columnas específicas.

### Sintaxis

```txt
fields=col1,col2,col3
```

### Ejemplo

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&fields=Product,Price,Country
```

---

## `order_by`

Permite ordenar el resultado.

### Sintaxis

```txt
order_by=columna asc
```

o

```txt
order_by=columna desc
```

### Ejemplo

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&order_by=Price desc
```

---

## `limit`

Limita la cantidad de registros devueltos.

### Ejemplo

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&limit=20
```

---

## `group_by` + `agg`

Permite agrupar por una columna y aplicar una agregación.

> Actualmente solo se soporta:

```txt
agg=SUM(columna)
```

### Ejemplo

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&group_by=Category&agg=SUM(Price)
```

### Respuesta esperada

```json
{
  "ok": true,
  "count": 3,
  "source": "https://raw.githubusercontent.com/usuario/repo/main/ventas.csv",
  "data": [
    {
      "Category": "Snacks",
      "total": 250
    },
    {
      "Category": "Beverages",
      "total": 430
    },
    {
      "Category": "Dairy",
      "total": 180
    }
  ]
}
```

---

# Ejemplos de uso

## Postman

### Generar API Key

```http
GET {{base_url}}?action=create_key&user=TU_USUARIO&pass=TU_PASSWORD
```

### Registrar dataset

```http
GET {{base_url}}?action=register_dataset&api_key=TU_API_KEY&url=https://raw.githubusercontent.com/usuario/repo/main/ventas.csv
```

### Listar datasets

```http
GET {{base_url}}?action=list_datasets&api_key=TU_API_KEY
```

### Describir dataset

```http
GET {{base_url}}?action=describe_dataset&api_key=TU_API_KEY&dataset=ventas.csv
```

### Consultar dataset con filtros

```http
GET {{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&filter=Country:Colombia&fields=Product,Price&order_by=Price desc&limit=10
```

---

## Python

```python
import requests

base_url = "{{base_url}}"

params = {
    "action": "query_dataset",
    "api_key": "TU_API_KEY",
    "dataset": "ventas.csv",
    "filter": "Country:Colombia",
    "fields": "Product,Price,Country",
    "order_by": "Price desc",
    "limit": 10
}

response = requests.get(base_url, params=params)
data = response.json()

print(data)
```

### Generar API Key con Python

```python
import requests

base_url = "{{base_url}}"

params = {
    "action": "create_key",
    "user": "TU_USUARIO",
    "pass": "TU_PASSWORD"
}

response = requests.get(base_url, params=params)
print(response.json())
```

---

## TypeScript / JavaScript

```ts
const baseUrl = "{{base_url}}";

async function consultarDataset() {
  const url = new URL(baseUrl);

  url.searchParams.set("action", "query_dataset");
  url.searchParams.set("api_key", "TU_API_KEY");
  url.searchParams.set("dataset", "ventas.csv");
  url.searchParams.set("filter", "Country:Colombia");
  url.searchParams.set("fields", "Product,Price,Country");
  url.searchParams.set("order_by", "Price desc");
  url.searchParams.set("limit", "10");

  const response = await fetch(url.toString());
  const data = await response.json();

  console.log(data);
}

consultarDataset();
```

### Generar API Key con TypeScript / JavaScript

```ts
const baseUrl = "{{base_url}}";

async function generarApiKey() {
  const url = new URL(baseUrl);

  url.searchParams.set("action", "create_key");
  url.searchParams.set("user", "TU_USUARIO");
  url.searchParams.set("pass", "TU_PASSWORD");

  const response = await fetch(url.toString());
  const data = await response.json();

  console.log(data);
}

generarApiKey();
```

---

## Power BI (Power Query)

```powerquery
let
    BaseUrl = "{{base_url}}",
    Source = Json.Document(
        Web.Contents(
            BaseUrl,
            [
                Query = [
                    action = "query_dataset",
                    api_key = "TU_API_KEY",
                    dataset = "ventas.csv",
                    filter = "Country:Colombia",
                    fields = "Product,Price,Country",
                    order_by = "Price desc",
                    limit = "10"
                ]
            ]
        )
    ),
    Data = Source[data],
    TableResult = Table.FromRecords(Data)
in
    TableResult
```

---

## Excel

Si quieres consumir la API desde Excel usando **Power Query**:

1. Ve a **Datos**
2. Selecciona **Obtener datos**
3. Elige **Desde otras fuentes** > **Desde la Web**
4. Usa una URL como esta:

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&limit=20
```

Si necesitas un resultado tabular más simple, puedes usar:

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&fields=Product,Price,Country&limit=20
```

---

# Estructura de datos interna

## Hoja `users`

La hoja `users` debe tener la siguiente estructura:

| id | email | usuario | password | key |
|----|-------|---------|----------|-----|

### Descripción

- `id`: identificador interno del usuario
- `email`: correo del usuario
- `usuario`: nombre de usuario
- `password`: contraseña
- `key`: API key generada

---

## Hoja `data`

La hoja `data` debe tener la siguiente estructura:

| id | url | name |
|----|-----|------|

### Descripción

- `id`: UUID del dataset
- `url`: URL pública del CSV
- `name`: nombre del archivo CSV

---

# Formato de respuestas

## Respuesta exitosa genérica

```json
{
  "ok": true,
  "count": 10,
  "source": "https://...",
  "data": []
}
```

## Respuesta de error genérica

```json
{
  "ok": false,
  "error": "Bad Request",
  "message": "Descripción del error"
}
```

## Error de autenticación

```json
{
  "ok": false,
  "error": "Unauthorized",
  "message": "Debes enviar una API key válida en ?api_key=..."
}
```

---

# Limitaciones actuales

Esta API fue construida con fines educativos y tiene varias limitaciones intencionales:

- Solo soporta método **GET**
- Solo trabaja con archivos **CSV**
- El filtro es por **igualdad exacta**
- No soporta operadores como:
  - `>`
  - `<`
  - `!=`
  - `LIKE`
- La agregación actual solo soporta:

```txt
SUM(columna)
```

- No hay paginación
- No hay expiración automática de API keys
- No se manejan códigos HTTP personalizados
- La inferencia de tipo de columnas es básica
- La carga del CSV depende de que la URL sea pública y accesible

---

# Compatibilidad legacy

Además de la sintaxis recomendada con `action=...`, existen alias heredados:

- `action=getkey`
- `register_data=URL`
- `history_data`
- `data=archivo.csv`
- `details-column`
- `d-csv`

> Recomendación: usar siempre la sintaxis estándar basada en `action=...`.

---

# Notas técnicas

## Conversión automática de URLs GitHub

La API intenta normalizar URLs de GitHub:

- Si la URL ya viene en formato `raw.githubusercontent.com`, la usa directamente.
- Si recibe una URL tipo `github.com/.../blob/...`, intenta convertirla a formato raw.

## Exclusión de columnas

Durante la lectura del CSV, se excluyen las columnas:

- vacías (`""`)
- `ID`

## Detección de números

La API considera como numéricos valores como:

- `10`
- `10.5`
- `25 kg`
- `30 g`
- `50 lbs`
- `12 %`

---

# Recomendaciones de uso

- Registrar datasets con nombres claros y únicos
- Usar archivos CSV limpios y con encabezados correctos
- Evitar columnas duplicadas
- Evitar CSV demasiado pesados para no afectar el rendimiento
- Usar `fields` y `limit` para optimizar consultas

---

# Ejemplos completos

## Ejemplo 1: listar datasets

```txt
{{base_url}}?action=list_datasets&api_key=TU_API_KEY
```

## Ejemplo 2: describir columnas

```txt
{{base_url}}?action=describe_dataset&api_key=TU_API_KEY&dataset=ventas.csv
```

## Ejemplo 3: consulta simple

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv
```

## Ejemplo 4: consulta con filtros y ordenamiento

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&filter=Country:Colombia,Category:Snacks&fields=Product,Price,Country&order_by=Price desc&limit=10
```

## Ejemplo 5: agrupación

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&group_by=Category&agg=SUM(Price)
```

## Ejemplo 6: exportar CSV

```txt
{{base_url}}?action=query_dataset&api_key=TU_API_KEY&dataset=ventas.csv&fields=Product,Price&limit=20&export=csv
```

---

# Autor

Proyecto desarrollado con fines académicos y de práctica para explorar:

- diseño de APIs REST-like
- autenticación simple por API key
- integración con Google Apps Script
- consumo desde múltiples clientes

---

# Licencia

Puedes agregar aquí la licencia que prefieras, por ejemplo:

```txt
MIT
```

o bien indicar:

```txt
Uso educativo y de práctica
```
