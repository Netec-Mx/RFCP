---LAB_START---
LAB_ID: 08-00-01
---MARKDOWN---
# Práctica 2 — Suite de Pruebas API (Smoke/Regresión) Data-Driven

## Metadatos

| Campo            | Valor                                      |
|------------------|--------------------------------------------|
| **Duración**     | 144 minutos                                |
| **Complejidad**  | Media                                      |
| **Nivel Bloom**  | Aplicar (Apply)                            |
| **Etiquetas**    | `smoke`, `regression`, `api`, `data-driven`|
| **API objetivo** | ReqRes.in / WireMock (alternativa offline) |

---

## Visión General

En este laboratorio construirás una suite profesional de pruebas API REST en Robot Framework que cubre el ciclo completo de un recurso: autenticación, listado, creación, actualización parcial y eliminación. Aplicarás los conceptos de métodos HTTP, códigos de estado, headers y payloads estudiados en la Lección 8.1, materializándolos en keywords reutilizables, pruebas data-driven con archivos CSV y validaciones de contrato JSON. La suite quedará organizada en dos niveles de ejecución —**smoke** y **regresión**— seleccionables desde la línea de comandos mediante tags, tal como se hace en proyectos de automatización reales.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio serás capaz de:

- [ ] Construir keywords base de sesión HTTP y flujos de autenticación Bearer reutilizables con `RequestsLibrary`
- [ ] Organizar casos de prueba en niveles smoke y regresión usando tags de Robot Framework
- [ ] Implementar pruebas data-driven con `robotframework-datadriver` leyendo combinaciones desde un archivo CSV
- [ ] Validar respuestas JSON verificando código de estado, campos requeridos y tipos de datos con `JSONLibrary`
- [ ] Gestionar datos dinámicos entre requests (IDs creados, tokens) usando variables de suite y keywords de setup/teardown

---

## Prerrequisitos

### Conocimiento

- Laboratorio 07-00-01 completado **o** experiencia equivalente con la estructura de proyectos Robot Framework (archivos `.robot`, `*** Settings ***`, `*** Keywords ***`, `*** Variables ***`)
- Conceptos HTTP: métodos GET/POST/PUT/PATCH/DELETE, códigos de estado 2xx/4xx/5xx, headers `Content-Type` y `Authorization`
- Comprensión básica de JSON: objetos, arrays, tipos de datos primitivos
- Familiaridad conceptual con autenticación por token Bearer

### Acceso y Software

| Componente                       | Versión mínima | Verificación                          |
|----------------------------------|---------------|---------------------------------------|
| Python                           | 3.10          | `python --version`                    |
| Robot Framework                  | 7.0           | `robot --version`                     |
| RequestsLibrary                  | 0.9.4         | `pip show robotframework-requests`    |
| JSONLibrary                      | 0.5           | `pip show robotframework-jsonlibrary` |
| robotframework-datadriver        | 1.11          | `pip show robotframework-datadriver`  |
| Acceso a internet **o** WireMock | —             | Ver Paso 1                            |

---

## Entorno del Laboratorio

### Hardware Recomendado

| Recurso    | Mínimo             | Recomendado        |
|------------|--------------------|--------------------|
| RAM        | 8 GB               | 16 GB              |
| Disco      | 10 GB libres (SSD) | 10 GB libres (SSD) |
| Resolución | 1280×768           | 1920×1080          |
| Red        | Conexión a internet o WireMock local | |

### Estructura de Directorios del Laboratorio

```
lab_08_api/
├── tests/
│   ├── smoke/
│   │   └── api_smoke.robot
│   └── regression/
│       ├── api_regression.robot
│       └── api_datadriven.robot
├── resources/
│   ├── api_keywords.resource
│   └── auth_keywords.resource
├── data/
│   └── login_scenarios.csv
├── config/
│   └── variables.robot
├── wiremock/
│   └── mappings/
│       ├── login_success.json
│       ├── login_failure.json
│       ├── users_list.json
│       ├── user_single.json
│       ├── user_create.json
│       ├── user_update.json
│       └── user_delete.json
└── results/
```

### Comandos de Configuración Inicial

```bash
# 1. Crear y activar entorno virtual
python -m venv venv

# Windows
venv\Scripts\activate

# Linux / macOS
source venv/bin/activate

# 2. Instalar dependencias
pip install \
  robotframework>=7.0 \
  robotframework-requests>=0.9.4 \
  robotframework-jsonlibrary>=0.5 \
  robotframework-datadriver>=1.11

# 3. Verificar instalaciones
pip show robotframework-requests robotframework-jsonlibrary robotframework-datadriver

# 4. Crear estructura de directorios
mkdir -p lab_08_api/tests/smoke
mkdir -p lab_08_api/tests/regression
mkdir -p lab_08_api/resources
mkdir -p lab_08_api/data
mkdir -p lab_08_api/config
mkdir -p lab_08_api/wiremock/mappings
mkdir -p lab_08_api/results
```

> **Entornos con proxy corporativo:** Configura las variables de entorno antes de ejecutar `pip`:
> ```bash
> export HTTP_PROXY=http://proxy.empresa.com:8080
> export HTTPS_PROXY=http://proxy.empresa.com:8080
> # O usa el flag directo:
> pip install --proxy http://proxy.empresa.com:8080 robotframework-requests
> ```

---

## Pasos del Laboratorio

---

### Paso 1 — Configurar la API Objetivo (ReqRes.in o WireMock)

**Objetivo:** Garantizar que existe un endpoint HTTP accesible antes de escribir cualquier prueba.

#### Opción A — ReqRes.in (Requiere Internet)

Verifica que el servicio está disponible ejecutando desde terminal:

```bash
curl -s -o /dev/null -w "%{http_code}" https://reqres.in/api/users/1
# Resultado esperado: 200
```

Si obtienes `200`, continúa al Paso 2 con `BASE_URL = https://reqres.in`.

#### Opción B — WireMock (Alternativa Offline)

**Instrucciones:**

1. Descarga WireMock standalone si no está disponible:
   ```bash
   curl -L -o wiremock.jar \
     https://repo1.maven.org/maven2/org/wiremock/wiremock-standalone/3.3.1/wiremock-standalone-3.3.1.jar
   ```

2. Crea los mappings JSON dentro de `lab_08_api/wiremock/mappings/`:

   **`login_success.json`**
   ```json
   {
     "request": {
       "method": "POST",
       "url": "/api/login",
       "bodyPatterns": [{"contains": "eve.holt@reqres.in"}]
     },
     "response": {
       "status": 200,
       "headers": {"Content-Type": "application/json"},
       "jsonBody": {"token": "QpwL5tpe83ilfN2"}
     }
   }
   ```

   **`login_failure.json`**
   ```json
   {
     "request": {
       "method": "POST",
       "url": "/api/login",
       "bodyPatterns": [{"contains": "invalid@test.com"}]
     },
     "response": {
       "status": 400,
       "headers": {"Content-Type": "application/json"},
       "jsonBody": {"error": "user not found"}
     }
   }
   ```

   **`users_list.json`**
   ```json
   {
     "request": {"method": "GET", "url": "/api/users?page=2"},
     "response": {
       "status": 200,
       "headers": {"Content-Type": "application/json"},
       "jsonBody": {
         "page": 2, "per_page": 6, "total": 12, "total_pages": 2,
         "data": [
           {"id": 7, "email": "michael.lawson@reqres.in",
            "first_name": "Michael", "last_name": "Lawson",
            "avatar": "https://reqres.in/img/faces/7-image.jpg"}
         ]
       }
     }
   }
   ```

   **`user_single.json`**
   ```json
   {
     "request": {"method": "GET", "url": "/api/users/2"},
     "response": {
       "status": 200,
       "headers": {"Content-Type": "application/json"},
       "jsonBody": {
         "data": {"id": 2, "email": "janet.weaver@reqres.in",
                  "first_name": "Janet", "last_name": "Weaver",
                  "avatar": "https://reqres.in/img/faces/2-image.jpg"},
         "support": {"url": "https://reqres.in/#support-heading",
                     "text": "To keep ReqRes free, contributions towards server costs are appreciated!"}
       }
     }
   }
   ```

   **`user_create.json`**
   ```json
   {
     "request": {"method": "POST", "url": "/api/users"},
     "response": {
       "status": 201,
       "headers": {"Content-Type": "application/json"},
       "jsonBody": {"name": "morpheus", "job": "leader",
                    "id": "999", "createdAt": "2024-01-15T10:00:00.000Z"}
     }
   }
   ```

   **`user_update.json`**
   ```json
   {
     "request": {"method": "PUT", "url": "/api/users/2"},
     "response": {
       "status": 200,
       "headers": {"Content-Type": "application/json"},
       "jsonBody": {"name": "morpheus", "job": "zion resident",
                    "updatedAt": "2024-01-15T10:05:00.000Z"}
     }
   }
   ```

   **`user_delete.json`**
   ```json
   {
     "request": {"method": "DELETE", "url": "/api/users/2"},
     "response": {"status": 204, "body": ""}
   }
   ```

3. Inicia WireMock:
   ```bash
   cd lab_08_api
   java -jar wiremock.jar --port 8080 \
     --root-dir wiremock --verbose
   ```

4. Verifica:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/api/users/2
   # Resultado esperado: 200
   ```

**Resultado esperado:** Terminal de WireMock muestra `Started WireMock on port 8080`.

**Verificación:**
- Opción A: `curl` retorna `200` a `https://reqres.in/api/users/1`
- Opción B: `curl` retorna `200` a `http://localhost:8080/api/users/2`

---

### Paso 2 — Crear el Archivo de Variables Globales

**Objetivo:** Centralizar la URL base, endpoints y constantes en un único archivo para facilitar el cambio entre ReqRes.in y WireMock sin modificar los tests.

#### Instrucciones

1. Crea el archivo `lab_08_api/config/variables.robot`:

```robot
*** Variables ***
# ─── URL Base ─────────────────────────────────────────────────────────────────
# Usar ReqRes.in (online):
${BASE_URL}             https://reqres.in
# Usar WireMock (offline) — descomenta la línea siguiente y comenta la anterior:
# ${BASE_URL}           http://localhost:8080

# ─── Endpoints ────────────────────────────────────────────────────────────────
${ENDPOINT_LOGIN}       /api/login
${ENDPOINT_USERS}       /api/users
${ENDPOINT_USER_2}      /api/users/2
${ENDPOINT_USERS_PAGE2}    /api/users?page=2

# ─── Credenciales de prueba (ReqRes.in) ───────────────────────────────────────
${VALID_EMAIL}          eve.holt@reqres.in
${VALID_PASSWORD}       cityslicka
${INVALID_EMAIL}        invalid@test.com
${INVALID_PASSWORD}     wrongpassword

# ─── Configuración HTTP ───────────────────────────────────────────────────────
${TIMEOUT}              10
${SESSION_ALIAS}        api_session

# ─── Variables de suite (se populan en runtime) ───────────────────────────────
${AUTH_TOKEN}           ${EMPTY}
${CREATED_USER_ID}      ${EMPTY}
```

**Resultado esperado:** Archivo creado sin errores de sintaxis.

**Verificación:**
```bash
python -m robot --dryrun config/variables.robot 2>&1 | head -5
# No debe mostrar errores de parsing
```

---

### Paso 3 — Implementar Keywords de Sesión y Autenticación

**Objetivo:** Crear keywords reutilizables que encapsulen la creación de sesión HTTP y el flujo de autenticación Bearer, siguiendo el principio de separación de capas establecido en el Capítulo 7.

#### Instrucciones

1. Crea `lab_08_api/resources/auth_keywords.resource`:

```robot
*** Settings ***
Library    RequestsLibrary
Library    Collections
Variables    ../config/variables.robot

*** Keywords ***
Crear Sesión HTTP
    [Documentation]    Crea una sesión HTTP reutilizable hacia la BASE_URL.
    ...                Debe llamarse en Suite Setup antes de cualquier request.
    Create Session
    ...    alias=${SESSION_ALIAS}
    ...    url=${BASE_URL}
    ...    verify=True
    ...    timeout=${TIMEOUT}
    Log    Sesión HTTP creada hacia: ${BASE_URL}

Cerrar Sesión HTTP
    [Documentation]    Elimina la sesión HTTP activa. Llamar en Suite Teardown.
    Delete All Sessions
    Log    Sesión HTTP cerrada.

Obtener Token De Autenticación
    [Documentation]    Realiza POST /api/login con credenciales válidas,
    ...                extrae el token y lo almacena en la variable de suite ${AUTH_TOKEN}.
    [Arguments]    ${email}=${VALID_EMAIL}    ${password}=${VALID_PASSWORD}
    ${payload}=    Create Dictionary
    ...    email=${email}
    ...    password=${password}
    ${headers}=    Create Dictionary
    ...    Content-Type=application/json
    ...    Accept=application/json
    ${response}=    POST On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_LOGIN}
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=200
    ${token}=    Set Variable    ${response.json()}[token]
    Set Suite Variable    ${AUTH_TOKEN}    ${token}
    Log    Token obtenido y almacenado en variable de suite: ${token}
    RETURN    ${token}

Construir Headers Autenticados
    [Documentation]    Retorna un diccionario de headers con Bearer token.
    ...                Requiere que ${AUTH_TOKEN} esté populado.
    [Arguments]    ${extra_headers}=${None}
    ${headers}=    Create Dictionary
    ...    Content-Type=application/json
    ...    Accept=application/json
    ...    Authorization=Bearer ${AUTH_TOKEN}
    IF    $extra_headers is not None
        FOR    ${key}    ${value}    IN    &{extra_headers}
            Set To Dictionary    ${headers}    ${key}=${value}
        END
    END
    RETURN    ${headers}

Verificar Código De Estado
    [Documentation]    Keyword auxiliar para validar el status code de una respuesta.
    [Arguments]    ${response}    ${expected_code}
    Should Be Equal As Numbers
    ...    ${response.status_code}
    ...    ${expected_code}
    ...    msg=Código de estado esperado: ${expected_code}, recibido: ${response.status_code}
```

2. Crea `lab_08_api/resources/api_keywords.resource`:

```robot
*** Settings ***
Library    RequestsLibrary
Library    JSONLibrary
Library    Collections
Resource   ./auth_keywords.resource
Variables  ../config/variables.robot

*** Keywords ***
GET Usuarios Paginados
    [Documentation]    Ejecuta GET /api/users?page=2 y retorna el objeto de respuesta completo.
    [Arguments]    ${page}=2
    ${headers}=    Construir Headers Autenticados
    ${response}=    GET On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}
    ...    params=page=${page}
    ...    headers=${headers}
    ...    expected_status=200
    RETURN    ${response}

GET Usuario Por ID
    [Documentation]    Ejecuta GET /api/users/{id} y retorna la respuesta.
    [Arguments]    ${user_id}=2    ${expected_status}=200
    ${headers}=    Construir Headers Autenticados
    ${response}=    GET On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}/${user_id}
    ...    headers=${headers}
    ...    expected_status=${expected_status}
    RETURN    ${response}

POST Crear Usuario
    [Documentation]    Crea un nuevo usuario. Almacena el ID retornado en ${CREATED_USER_ID}.
    [Arguments]    ${name}=morpheus    ${job}=leader
    ${payload}=    Create Dictionary    name=${name}    job=${job}
    ${headers}=    Construir Headers Autenticados
    ${response}=    POST On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=201
    ${user_id}=    Set Variable    ${response.json()}[id]
    Set Suite Variable    ${CREATED_USER_ID}    ${user_id}
    Log    Usuario creado con ID: ${user_id}
    RETURN    ${response}

PUT Actualizar Usuario Completo
    [Documentation]    Reemplaza completamente un usuario con PUT.
    [Arguments]    ${user_id}=2    ${name}=morpheus    ${job}=zion resident
    ${payload}=    Create Dictionary    name=${name}    job=${job}
    ${headers}=    Construir Headers Autenticados
    ${response}=    PUT On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}/${user_id}
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=200
    RETURN    ${response}

PATCH Actualizar Usuario Parcial
    [Documentation]    Actualiza parcialmente un usuario con PATCH.
    [Arguments]    ${user_id}=2    &{fields}
    ${headers}=    Construir Headers Autenticados
    ${response}=    PATCH On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}/${user_id}
    ...    json=${fields}
    ...    headers=${headers}
    ...    expected_status=200
    RETURN    ${response}

DELETE Eliminar Usuario
    [Documentation]    Elimina un usuario. Espera 204 No Content.
    [Arguments]    ${user_id}=2
    ${headers}=    Construir Headers Autenticados
    ${response}=    DELETE On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}/${user_id}
    ...    headers=${headers}
    ...    expected_status=204
    RETURN    ${response}

Validar Campos Requeridos En Respuesta
    [Documentation]    Verifica que todos los campos de la lista existen en el JSON de respuesta.
    [Arguments]    ${response_body}    @{required_fields}
    FOR    ${field}    IN    @{required_fields}
        Dictionary Should Contain Key
        ...    ${response_body}
        ...    ${field}
        ...    msg=Campo requerido ausente en la respuesta: '${field}'
    END

Validar Tipo De Campo
    [Documentation]    Verifica que un campo del diccionario es del tipo Python esperado.
    [Arguments]    ${response_body}    ${field}    ${expected_type}
    ${value}=    Get From Dictionary    ${response_body}    ${field}
    Should Be True
    ...    isinstance($value, ${expected_type})
    ...    msg=Campo '${field}' tiene tipo incorrecto. Esperado: ${expected_type}
```

**Resultado esperado:** Dos archivos `.resource` creados con sintaxis válida.

**Verificación:**
```bash
cd lab_08_api
python -m robot --dryrun resources/auth_keywords.resource
python -m robot --dryrun resources/api_keywords.resource
```
Ambos comandos deben terminar sin errores.

---

### Paso 4 — Construir la Suite Smoke

**Objetivo:** Implementar los 6 casos críticos del happy path etiquetados con `smoke` que validan el ciclo completo de vida de un recurso.

#### Instrucciones

1. Crea `lab_08_api/tests/smoke/api_smoke.robot`:

```robot
*** Settings ***
Documentation    Suite SMOKE — Happy path crítico de la API ReqRes.in
...              Cubre: autenticación, listado, obtención, creación, actualización y eliminación.
...              Ejecutar con: robot --include smoke tests/smoke/
Resource         ../../resources/auth_keywords.resource
Resource         ../../resources/api_keywords.resource
Variables        ../../config/variables.robot

Suite Setup      Inicializar Suite Smoke
Suite Teardown   Cerrar Sesión HTTP

*** Keywords ***
Inicializar Suite Smoke
    [Documentation]    Crea la sesión HTTP y obtiene el token de autenticación
    ...                antes de ejecutar cualquier test de la suite.
    Crear Sesión HTTP
    Obtener Token De Autenticación

*** Test Cases ***
SMOKE-01: Login exitoso retorna token válido
    [Documentation]    Verifica que POST /api/login con credenciales correctas
    ...                retorna HTTP 200 y un token no vacío.
    [Tags]    smoke    auth    happy-path
    ${payload}=    Create Dictionary
    ...    email=${VALID_EMAIL}
    ...    password=${VALID_PASSWORD}
    ${headers}=    Create Dictionary
    ...    Content-Type=application/json
    ...    Accept=application/json
    ${response}=    POST On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_LOGIN}
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=200
    # Validar estructura del cuerpo
    ${body}=    Set Variable    ${response.json()}
    Validar Campos Requeridos En Respuesta    ${body}    token
    Should Not Be Empty    ${body}[token]
    ...    msg=El token no debe estar vacío tras un login exitoso
    Log    Token recibido: ${body}[token]

SMOKE-02: Listar usuarios retorna página con datos
    [Documentation]    Verifica que GET /api/users?page=2 retorna HTTP 200
    ...                y una lista de usuarios con campos requeridos.
    [Tags]    smoke    users    happy-path
    ${response}=    GET Usuarios Paginados    page=2
    ${body}=    Set Variable    ${response.json()}
    # Validar estructura de paginación
    Validar Campos Requeridos En Respuesta    ${body}    page    per_page    total    data
    # Validar que 'data' es una lista no vacía
    ${data}=    Get From Dictionary    ${body}    data
    Should Not Be Empty    ${data}    msg=La lista de usuarios no debe estar vacía
    # Validar que el primer usuario tiene los campos esperados
    ${first_user}=    Set Variable    ${data}[0]
    Validar Campos Requeridos En Respuesta    ${first_user}    id    email    first_name    last_name
    Log    Total de usuarios en API: ${body}[total]

SMOKE-03: Obtener usuario por ID retorna datos correctos
    [Documentation]    Verifica que GET /api/users/2 retorna HTTP 200
    ...                y los datos del usuario con ID 2.
    [Tags]    smoke    users    happy-path
    ${response}=    GET Usuario Por ID    user_id=2
    ${body}=    Set Variable    ${response.json()}
    Validar Campos Requeridos En Respuesta    ${body}    data    support
    ${user_data}=    Get From Dictionary    ${body}    data
    Validar Campos Requeridos En Respuesta    ${user_data}    id    email    first_name    last_name
    Should Be Equal As Numbers    ${user_data}[id]    2
    Log    Usuario obtenido: ${user_data}[first_name] ${user_data}[last_name]

SMOKE-04: Crear usuario retorna 201 con ID asignado
    [Documentation]    Verifica que POST /api/users crea un recurso,
    ...                retorna HTTP 201 y el cuerpo incluye el ID generado.
    [Tags]    smoke    users    create    happy-path
    ${response}=    POST Crear Usuario    name=morpheus    job=leader
    ${body}=    Set Variable    ${response.json()}
    Validar Campos Requeridos En Respuesta    ${body}    name    job    id    createdAt
    Should Not Be Empty    ${body}[id]    msg=El ID del usuario creado no debe estar vacío
    Should Be Equal    ${body}[name]    morpheus
    Should Be Equal    ${body}[job]     leader
    Log    Usuario creado con ID: ${body}[id]

SMOKE-05: Actualizar usuario completo retorna 200 con updatedAt
    [Documentation]    Verifica que PUT /api/users/2 actualiza el recurso
    ...                y retorna HTTP 200 con el timestamp de actualización.
    [Tags]    smoke    users    update    happy-path
    ${response}=    PUT Actualizar Usuario Completo
    ...    user_id=2    name=morpheus    job=zion resident
    ${body}=    Set Variable    ${response.json()}
    Validar Campos Requeridos En Respuesta    ${body}    name    job    updatedAt
    Should Be Equal    ${body}[name]    morpheus
    Should Be Equal    ${body}[job]     zion resident
    Should Not Be Empty    ${body}[updatedAt]

SMOKE-06: Eliminar usuario retorna 204 sin cuerpo
    [Documentation]    Verifica que DELETE /api/users/2 retorna HTTP 204
    ...                y el cuerpo de respuesta está vacío.
    [Tags]    smoke    users    delete    happy-path
    ${response}=    DELETE Eliminar Usuario    user_id=2
    Should Be Equal As Numbers    ${response.status_code}    204
    ${body_text}=    Set Variable    ${response.text}
    Should Be Empty    ${body_text}
    ...    msg=DELETE exitoso no debe retornar cuerpo (204 No Content)
```

**Resultado esperado:** Archivo `.robot` creado con 6 test cases.

**Verificación (dry-run):**
```bash
cd lab_08_api
robot --dryrun tests/smoke/api_smoke.robot
```
Debe mostrar `0 tests failed` en modo dry-run.

---

### Paso 5 — Construir la Suite de Regresión

**Objetivo:** Implementar 10+ casos de prueba que cubran escenarios negativos, límites y errores esperados, organizados con el tag `regression`.

#### Instrucciones

1. Crea `lab_08_api/tests/regression/api_regression.robot`:

```robot
*** Settings ***
Documentation    Suite REGRESIÓN — Escenarios negativos, límites y errores esperados.
...              Ejecutar con: robot --include regression tests/regression/
Resource         ../../resources/auth_keywords.resource
Resource         ../../resources/api_keywords.resource
Variables        ../../config/variables.robot

Suite Setup      Inicializar Suite Regresión
Suite Teardown   Cerrar Sesión HTTP

*** Keywords ***
Inicializar Suite Regresión
    Crear Sesión HTTP
    Obtener Token De Autenticación

*** Test Cases ***
REG-01: Login con contraseña incorrecta retorna 400
    [Documentation]    Verifica que POST /api/login con password inválido retorna 400
    ...                y un mensaje de error descriptivo.
    [Tags]    regression    auth    negative
    ${payload}=    Create Dictionary
    ...    email=peter@klaven.com
    ...    password=cityslicka
    ${headers}=    Create Dictionary
    ...    Content-Type=application/json
    ...    Accept=application/json
    ${response}=    POST On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_LOGIN}
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=400
    ${body}=    Set Variable    ${response.json()}
    Validar Campos Requeridos En Respuesta    ${body}    error
    Should Not Be Empty    ${body}[error]

REG-02: Login sin campo password retorna 400
    [Documentation]    Verifica que un payload incompleto (sin 'password') retorna 400.
    [Tags]    regression    auth    negative    boundary
    ${payload}=    Create Dictionary    email=eve.holt@reqres.in
    ${headers}=    Create Dictionary    Content-Type=application/json
    ${response}=    POST On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_LOGIN}
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=400
    ${body}=    Set Variable    ${response.json()}
    Should Contain    ${body}[error]    Missing password

REG-03: Obtener usuario inexistente retorna 404
    [Documentation]    Verifica que GET /api/users/9999 retorna 404
    ...                cuando el recurso no existe.
    [Tags]    regression    users    negative    not-found
    ${headers}=    Construir Headers Autenticados
    ${response}=    GET On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}/9999
    ...    headers=${headers}
    ...    expected_status=404
    Should Be Equal As Numbers    ${response.status_code}    404
    # ReqRes retorna cuerpo vacío {} en 404
    ${body}=    Set Variable    ${response.json()}
    Log    Cuerpo de respuesta 404: ${body}

REG-04: Listar usuarios página inexistente retorna lista vacía
    [Documentation]    Verifica que una página fuera de rango retorna 200
    ...                pero con lista 'data' vacía.
    [Tags]    regression    users    boundary
    ${response}=    GET Usuarios Paginados    page=999
    ${body}=    Set Variable    ${response.json()}
    ${data}=    Get From Dictionary    ${body}    data
    Should Be Empty    ${data}
    ...    msg=Una página inexistente debe retornar lista de datos vacía

REG-05: Crear usuario sin campo 'name' — validar respuesta parcial
    [Documentation]    Verifica que POST /api/users con payload mínimo (solo job)
    ...                retorna 201 (ReqRes acepta payloads parciales).
    [Tags]    regression    users    create    boundary
    ${payload}=    Create Dictionary    job=tester
    ${headers}=    Construir Headers Autenticados
    ${response}=    POST On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=201
    ${body}=    Set Variable    ${response.json()}
    Validar Campos Requeridos En Respuesta    ${body}    id    createdAt
    Log    ID generado para usuario sin nombre: ${body}[id]

REG-06: PUT con payload vacío retorna 200 con updatedAt
    [Documentation]    Verifica que PUT /api/users/2 con payload vacío {}
    ...                retorna 200 y el campo updatedAt (comportamiento de ReqRes).
    [Tags]    regression    users    update    boundary
    ${payload}=    Create Dictionary
    ${headers}=    Construir Headers Autenticados
    ${response}=    PUT On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_USERS}/2
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=200
    ${body}=    Set Variable    ${response.json()}
    Validar Campos Requeridos En Respuesta    ${body}    updatedAt

REG-07: PATCH actualización parcial solo modifica campos enviados
    [Documentation]    Verifica que PATCH /api/users/2 con solo 'job'
    ...                retorna 200 y el campo updatedAt.
    [Tags]    regression    users    patch    partial-update
    ${response}=    PATCH Actualizar Usuario Parcial    user_id=2    job=automation tester
    ${body}=    Set Variable    ${response.json()}
    Validar Campos Requeridos En Respuesta    ${body}    job    updatedAt
    Should Be Equal    ${body}[job]    automation tester

REG-08: DELETE usuario ya eliminado retorna 204
    [Documentation]    Verifica que DELETE es idempotente en ReqRes:
    ...                siempre retorna 204 independientemente del ID.
    [Tags]    regression    users    delete    idempotent
    ${response}=    DELETE Eliminar Usuario    user_id=2
    Should Be Equal As Numbers    ${response.status_code}    204

REG-09: Validar tipos de datos en respuesta de usuario
    [Documentation]    Verifica que los campos del usuario tienen los tipos Python correctos:
    ...                id es int, email/first_name/last_name son str.
    [Tags]    regression    users    contract    data-types
    ${response}=    GET Usuario Por ID    user_id=2
    ${body}=    Set Variable    ${response.json()}
    ${user_data}=    Get From Dictionary    ${body}    data
    Validar Tipo De Campo    ${user_data}    id            int
    Validar Tipo De Campo    ${user_data}    email         str
    Validar Tipo De Campo    ${user_data}    first_name    str
    Validar Tipo De Campo    ${user_data}    last_name     str
    Log    Validación de tipos completada para usuario ID 2

REG-10: Validar contrato de paginación en listado de usuarios
    [Documentation]    Verifica que la respuesta de listado cumple el contrato:
    ...                page (int), per_page (int), total (int), data (list).
    [Tags]    regression    users    contract    pagination
    ${response}=    GET Usuarios Paginados    page=1
    ${body}=    Set Variable    ${response.json()}
    Validar Tipo De Campo    ${body}    page         int
    Validar Tipo De Campo    ${body}    per_page     int
    Validar Tipo De Campo    ${body}    total        int
    Validar Tipo De Campo    ${body}    total_pages  int
    Validar Tipo De Campo    ${body}    data         list
    # Verificar consistencia: total_pages = ceil(total / per_page)
    ${total}=        Get From Dictionary    ${body}    total
    ${per_page}=     Get From Dictionary    ${body}    per_page
    ${total_pages}=  Get From Dictionary    ${body}    total_pages
    ${expected_pages}=    Evaluate    math.ceil($total / $per_page)    modules=math
    Should Be Equal As Numbers    ${total_pages}    ${expected_pages}

REG-11: Flujo completo — Crear, Actualizar y Eliminar recurso dinámico
    [Documentation]    Prueba de integración que captura el ID creado en POST
    ...                y lo reutiliza en PUT y DELETE del mismo recurso.
    [Tags]    regression    users    integration    dynamic-data
    # PASO 1: Crear usuario y capturar ID dinámico
    ${create_response}=    POST Crear Usuario    name=test_user    job=qa_engineer
    ${body_create}=    Set Variable    ${create_response.json()}
    ${dynamic_id}=    Set Variable    ${body_create}[id]
    Should Not Be Empty    ${dynamic_id}
    Log    ID dinámico capturado: ${dynamic_id}
    # PASO 2: Actualizar el usuario recién creado (PUT)
    # Nota: ReqRes no persiste datos; PUT sobre el ID dinámico devuelve 200 igualmente
    ${update_response}=    PUT Actualizar Usuario Completo
    ...    user_id=${dynamic_id}    name=test_user    job=senior_qa
    Should Be Equal As Numbers    ${update_response.status_code}    200
    # PASO 3: Eliminar el recurso usando el ID capturado
    ${delete_response}=    DELETE Eliminar Usuario    user_id=${dynamic_id}
    Should Be Equal As Numbers    ${delete_response.status_code}    204
    Log    Flujo completo ejecutado para ID dinámico: ${dynamic_id}
```

**Resultado esperado:** Archivo con 11 test cases de regresión.

**Verificación:**
```bash
robot --dryrun tests/regression/api_regression.robot
```

---

### Paso 6 — Implementar Pruebas Data-Driven con CSV

**Objetivo:** Crear un único test case parametrizado que ejecute múltiples escenarios de login desde un archivo CSV usando `robotframework-datadriver`.

#### Instrucciones

1. Crea el archivo de datos `lab_08_api/data/login_scenarios.csv`:

```csv
*** Test Cases ***,email,password,expected_status,expected_field,expected_value,scenario_description
,eve.holt@reqres.in,cityslicka,200,token,,Login exitoso con credenciales correctas
,peter@klaven.com,cityslicka,400,error,Missing password,Login sin password correcto
,eve.holt@reqres.in,,400,error,Missing password,Login sin password (campo vacío)
,,cityslicka,400,error,,Login sin email
,invalid@test.com,wrongpassword,400,error,,Login con email inexistente
,eve.holt@reqres.in,wrongpassword,400,error,,Login con password incorrecto
,not-an-email,cityslicka,400,error,,Login con email malformado
,eve.holt@reqres.in,cityslicka,200,token,,Login exitoso repetido (idempotencia)
```

> **Nota sobre el CSV:** La primera fila es el encabezado requerido por DataDriver. La columna `*** Test Cases ***` debe estar vacía en las filas de datos (DataDriver genera el nombre del test automáticamente desde `scenario_description` si se configura).

2. Crea `lab_08_api/tests/regression/api_datadriven.robot`:

```robot
*** Settings ***
Documentation    Suite DATA-DRIVEN — Escenarios de login parametrizados desde CSV.
...              Usa robotframework-datadriver para ejecutar un único test case
...              con múltiples combinaciones de datos.
...              Ejecutar con: robot --include data-driven tests/regression/
Library          RequestsLibrary
Library          Collections
Library          DataDriver    ../../../data/login_scenarios.csv    encoding=utf-8
Resource         ../../resources/auth_keywords.resource
Variables        ../../config/variables.robot

Suite Setup      Crear Sesión HTTP
Suite Teardown   Cerrar Sesión HTTP

Test Template    Ejecutar Escenario De Login

*** Test Cases ***
Escenario de login: ${scenario_description}
    [Tags]    data-driven    regression    auth

*** Keywords ***
Ejecutar Escenario De Login
    [Documentation]    Template que ejecuta un escenario de login con los datos del CSV.
    [Arguments]
    ...    ${email}
    ...    ${password}
    ...    ${expected_status}
    ...    ${expected_field}
    ...    ${expected_value}
    ...    ${scenario_description}
    # Construir payload dinámicamente (omitir campos vacíos)
    ${payload}=    Create Dictionary
    IF    '${email}' != '${EMPTY}'
        Set To Dictionary    ${payload}    email=${email}
    END
    IF    '${password}' != '${EMPTY}'
        Set To Dictionary    ${payload}    password=${password}
    END
    ${headers}=    Create Dictionary
    ...    Content-Type=application/json
    ...    Accept=application/json
    # Ejecutar request sin expected_status para capturar cualquier código
    ${response}=    POST On Session
    ...    ${SESSION_ALIAS}
    ...    ${ENDPOINT_LOGIN}
    ...    json=${payload}
    ...    headers=${headers}
    ...    expected_status=any
    # Validar código de estado
    Should Be Equal As Numbers
    ...    ${response.status_code}
    ...    ${expected_status}
    ...    msg=[${scenario_description}] Código esperado: ${expected_status}, recibido: ${response.status_code}
    # Validar campo esperado en respuesta (si está definido)
    IF    '${expected_field}' != '${EMPTY}'
        ${body}=    Set Variable    ${response.json()}
        Dictionary Should Contain Key    ${body}    ${expected_field}
        ...    msg=[${scenario_description}] Campo '${expected_field}' no encontrado en respuesta
        # Validar valor esperado (si está definido)
        IF    '${expected_value}' != '${EMPTY}'
            Should Be Equal
            ...    ${body}[${expected_field}]
            ...    ${expected_value}
            ...    msg=[${scenario_description}] Valor de '${expected_field}' incorrecto
        END
    END
    Log    [${scenario_description}] → HTTP ${response.status_code} ✓
```

**Resultado esperado:** Archivo data-driven creado. DataDriver generará 8 test cases individuales en tiempo de ejecución.

**Verificación:**
```bash
robot --dryrun tests/regression/api_datadriven.robot
# Debe mostrar 8 test cases generados por DataDriver
```

---

### Paso 7 — Ejecutar las Suites y Analizar Reportes

**Objetivo:** Ejecutar las tres suites por separado y de forma combinada usando tags, verificando que los reportes HTML contienen la información esperada.

#### Instrucciones

1. **Ejecutar solo suite smoke:**
   ```bash
   cd lab_08_api
   robot \
     --include smoke \
     --outputdir results/smoke \
     --output output.xml \
     --log log.html \
     --report report.html \
     tests/smoke/
   ```

2. **Ejecutar solo suite de regresión:**
   ```bash
   robot \
     --include regression \
     --outputdir results/regression \
     --output output.xml \
     --log log.html \
     --report report.html \
     tests/regression/
   ```

3. **Ejecutar suite completa (smoke + regresión):**
   ```bash
   robot \
     --outputdir results/full \
     --output output.xml \
     --log log.html \
     --report report.html \
     tests/
   ```

4. **Ejecutar con variables externas (cambiar a WireMock sin editar archivos):**
   ```bash
   robot \
     --variable BASE_URL:http://localhost:8080 \
     --include smoke \
     --outputdir results/wiremock \
     tests/smoke/
   ```

5. Crea scripts de ejecución multiplataforma:

   **`run_smoke.sh`** (Linux/macOS):
   ```bash
   #!/usr/bin/env bash
   set -e
   RESULTS_DIR="results/smoke_$(date +%Y%m%d_%H%M%S)"
   mkdir -p "$RESULTS_DIR"
   robot \
     --include smoke \
     --outputdir "$RESULTS_DIR" \
     --output output.xml \
     --log log.html \
     --report report.html \
     tests/smoke/
   echo "Reporte disponible en: $RESULTS_DIR/report.html"
   ```

   **`run_smoke.bat`** (Windows):
   ```batch
   @echo off
   SET TIMESTAMP=%DATE:~-4%%DATE:~3,2%%DATE:~0,2%_%TIME:~0,2%%TIME:~3,2%%TIME:~6,2%
   SET RESULTS_DIR=results\smoke_%TIMESTAMP: =0%
   mkdir %RESULTS_DIR%
   robot ^
     --include smoke ^
     --outputdir %RESULTS_DIR% ^
     --output output.xml ^
     --log log.html ^
     --report report.html ^
     tests\smoke\
   echo Reporte disponible en: %RESULTS_DIR%\report.html
   ```

**Resultado esperado:**
- Suite smoke: 6 tests, 6 passed (0 failed)
- Suite regresión: 11 tests, 11 passed (0 failed)
- Suite data-driven: 8 tests, 8 passed (0 failed)

**Verificación:**
Abre `results/full/report.html` en el navegador y confirma:
- Sección **Statistics by Tag** muestra conteos para `smoke`, `regression`, `data-driven`
- Todos los tests aparecen en verde
- El log detalla los valores de token y IDs capturados en cada paso

---

## Validación y Pruebas

### Lista de Verificación Final

Ejecuta los siguientes comandos y confirma que cada uno retorna el resultado esperado:

```bash
cd lab_08_api

# 1. Verificar que existen exactamente 6 tests smoke
robot --dryrun --include smoke tests/ 2>&1 | grep "tests, 0 tests"

# 2. Verificar que existen al menos 10 tests de regresión
robot --dryrun --include regression tests/ 2>&1 | grep "tests, 0 tests"

# 3. Verificar que DataDriver genera 8 variantes data-driven
robot --dryrun tests/regression/api_datadriven.robot 2>&1 | grep "8 tests"

# 4. Ejecutar suite smoke completa y verificar 0 fallos
robot --include smoke --outputdir results/validation tests/smoke/
grep -c "PASS" results/validation/output.xml

# 5. Verificar que el output.xml contiene tags correctos
grep -o 'tag>[^<]*</tag' results/validation/output.xml | sort | uniq
```

### Validación de Contrato JSON Manual

Para confirmar que las keywords de contrato funcionan, ejecuta este test de validación aislado:

```bash
robot \
  --test "REG-09: Validar tipos de datos en respuesta de usuario" \
  --test "REG-10: Validar contrato de paginación en listado de usuarios" \
  --outputdir results/contract \
  tests/regression/api_regression.robot
```

Ambos tests deben pasar con `PASS`.

### Criterios de Aceptación

| Criterio                                                    | Resultado esperado       |
|-------------------------------------------------------------|--------------------------|
| Suite smoke ejecuta 6 tests en menos de 30 segundos        | ✅ 6 PASS                |
| Suite regresión ejecuta 11 tests                           | ✅ 11 PASS               |
| DataDriver genera y ejecuta 8 variantes del test de login  | ✅ 8 PASS                |
| Token Bearer aparece en headers de requests autenticados   | ✅ Visible en log.html   |
| ID dinámico capturado en POST se usa en PUT y DELETE       | ✅ REG-11 PASS           |
| Tags `smoke`/`regression` filtran correctamente con `--include` | ✅ Conteos correctos |

---

## Solución de Problemas

### Problema 1: `ConnectionError` o `SSLError` al conectar con ReqRes.in

**Síntoma:**
```
RequestsLibrary.RequestsError: HTTPSConnectionPool(host='reqres.in', port=443):
Max retries exceeded with url: /api/login
```
O bien:
```
ssl.SSLCertVerificationError: certificate verify failed
```

**Causa:**
El entorno está detrás de un proxy corporativo que intercepta el tráfico HTTPS, o las variables de entorno de proxy no están configuradas para el proceso Python que ejecuta Robot Framework.

**Solución:**

Opción A — Configurar proxy vía variables de entorno:
```bash
# Linux/macOS
export HTTP_PROXY=http://proxy.empresa.com:8080
export HTTPS_PROXY=http://proxy.empresa.com:8080
export NO_PROXY=localhost,127.0.0.1
robot --include smoke tests/smoke/

# Windows (PowerShell)
$env:HTTP_PROXY = "http://proxy.empresa.com:8080"
$env:HTTPS_PROXY = "http://proxy.empresa.com:8080"
robot --include smoke tests/smoke/
```

Opción B — Deshabilitar verificación SSL temporalmente en `auth_keywords.resource` (solo para entornos de laboratorio):
```robot
Create Session
...    alias=${SESSION_ALIAS}
...    url=${BASE_URL}
...    verify=False    # ⚠️ Solo para desarrollo/laboratorio
...    timeout=${TIMEOUT}
```

Opción C — Cambiar a WireMock local (sin dependencia de red):
```bash
robot --variable BASE_URL:http://localhost:8080 --include smoke tests/smoke/
```

---

### Problema 2: `DataDriverError` — DataDriver no genera los 8 test cases esperados

**Síntoma:**
```
DataDriver: No test cases found in CSV file.
```
O el test solo aparece una vez (sin variantes) en el dry-run:
```
1 test, 1 test run, 0 tests failed.
```

**Causa:**
El archivo CSV tiene problemas de formato: (a) la primera columna no es exactamente `*** Test Cases ***`, (b) hay caracteres BOM al inicio del archivo si se guardó con codificación UTF-8-BOM en Windows, o (c) el nombre del test case en el archivo `.robot` no usa la sintaxis de variable `${scenario_description}` que coincide con la columna del CSV.

**Solución:**

1. Verificar que el CSV no tiene BOM:
   ```bash
   # Linux/macOS — detectar BOM
   file data/login_scenarios.csv
   # Si dice "with BOM", recrear sin BOM:
   python -c "
   with open('data/login_scenarios.csv', 'r', encoding='utf-8-sig') as f:
       content = f.read()
   with open('data/login_scenarios.csv', 'w', encoding='utf-8') as f:
       f.write(content)
   print('BOM eliminado.')
   "
   ```

2. Verificar que la primera fila del CSV es exactamente:
   ```
   *** Test Cases ***,email,password,expected_status,expected_field,expected_value,scenario_description
   ```
   (sin espacios extra, sin comillas alrededor de `*** Test Cases ***`)

3. Verificar que el nombre del test case en el `.robot` usa exactamente la misma variable que la columna del CSV:
   ```robot
   # ✅ Correcto — coincide con columna 'scenario_description'
   Escenario de login: ${scenario_description}

   # ❌ Incorrecto — nombre de variable diferente
   Escenario de login: ${descripcion}
   ```

4. Probar con ruta absoluta al CSV para descartar problemas de ruta relativa:
   ```robot
   Library    DataDriver    /ruta/absoluta/al/lab_08_api/data/login_scenarios.csv
   ```

---

## Limpieza

Una vez completado el laboratorio, ejecuta los siguientes pasos para dejar el entorno en estado limpio:

```bash
# 1. Detener WireMock si está corriendo (Ctrl+C en su terminal o:)
pkill -f wiremock  # Linux/macOS
# En Windows: cierra la ventana de terminal donde corre WireMock

# 2. Archivar resultados importantes antes de limpiar
cp -r lab_08_api/results lab_08_api/results_backup_$(date +%Y%m%d)

# 3. Limpiar directorio de resultados temporales
rm -rf lab_08_api/results/smoke
rm -rf lab_08_api/results/regression
rm -rf lab_08_api/results/validation
# Conservar results/full como evidencia del laboratorio completo

# 4. Desactivar entorno virtual
deactivate

# 5. (Opcional) Eliminar entorno virtual si no se usará en el siguiente laboratorio
# rm -rf venv
```

> **Conserva** el directorio `lab_08_api/` completo: la estructura de recursos, keywords y datos CSV será la base del Laboratorio 09 donde integrarás estas pruebas API con pruebas web en un pipeline unificado.

---

## Resumen

En este laboratorio construiste una suite profesional de pruebas API REST en Robot Framework aplicando los conceptos de métodos HTTP, códigos de estado, headers y payloads de la Lección 8.1. Los logros principales fueron:

| Componente implementado                        | Archivo                                  |
|------------------------------------------------|------------------------------------------|
| Variables globales y configuración de entorno  | `config/variables.robot`                 |
| Keywords de sesión HTTP y autenticación Bearer | `resources/auth_keywords.resource`       |
| Keywords reutilizables para cada método HTTP   | `resources/api_keywords.resource`        |
| Suite smoke (6 casos happy path)               | `tests/smoke/api_smoke.robot`            |
| Suite regresión (11 casos negativos/límites)   | `tests/regression/api_regression.robot`  |
| Suite data-driven con CSV (8 variantes)        | `tests/regression/api_datadriven.robot`  |
| Datos de prueba parametrizados                 | `data/login_scenarios.csv`               |
| Scripts de ejecución multiplataforma           | `run_smoke.sh` / `run_smoke.bat`         |

### Patrones Aplicados

- **Separación de capas:** las keywords de sesión/auth están en recursos separados de las keywords de negocio API
- **DRY (Don't Repeat Yourself):** un único template data-driven reemplaza 8 test cases duplicados
- **Datos dinámicos:** el ID creado en `POST` se captura con `Set Suite Variable` y se reutiliza en `PUT`/`DELETE`
- **Selección por tags:** `--include smoke` ejecuta solo el happy path crítico; `--include regression` ejecuta la batería completa

### Recursos Adicionales

- [Documentación oficial de RequestsLibrary](https://marketsquare.github.io/robotframework-requests/doc/RequestsLibrary.html)
- [robotframework-datadriver en PyPI](https://pypi.org/project/robotframework-datadriver/)
- [robotframework-jsonlibrary en GitHub](https://github.com/robotframework-thailand/robotframework-jsonlibrary)
- [ReqRes.in — API de práctica](https://reqres.in)
- [WireMock Standalone — servidor mock HTTP](https://wiremock.org/docs/standalone/java-jar/)
- [RFC 9110 — Semántica HTTP](https://www.rfc-editor.org/rfc/rfc9110)

---
LAB_END---
