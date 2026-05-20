---LAB_START---
LAB_ID: 07-00-01
---MARKDOWN---
# Práctica 1 — Flujo E2E Web (Login, Navegación, Validaciones)

## Metadatos

| Atributo         | Valor                                      |
|------------------|--------------------------------------------|
| **Duración**     | 216 minutos (3 bloques: 60 + 90 + 66 min) |
| **Complejidad**  | Alta                                       |
| **Nivel Bloom**  | Aplicar (Apply)                            |
| **Laboratorio**  | 07-00-01                                   |
| **Capítulo**     | 7 — Automatización Web con SeleniumLibrary |

---

## Descripción General

En este laboratorio construirás un flujo de automatización end-to-end completo sobre la aplicación web de práctica **The Internet** (https://the-internet.herokuapp.com). Partiendo desde la configuración del entorno y la gestión del driver de Chrome, implementarás pruebas que cubren login, navegación entre secciones, formularios, tablas dinámicas, alertas JavaScript e iframes. Aplicarás el patrón **Page Object adaptado a Robot Framework** mediante archivos `.resource` separados por página, waits explícitos en todos los puntos de sincronización y captura automática de screenshots en cada paso crítico. Al finalizar analizarás el reporte HTML generado por Robot Framework para verificar la trazabilidad completa de la ejecución.

> **Nota sobre disponibilidad:** Si `the-internet.herokuapp.com` no está disponible en tu entorno, consulta el [Apéndice A](#apéndice-a-alternativa-local-con-docker) al final de este laboratorio para ejecutar la aplicación localmente con Docker.

---

## Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Configurar una sesión de navegador Chrome (headless y visible) en Robot Framework usando SeleniumLibrary con gestión automática de drivers mediante `webdriver-manager`.
- [ ] Construir localizadores robustos mediante selectores CSS y expresiones XPath para identificar elementos de formularios, tablas y componentes dinámicos.
- [ ] Implementar estrategias de sincronización explícita (`Wait Until Element Is Visible`, `Wait Until Page Contains Element`) para garantizar estabilidad ante cargas asíncronas.
- [ ] Aplicar el patrón Page Object adaptado a Robot Framework mediante archivos `.resource` con keywords de alto nivel que encapsulan la lógica de cada página.
- [ ] Manejar alertas nativas del navegador, iframes embebidos y capturar screenshots automáticos integrados en el reporte HTML.

---

## Prerrequisitos

### Conocimientos

| Área                          | Nivel requerido                                                        |
|-------------------------------|------------------------------------------------------------------------|
| Robot Framework (sintaxis)    | Básico — Settings, Variables, Test Cases, Keywords                     |
| HTML / CSS                    | Básico — estructura del DOM, clases, IDs, atributos                    |
| XPath                         | Básico — expresiones simples como `//tag[@attr='value']`               |
| Python / pip                  | Básico — instalación de paquetes, entornos virtuales                   |
| Terminal / línea de comandos  | Básico — navegación de directorios, ejecución de comandos              |

### Acceso y Software

| Requisito                            | Verificación                                      |
|--------------------------------------|---------------------------------------------------|
| Python 3.10 o 3.11                   | `python --version`                                |
| Robot Framework 7.x                  | `robot --version`                                 |
| SeleniumLibrary 6.x                  | `pip show robotframework-seleniumlibrary`         |
| Google Chrome (versión ≥ 120)        | Menú Chrome → Ayuda → Acerca de Google Chrome     |
| VS Code + RF Language Server         | Extensión activa en VS Code                       |
| Conexión a internet                  | Acceso a `the-internet.herokuapp.com`             |
| Git 2.40+                            | `git --version`                                   |

---

## Entorno de Laboratorio

### Especificaciones de Hardware

| Componente    | Mínimo                              | Recomendado                  |
|---------------|-------------------------------------|------------------------------|
| Procesador    | Intel i5 8ª gen / AMD Ryzen 5       | Intel i7 / AMD Ryzen 7       |
| RAM           | 8 GB                                | 16 GB                        |
| Almacenamiento| 10 GB libres en SSD                 | 20 GB libres en SSD          |
| Pantalla      | 1280 × 768                          | 1920 × 1080                  |
| Red           | Conexión a internet                 | Banda ancha ≥ 10 Mbps        |

### Preparación del Entorno

Ejecuta los siguientes comandos **antes de iniciar el Bloque A**. Si ya tienes el entorno configurado de laboratorios anteriores, verifica las versiones y salta al paso de creación de la estructura del proyecto.

```bash
# 1. Crear y activar entorno virtual
python -m venv venv

# Linux / macOS
source venv/bin/activate

# Windows (PowerShell)
venv\Scripts\Activate.ps1

# Windows (CMD)
venv\Scripts\activate.bat

# 2. Actualizar pip
pip install --upgrade pip

# 3. Instalar dependencias del laboratorio
pip install robotframework>=7.0
pip install robotframework-seleniumlibrary>=6.2
pip install webdriver-manager>=4.0

# 4. Verificar instalaciones
robot --version
python -c "import SeleniumLibrary; print(SeleniumLibrary.__version__)"
python -c "from webdriver_manager.chrome import ChromeDriverManager; print('webdriver-manager OK')"
```

> **Entornos corporativos con proxy:** Si estás detrás de un proxy, usa:
> ```bash
> pip install --proxy http://usuario:password@proxy.empresa.com:8080 robotframework-seleniumlibrary
> ```
> O configura las variables de entorno `HTTP_PROXY` y `HTTPS_PROXY` antes de ejecutar pip.

---

## Instrucciones Paso a Paso

El laboratorio está dividido en **tres bloques** con checkpoints claros. Si llegas a un checkpoint con tiempo insuficiente, el instructor proporcionará el código parcial del bloque anterior para que puedas continuar.

---

## BLOQUE A (60 minutos): Configuración, Estructura y Login Básico

### Paso A1 — Crear la Estructura del Proyecto

**Objetivo:** Establecer la arquitectura de directorios del proyecto siguiendo las convenciones de Page Object para Robot Framework.

**Instrucciones:**

1. Crea el directorio raíz del proyecto y navega a él:

```bash
mkdir lab-07-e2e-web
cd lab-07-e2e-web
```

2. Crea la estructura de directorios completa:

```bash
# Linux / macOS
mkdir -p tests resources/pages resources/common results drivers

# Windows (PowerShell)
New-Item -ItemType Directory -Path tests, resources\pages, resources\common, results, drivers -Force
```

3. Verifica que la estructura es correcta:

```bash
# Linux / macOS / Git Bash
find . -type d | sort

# Windows (PowerShell)
Get-ChildItem -Recurse -Directory | Select-Object FullName
```

**Salida esperada:**

```
./drivers
./resources
./resources/common
./resources/pages
./results
./tests
```

**Verificación:** La estructura de directorios debe coincidir exactamente con la mostrada arriba. Los archivos `.resource` de páginas irán en `resources/pages/`, los recursos compartidos en `resources/common/`, y los archivos de suite de pruebas en `tests/`.

---

### Paso A2 — Configurar el Módulo de Driver con webdriver-manager

**Objetivo:** Crear un módulo Python que gestione la inicialización automática de ChromeDriver, siguiendo el patrón enseñado en la lección 7.1.

**Instrucciones:**

1. Crea el archivo `resources/common/driver_setup.py`:

```python
# resources/common/driver_setup.py
"""
Módulo de configuración de WebDriver.
Gestiona la descarga automática de ChromeDriver mediante webdriver-manager.
Compatible con entornos headless (CI/CD) y con interfaz gráfica.
"""

from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager


def get_chrome_options(headless: bool = False) -> Options:
    """Construye y retorna las opciones de Chrome configuradas."""
    opciones = Options()
    if headless:
        opciones.add_argument("--headless")
        opciones.add_argument("--no-sandbox")
        opciones.add_argument("--disable-dev-shm-usage")
        opciones.add_argument("--window-size=1920,1080")
    opciones.add_argument("--disable-extensions")
    opciones.add_argument("--disable-gpu")
    return opciones


def get_chrome_driver(headless: bool = False) -> webdriver.Chrome:
    """
    Inicializa y retorna un driver de Chrome con gestión automática de versiones.
    
    Args:
        headless: Si True, ejecuta Chrome sin interfaz gráfica.
    
    Returns:
        Instancia de webdriver.Chrome lista para usar.
    """
    servicio = Service(ChromeDriverManager().install())
    opciones = get_chrome_options(headless)
    return webdriver.Chrome(service=servicio, options=opciones)
```

2. Crea el archivo de variables globales `resources/common/variables.resource`:

```robotframework
*** Settings ***
Documentation    Variables globales compartidas por todos los tests del laboratorio.

*** Variables ***
# ── Configuración del navegador ──────────────────────────────────────────────
${NAVEGADOR}              chrome
${URL_BASE}               https://the-internet.herokuapp.com
${TIMEOUT_GLOBAL}         15s
${TIMEOUT_CORTO}          5s
${MODO_HEADLESS}          ${FALSE}

# ── Credenciales de prueba ───────────────────────────────────────────────────
${USUARIO_VALIDO}         tomsmith
${PASSWORD_VALIDO}        SuperSecretPassword!
${USUARIO_INVALIDO}       usuario_incorrecto
${PASSWORD_INVALIDO}      password_incorrecto

# ── Rutas de resultados ──────────────────────────────────────────────────────
${DIR_SCREENSHOTS}        ${CURDIR}${/}..${/}..${/}results${/}screenshots
```

**Salida esperada:** Los archivos se crean sin errores. Al abrir `variables.resource` en VS Code, el Language Server no debe mostrar errores de sintaxis.

**Verificación:**

```bash
python -c "from resources.common.driver_setup import get_chrome_driver; print('Módulo importado OK')"
```

---

### Paso A3 — Crear el Recurso de Configuración del Navegador

**Objetivo:** Implementar las keywords de apertura y cierre de sesión del navegador en un archivo `.resource` reutilizable, siguiendo el principio de centralización de la lección 7.1.

**Instrucciones:**

1. Crea el archivo `resources/common/browser.resource`:

```robotframework
*** Settings ***
Documentation    Keywords de gestión del ciclo de vida del navegador.
...              Centraliza la apertura, configuración y cierre de sesiones Chrome.
...              Soporta modo headless y visible mediante la variable ${MODO_HEADLESS}.
Library          SeleniumLibrary    timeout=${TIMEOUT_GLOBAL}    implicit_wait=0s
Resource         variables.resource

*** Keywords ***
Abrir Navegador Chrome
    [Documentation]    Abre Chrome en modo visible o headless según ${MODO_HEADLESS}.
    ...                Maximiza la ventana y navega a ${URL_BASE}.
    [Arguments]    ${url}=${URL_BASE}    ${headless}=${MODO_HEADLESS}
    IF    ${headless}
        ${opciones}=    Evaluate
        ...    selenium.webdriver.ChromeOptions()
        ...    modules=selenium.webdriver
        Call Method    ${opciones}    add_argument    --headless
        Call Method    ${opciones}    add_argument    --no-sandbox
        Call Method    ${opciones}    add_argument    --disable-dev-shm-usage
        Call Method    ${opciones}    add_argument    --window-size=1920,1080
        Create Webdriver    Chrome    options=${opciones}
        Go To    ${url}
    ELSE
        Open Browser    ${url}    ${NAVEGADOR}
        Maximize Browser Window
    END
    Set Selenium Timeout    ${TIMEOUT_GLOBAL}
    Set Selenium Implicit Wait    0s
    Log    Navegador abierto en: ${url}    level=INFO

Cerrar Navegador
    [Documentation]    Cierra la sesión de navegador activa.
    Close Browser

Cerrar Todos Los Navegadores
    [Documentation]    Cierra todas las sesiones de navegador abiertas.
    ...                Usar en Suite Teardown para garantizar limpieza total.
    Close All Browsers

Tomar Screenshot
    [Documentation]    Captura screenshot y lo guarda con nombre descriptivo.
    [Arguments]    ${nombre}=screenshot
    ${timestamp}=    Get Time    epoch
    ${archivo}=      Set Variable    ${nombre}_${timestamp}
    Capture Page Screenshot    ${DIR_SCREENSHOTS}${/}${archivo}.png
    Log    Screenshot guardado: ${archivo}.png    level=INFO

Tomar Screenshot En Fallo
    [Documentation]    Keyword de teardown para capturar evidencia en caso de fallo.
    Run Keyword If Test Failed    Tomar Screenshot    fallo_${TEST_NAME}
```

**Salida esperada:** El archivo se crea sin errores. La estructura de keywords sigue las convenciones de Robot Framework 7.x con bloques `IF/ELSE`.

**Verificación:** Verifica la sintaxis ejecutando:

```bash
python -m robot --dryrun --outputdir results/dryrun resources/common/browser.resource 2>&1 | head -20
```

---

### Paso A4 — Implementar el Page Object de Login

**Objetivo:** Crear el archivo `.resource` de la página de Login encapsulando todos los localizadores y keywords de interacción, aplicando el patrón Page Object.

**Instrucciones:**

1. Crea el archivo `resources/pages/LoginPage.resource`:

```robotframework
*** Settings ***
Documentation    Page Object para la página de Login de The Internet.
...              Encapsula localizadores y keywords de interacción con el formulario de login.
Library          SeleniumLibrary
Resource         ../common/browser.resource

*** Variables ***
# ── Localizadores de la página de Login ──────────────────────────────────────
${LOGIN_URL}              ${URL_BASE}/login
${INPUT_USUARIO}          id:username
${INPUT_PASSWORD}         id:password
${BTN_LOGIN}              css:button[type='submit']
${MSG_EXITO}              css:div.flash.success
${MSG_ERROR}              css:div.flash.error
${LINK_LOGOUT}            css:a[href='/logout']
${TITULO_PAGINA_LOGIN}    Login Page

*** Keywords ***
Navegar A Página De Login
    [Documentation]    Navega a la URL de login y verifica que la página cargó correctamente.
    Go To    ${LOGIN_URL}
    Wait Until Page Contains Element    ${INPUT_USUARIO}    timeout=${TIMEOUT_GLOBAL}
    Title Should Be    ${TITULO_PAGINA_LOGIN}
    Log    Página de login cargada correctamente    level=INFO

Ingresar Credenciales
    [Documentation]    Completa el formulario de login con usuario y contraseña.
    [Arguments]    ${usuario}    ${password}
    Wait Until Element Is Visible    ${INPUT_USUARIO}    timeout=${TIMEOUT_GLOBAL}
    Clear Element Text    ${INPUT_USUARIO}
    Input Text    ${INPUT_USUARIO}    ${usuario}
    Clear Element Text    ${INPUT_PASSWORD}
    Input Password    ${INPUT_PASSWORD}    ${password}
    Log    Credenciales ingresadas para usuario: ${usuario}    level=INFO

Hacer Clic En Botón Login
    [Documentation]    Hace clic en el botón de submit del formulario.
    Wait Until Element Is Visible    ${BTN_LOGIN}    timeout=${TIMEOUT_CORTO}
    Click Button    ${BTN_LOGIN}

Login Con Credenciales Válidas
    [Documentation]    Flujo completo de login exitoso.
    [Arguments]    ${usuario}=${USUARIO_VALIDO}    ${password}=${PASSWORD_VALIDO}
    Navegar A Página De Login
    Ingresar Credenciales    ${usuario}    ${password}
    Hacer Clic En Botón Login
    Tomar Screenshot    login_exitoso

Login Con Credenciales Inválidas
    [Documentation]    Flujo de login con credenciales incorrectas.
    [Arguments]    ${usuario}=${USUARIO_INVALIDO}    ${password}=${PASSWORD_INVALIDO}
    Navegar A Página De Login
    Ingresar Credenciales    ${usuario}    ${password}
    Hacer Clic En Botón Login
    Tomar Screenshot    login_fallido

Verificar Login Exitoso
    [Documentation]    Valida que el mensaje de éxito es visible y contiene el texto esperado.
    Wait Until Element Is Visible    ${MSG_EXITO}    timeout=${TIMEOUT_GLOBAL}
    Element Should Contain    ${MSG_EXITO}    You logged into a secure area!
    Log    Login exitoso verificado    level=INFO

Verificar Mensaje De Error Login
    [Documentation]    Valida que el mensaje de error es visible y contiene texto de error.
    [Arguments]    ${texto_error}=Your username is invalid!
    Wait Until Element Is Visible    ${MSG_ERROR}    timeout=${TIMEOUT_GLOBAL}
    Element Should Contain    ${MSG_ERROR}    ${texto_error}
    Log    Mensaje de error verificado: ${texto_error}    level=INFO

Hacer Logout
    [Documentation]    Ejecuta el flujo de cierre de sesión.
    Wait Until Element Is Visible    ${LINK_LOGOUT}    timeout=${TIMEOUT_GLOBAL}
    Click Link    ${LINK_LOGOUT}
    Wait Until Page Contains Element    ${INPUT_USUARIO}    timeout=${TIMEOUT_GLOBAL}
    Tomar Screenshot    logout_completado
    Log    Logout ejecutado correctamente    level=INFO
```

**Salida esperada:** El archivo `LoginPage.resource` queda creado con 8 keywords bien documentadas.

**Verificación:**

```bash
python -m robot --dryrun --outputdir results/dryrun resources/pages/LoginPage.resource 2>&1 | grep -E "(ERROR|WARN|OK)"
```

---

### Paso A5 — Crear y Ejecutar los Tests de Login

**Objetivo:** Escribir los casos de prueba de login y ejecutarlos para validar el flujo básico.

**Instrucciones:**

1. Crea el directorio de resultados de screenshots:

```bash
# Linux / macOS
mkdir -p results/screenshots

# Windows (PowerShell)
New-Item -ItemType Directory -Path results\screenshots -Force
```

2. Crea el archivo `tests/01_login_tests.robot`:

```robotframework
*** Settings ***
Documentation     Suite de pruebas para el flujo de Login de The Internet.
...               Cubre: login exitoso, login fallido, logout y validación de mensajes.
Resource          ../resources/common/browser.resource
Resource          ../resources/pages/LoginPage.resource

Suite Setup       Abrir Navegador Chrome
Suite Teardown    Cerrar Todos Los Navegadores
Test Teardown     Tomar Screenshot En Fallo

*** Test Cases ***
TC-LOGIN-01: Login Exitoso Con Credenciales Válidas
    [Documentation]    Verifica que un usuario válido puede autenticarse correctamente.
    [Tags]    login    smoke    critico
    Login Con Credenciales Válidas    ${USUARIO_VALIDO}    ${PASSWORD_VALIDO}
    Verificar Login Exitoso
    Tomar Screenshot    tc_login_01_exito

TC-LOGIN-02: Login Fallido Con Usuario Inválido
    [Documentation]    Verifica que credenciales incorrectas muestran mensaje de error.
    [Tags]    login    negativo
    Login Con Credenciales Inválidas    ${USUARIO_INVALIDO}    ${PASSWORD_INVALIDO}
    Verificar Mensaje De Error Login    Your username is invalid!
    Tomar Screenshot    tc_login_02_error_usuario

TC-LOGIN-03: Login Fallido Con Password Inválido
    [Documentation]    Verifica el mensaje de error cuando el password es incorrecto.
    [Tags]    login    negativo
    Navegar A Página De Login
    Ingresar Credenciales    ${USUARIO_VALIDO}    password_incorrecto
    Hacer Clic En Botón Login
    Verificar Mensaje De Error Login    Your password is invalid!
    Tomar Screenshot    tc_login_03_error_password

TC-LOGIN-04: Logout Exitoso Después De Login
    [Documentation]    Verifica que el usuario puede cerrar sesión correctamente.
    [Tags]    login    logout    smoke
    Login Con Credenciales Válidas
    Verificar Login Exitoso
    Hacer Logout
    Page Should Contain Element    ${INPUT_USUARIO}
    Tomar Screenshot    tc_login_04_logout
```

3. Ejecuta la suite de login:

```bash
robot --outputdir results \
      --log results/log_login.html \
      --report results/report_login.html \
      --variable MODO_HEADLESS:${FALSE} \
      tests/01_login_tests.robot
```

> **Windows (PowerShell):**
> ```powershell
> robot --outputdir results `
>       --log results/log_login.html `
>       --report results/report_login.html `
>       tests/01_login_tests.robot
> ```

**Salida esperada:**

```
==============================================================================
01 Login Tests :: Suite de pruebas para el flujo de Login de The Internet.
==============================================================================
TC-LOGIN-01: Login Exitoso Con Credenciales Válidas              | PASS |
TC-LOGIN-02: Login Fallido Con Usuario Inválido                  | PASS |
TC-LOGIN-03: Login Fallido Con Password Inválido                 | PASS |
TC-LOGIN-04: Logout Exitoso Después De Login                     | PASS |
==============================================================================
01 Login Tests                                                   | PASS |
4 tests, 4 passed, 0 failed
==============================================================================
```

**Verificación:** Abre `results/report_login.html` en el navegador. Debes ver los 4 tests en verde con screenshots adjuntos en el log.

> **✅ CHECKPOINT BLOQUE A:** Si los 4 tests de login pasan, puedes continuar al Bloque B. Si algún test falla, consulta la sección de [Resolución de Problemas](#resolución-de-problemas).

---

## BLOQUE B (90 minutos): Formularios, Tablas y Page Object Completo

### Paso B1 — Implementar el Page Object del Dashboard

**Objetivo:** Crear el recurso de la página principal que permite navegar entre secciones del sitio.

**Instrucciones:**

1. Crea el archivo `resources/pages/DashboardPage.resource`:

```robotframework
*** Settings ***
Documentation    Page Object para la página principal (Dashboard) de The Internet.
...              Gestiona la navegación entre las distintas secciones de práctica.
Library          SeleniumLibrary
Resource         ../common/browser.resource

*** Variables ***
${DASHBOARD_URL}          ${URL_BASE}
${TITULO_DASHBOARD}       The Internet
${LINK_FORM_AUTH}         css:a[href='/login']
${LINK_CHECKBOXES}        css:a[href='/checkboxes']
${LINK_DROPDOWN}          css:a[href='/dropdown']
${LINK_DYNAMIC_TABLE}     css:a[href='/challenging_dom']
${LINK_ALERTS}            css:a[href='/javascript_alerts']
${LINK_FRAMES}            css:a[href='/iframe']
${LINK_WINDOWS}           css:a[href='/windows']
${HEADING_PRINCIPAL}      css:h1.heading

*** Keywords ***
Navegar A Dashboard
    [Documentation]    Navega a la página principal y verifica el título.
    Go To    ${DASHBOARD_URL}
    Wait Until Page Contains Element    ${HEADING_PRINCIPAL}    timeout=${TIMEOUT_GLOBAL}
    Title Should Be    ${TITULO_DASHBOARD}
    Log    Dashboard cargado correctamente    level=INFO

Navegar A Sección
    [Documentation]    Navega a una sección específica del sitio haciendo clic en su enlace.
    [Arguments]    ${localizador_enlace}    ${titulo_esperado}
    Wait Until Element Is Visible    ${localizador_enlace}    timeout=${TIMEOUT_GLOBAL}
    Click Link    ${localizador_enlace}
    Wait Until Page Contains    ${titulo_esperado}    timeout=${TIMEOUT_GLOBAL}
    Tomar Screenshot    navegacion_${titulo_esperado}
    Log    Navegación a '${titulo_esperado}' completada    level=INFO

Verificar Título De Página
    [Documentation]    Verifica que el heading H3 de la página contiene el texto esperado.
    [Arguments]    ${titulo_esperado}
    Wait Until Page Contains    ${titulo_esperado}    timeout=${TIMEOUT_GLOBAL}
    Page Should Contain    ${titulo_esperado}
```

---

### Paso B2 — Implementar el Page Object de Formularios

**Objetivo:** Crear el recurso para la página de checkboxes y dropdown, practicando interacciones con distintos tipos de controles de formulario.

**Instrucciones:**

1. Crea el archivo `resources/pages/FormPage.resource`:

```robotframework
*** Settings ***
Documentation    Page Object para páginas de formularios de The Internet.
...              Cubre: checkboxes, dropdowns y validación de estados de elementos.
Library          SeleniumLibrary
Resource         ../common/browser.resource

*** Variables ***
# ── Checkboxes ────────────────────────────────────────────────────────────────
${URL_CHECKBOXES}         ${URL_BASE}/checkboxes
${CHECKBOX_1}             css:form#checkboxes input:nth-of-type(1)
${CHECKBOX_2}             css:form#checkboxes input:nth-of-type(2)

# ── Dropdown ──────────────────────────────────────────────────────────────────
${URL_DROPDOWN}           ${URL_BASE}/dropdown
${SELECT_DROPDOWN}        id:dropdown
${OPCION_1}               Option 1
${OPCION_2}               Option 2

# ── Tabla dinámica (Challenging DOM) ─────────────────────────────────────────
${URL_TABLA}              ${URL_BASE}/challenging_dom
${TABLA_PRINCIPAL}        css:table.tablesorter
${FILAS_TABLA}            css:table.tablesorter tbody tr
${ENCABEZADOS_TABLA}      css:table.tablesorter thead th

*** Keywords ***
# ── Keywords para Checkboxes ─────────────────────────────────────────────────

Navegar A Página De Checkboxes
    [Documentation]    Navega a la sección de checkboxes.
    Go To    ${URL_CHECKBOXES}
    Wait Until Page Contains Element    ${CHECKBOX_1}    timeout=${TIMEOUT_GLOBAL}
    Log    Página de Checkboxes cargada    level=INFO

Verificar Estado Checkbox
    [Documentation]    Verifica si un checkbox está marcado o desmarcado.
    [Arguments]    ${localizador}    ${estado_esperado}
    ${estado_actual}=    Run Keyword And Return Status
    ...    Checkbox Should Be Selected    ${localizador}
    IF    ${estado_esperado} == ${TRUE}
        Checkbox Should Be Selected    ${localizador}
        Log    Checkbox está marcado (esperado: marcado)    level=INFO
    ELSE
        Checkbox Should Not Be Selected    ${localizador}
        Log    Checkbox está desmarcado (esperado: desmarcado)    level=INFO
    END

Marcar Checkbox
    [Arguments]    ${localizador}
    Select Checkbox    ${localizador}
    Tomar Screenshot    checkbox_marcado

Desmarcar Checkbox
    [Arguments]    ${localizador}
    Unselect Checkbox    ${localizador}
    Tomar Screenshot    checkbox_desmarcado

# ── Keywords para Dropdown ────────────────────────────────────────────────────

Navegar A Página De Dropdown
    [Documentation]    Navega a la sección de dropdown.
    Go To    ${URL_DROPDOWN}
    Wait Until Page Contains Element    ${SELECT_DROPDOWN}    timeout=${TIMEOUT_GLOBAL}
    Log    Página de Dropdown cargada    level=INFO

Seleccionar Opción Del Dropdown
    [Documentation]    Selecciona una opción del dropdown por texto visible.
    [Arguments]    ${opcion}
    Wait Until Element Is Visible    ${SELECT_DROPDOWN}    timeout=${TIMEOUT_GLOBAL}
    Select From List By Label    ${SELECT_DROPDOWN}    ${opcion}
    Tomar Screenshot    dropdown_seleccionado_${opcion}
    Log    Opción seleccionada: ${opcion}    level=INFO

Verificar Opción Seleccionada En Dropdown
    [Documentation]    Verifica que la opción actualmente seleccionada es la esperada.
    [Arguments]    ${opcion_esperada}
    ${opcion_actual}=    Get Selected List Label    ${SELECT_DROPDOWN}
    Should Be Equal    ${opcion_actual}    ${opcion_esperada}
    Log    Opción verificada: ${opcion_actual}    level=INFO

# ── Keywords para Tabla Dinámica ──────────────────────────────────────────────

Navegar A Página De Tabla
    [Documentation]    Navega a la sección de tabla dinámica (Challenging DOM).
    Go To    ${URL_TABLA}
    Wait Until Page Contains Element    ${TABLA_PRINCIPAL}    timeout=${TIMEOUT_GLOBAL}
    Log    Página de tabla cargada    level=INFO

Obtener Valor De Celda Por Fila Y Columna
    [Documentation]    Retorna el texto de una celda específica usando índices (1-based).
    [Arguments]    ${fila}    ${columna}
    ${xpath_celda}=    Set Variable
    ...    //table[contains(@class,'tablesorter')]//tbody/tr[${fila}]/td[${columna}]
    Wait Until Element Is Visible    xpath:${xpath_celda}    timeout=${TIMEOUT_GLOBAL}
    ${texto}=    Get Text    xpath:${xpath_celda}
    Log    Celda [${fila},${columna}] = '${texto}'    level=INFO
    RETURN    ${texto}

Verificar Encabezado De Columna
    [Documentation]    Verifica que el encabezado de una columna contiene el texto esperado.
    [Arguments]    ${columna}    ${texto_esperado}
    ${xpath_encabezado}=    Set Variable
    ...    //table[contains(@class,'tablesorter')]//thead/th[${columna}]
    ${texto_actual}=    Get Text    xpath:${xpath_encabezado}
    Should Contain    ${texto_actual}    ${texto_esperado}
    Log    Encabezado columna ${columna}: '${texto_actual}'    level=INFO

Contar Filas De Tabla
    [Documentation]    Retorna el número de filas en el cuerpo de la tabla.
    ${count}=    Get Element Count    ${FILAS_TABLA}
    Log    Número de filas en tabla: ${count}    level=INFO
    RETURN    ${count}

Buscar Fila Por Valor En Columna
    [Documentation]    Encuentra la primera fila donde una columna contiene el valor dado.
    ...                Usa XPath avanzado para localizar la celda y retorna el número de fila.
    [Arguments]    ${columna}    ${valor_buscado}
    ${xpath}=    Set Variable
    ...    //table[contains(@class,'tablesorter')]//tbody/tr[td[${columna}][contains(text(),'${valor_buscado}')]]
    Wait Until Page Contains Element    xpath:${xpath}    timeout=${TIMEOUT_GLOBAL}
    ${fila_elemento}=    Get WebElement    xpath:${xpath}
    Log    Fila encontrada con '${valor_buscado}' en columna ${columna}    level=INFO
    RETURN    ${fila_elemento}
```

---

### Paso B3 — Crear los Tests de Formularios y Tablas

**Objetivo:** Escribir y ejecutar los casos de prueba para checkboxes, dropdown y tabla dinámica.

**Instrucciones:**

1. Crea el archivo `tests/02_forms_tables_tests.robot`:

```robotframework
*** Settings ***
Documentation     Suite de pruebas para formularios y tablas de The Internet.
...               Cubre: checkboxes, dropdowns, tablas dinámicas con XPath avanzado.
Resource          ../resources/common/browser.resource
Resource          ../resources/pages/FormPage.resource

Suite Setup       Abrir Navegador Chrome
Suite Teardown    Cerrar Todos Los Navegadores
Test Teardown     Tomar Screenshot En Fallo

*** Test Cases ***
TC-FORM-01: Verificar Estado Inicial De Checkboxes
    [Documentation]    Verifica el estado inicial de los checkboxes en la página.
    [Tags]    formularios    checkboxes    smoke
    Navegar A Página De Checkboxes
    Verificar Estado Checkbox    ${CHECKBOX_1}    ${FALSE}
    Verificar Estado Checkbox    ${CHECKBOX_2}    ${TRUE}
    Tomar Screenshot    tc_form_01_estado_inicial

TC-FORM-02: Marcar Y Desmarcar Checkboxes
    [Documentation]    Verifica que los checkboxes cambian de estado correctamente.
    [Tags]    formularios    checkboxes
    Navegar A Página De Checkboxes
    Marcar Checkbox    ${CHECKBOX_1}
    Verificar Estado Checkbox    ${CHECKBOX_1}    ${TRUE}
    Desmarcar Checkbox    ${CHECKBOX_2}
    Verificar Estado Checkbox    ${CHECKBOX_2}    ${FALSE}
    Tomar Screenshot    tc_form_02_cambio_estado

TC-FORM-03: Seleccionar Opción 1 Del Dropdown
    [Documentation]    Verifica la selección de la primera opción del dropdown.
    [Tags]    formularios    dropdown    smoke
    Navegar A Página De Dropdown
    Seleccionar Opción Del Dropdown    ${OPCION_1}
    Verificar Opción Seleccionada En Dropdown    ${OPCION_1}

TC-FORM-04: Seleccionar Opción 2 Del Dropdown
    [Documentation]    Verifica la selección de la segunda opción del dropdown.
    [Tags]    formularios    dropdown
    Navegar A Página De Dropdown
    Seleccionar Opción Del Dropdown    ${OPCION_2}
    Verificar Opción Seleccionada En Dropdown    ${OPCION_2}

TC-TABLE-01: Verificar Encabezados De Tabla
    [Documentation]    Verifica que los encabezados de la tabla están presentes.
    [Tags]    tablas    smoke
    Navegar A Página De Tabla
    Verificar Encabezado De Columna    1    Lorem
    Verificar Encabezado De Columna    2    Ipsum
    Tomar Screenshot    tc_table_01_encabezados

TC-TABLE-02: Contar Filas De Tabla
    [Documentation]    Verifica que la tabla contiene el número esperado de filas.
    [Tags]    tablas
    Navegar A Página De Tabla
    ${total_filas}=    Contar Filas De Tabla
    Should Be True    ${total_filas} >= 5
    Log    Total de filas encontradas: ${total_filas}    level=INFO
    Tomar Screenshot    tc_table_02_conteo_filas

TC-TABLE-03: Obtener Valor De Celda Específica
    [Documentation]    Recupera y registra el valor de una celda específica de la tabla.
    [Tags]    tablas    xpath_avanzado
    Navegar A Página De Tabla
    ${valor}=    Obtener Valor De Celda Por Fila Y Columna    1    1
    Should Not Be Empty    ${valor}
    Log    Valor celda [1,1]: ${valor}    level=INFO
    Tomar Screenshot    tc_table_03_valor_celda
```

2. Ejecuta la suite de formularios:

```bash
robot --outputdir results \
      --log results/log_forms.html \
      --report results/report_forms.html \
      tests/02_forms_tables_tests.robot
```

**Salida esperada:**

```
==============================================================================
02 Forms Tables Tests
==============================================================================
TC-FORM-01: Verificar Estado Inicial De Checkboxes                | PASS |
TC-FORM-02: Marcar Y Desmarcar Checkboxes                         | PASS |
TC-FORM-03: Seleccionar Opción 1 Del Dropdown                     | PASS |
TC-FORM-04: Seleccionar Opción 2 Del Dropdown                     | PASS |
TC-TABLE-01: Verificar Encabezados De Tabla                       | PASS |
TC-TABLE-02: Contar Filas De Tabla                                 | PASS |
TC-TABLE-03: Obtener Valor De Celda Específica                     | PASS |
==============================================================================
7 tests, 7 passed, 0 failed
==============================================================================
```

**Verificación:** Abre `results/report_forms.html`. Confirma que los screenshots de cada test están visibles en `results/log_forms.html` haciendo clic en los nodos de cada test case.

> **✅ CHECKPOINT BLOQUE B:** Si los 7 tests pasan, continúa al Bloque C. Si hay fallos en los tests de tabla, verifica que la URL `${URL_TABLA}` carga correctamente en el navegador manualmente.

---

## BLOQUE C (66 minutos): Alertas, iFrames y Suite Completa

### Paso C1 — Implementar el Page Object de Componentes Avanzados

**Objetivo:** Crear el recurso para manejar alertas JavaScript e iframes, completando el conjunto de Page Objects del laboratorio.

**Instrucciones:**

1. Crea el archivo `resources/pages/AdvancedPage.resource`:

```robotframework
*** Settings ***
Documentation    Page Object para componentes avanzados de The Internet.
...              Cubre: alertas JavaScript (alert/confirm/prompt), iframes y múltiples ventanas.
Library          SeleniumLibrary
Resource         ../common/browser.resource

*** Variables ***
# ── Alertas JavaScript ────────────────────────────────────────────────────────
${URL_ALERTS}             ${URL_BASE}/javascript_alerts
${BTN_ALERT}              css:button[onclick='jsAlert()']
${BTN_CONFIRM}            css:button[onclick='jsConfirm()']
${BTN_PROMPT}             css:button[onclick='jsPrompt()']
${RESULTADO_ALERTA}       id:result

# ── iFrame ────────────────────────────────────────────────────────────────────
${URL_IFRAME}             ${URL_BASE}/iframe
${IFRAME_EDITOR}          id:mce_0_ifr
${CUERPO_IFRAME}          css:body#tinymce
${TEXTO_EDITOR_DEFAULT}   Your content goes here.

# ── Múltiples Ventanas ────────────────────────────────────────────────────────
${URL_WINDOWS}            ${URL_BASE}/windows
${LINK_NUEVA_VENTANA}     css:a[href='/windows/new']
${TITULO_NUEVA_VENTANA}   New Window

*** Keywords ***
# ── Keywords para Alertas ─────────────────────────────────────────────────────

Navegar A Página De Alertas
    [Documentation]    Navega a la sección de alertas JavaScript.
    Go To    ${URL_ALERTS}
    Wait Until Page Contains Element    ${BTN_ALERT}    timeout=${TIMEOUT_GLOBAL}
    Log    Página de alertas cargada    level=INFO

Disparar Y Aceptar Alerta Simple
    [Documentation]    Dispara un alert() de JavaScript y lo acepta.
    Click Button    ${BTN_ALERT}
    Handle Alert    ACCEPT
    Wait Until Element Is Visible    ${RESULTADO_ALERTA}    timeout=${TIMEOUT_CORTO}
    ${resultado}=    Get Text    ${RESULTADO_ALERTA}
    Tomar Screenshot    alerta_simple_aceptada
    Log    Resultado después de alert: ${resultado}    level=INFO
    RETURN    ${resultado}

Disparar Y Aceptar Confirm
    [Documentation]    Dispara un confirm() de JavaScript y lo acepta.
    Click Button    ${BTN_CONFIRM}
    Handle Alert    ACCEPT
    Wait Until Element Is Visible    ${RESULTADO_ALERTA}    timeout=${TIMEOUT_CORTO}
    ${resultado}=    Get Text    ${RESULTADO_ALERTA}
    Tomar Screenshot    confirm_aceptado
    RETURN    ${resultado}

Disparar Y Rechazar Confirm
    [Documentation]    Dispara un confirm() de JavaScript y lo rechaza (Dismiss).
    Click Button    ${BTN_CONFIRM}
    Handle Alert    DISMISS
    Wait Until Element Is Visible    ${RESULTADO_ALERTA}    timeout=${TIMEOUT_CORTO}
    ${resultado}=    Get Text    ${RESULTADO_ALERTA}
    Tomar Screenshot    confirm_rechazado
    RETURN    ${resultado}

Disparar Prompt Con Texto
    [Documentation]    Dispara un prompt() de JavaScript, ingresa texto y acepta.
    [Arguments]    ${texto_prompt}=Robot Framework
    Click Button    ${BTN_PROMPT}
    Handle Alert    ACCEPT    ${texto_prompt}
    Wait Until Element Is Visible    ${RESULTADO_ALERTA}    timeout=${TIMEOUT_CORTO}
    ${resultado}=    Get Text    ${RESULTADO_ALERTA}
    Tomar Screenshot    prompt_completado
    RETURN    ${resultado}

# ── Keywords para iFrame ──────────────────────────────────────────────────────

Navegar A Página De iFrame
    [Documentation]    Navega a la sección del editor con iframe.
    Go To    ${URL_IFRAME}
    Wait Until Page Contains Element    ${IFRAME_EDITOR}    timeout=${TIMEOUT_GLOBAL}
    Log    Página de iFrame cargada    level=INFO

Obtener Texto Del iFrame
    [Documentation]    Cambia al contexto del iframe, obtiene el texto del editor y vuelve.
    Select Frame    ${IFRAME_EDITOR}
    Wait Until Element Is Visible    ${CUERPO_IFRAME}    timeout=${TIMEOUT_GLOBAL}
    ${texto}=    Get Text    ${CUERPO_IFRAME}
    Tomar Screenshot    contenido_iframe
    Log    Texto en iframe: ${texto}    level=INFO
    Unselect Frame
    RETURN    ${texto}

Escribir En iFrame
    [Documentation]    Escribe texto en el editor dentro del iframe.
    [Arguments]    ${nuevo_texto}=Automatización con Robot Framework
    Select Frame    ${IFRAME_EDITOR}
    Wait Until Element Is Visible    ${CUERPO_IFRAME}    timeout=${TIMEOUT_GLOBAL}
    Execute Javascript    document.querySelector('body#tinymce').innerHTML = '${nuevo_texto}'
    Tomar Screenshot    texto_escrito_iframe
    ${texto_actual}=    Get Text    ${CUERPO_IFRAME}
    Unselect Frame
    Log    Texto escrito en iframe: ${nuevo_texto}    level=INFO
    RETURN    ${texto_actual}

# ── Keywords para Múltiples Ventanas ─────────────────────────────────────────

Navegar A Página De Ventanas
    [Documentation]    Navega a la sección de múltiples ventanas.
    Go To    ${URL_WINDOWS}
    Wait Until Page Contains Element    ${LINK_NUEVA_VENTANA}    timeout=${TIMEOUT_GLOBAL}
    Log    Página de ventanas cargada    level=INFO

Abrir Nueva Ventana Y Verificar
    [Documentation]    Abre una nueva ventana, verifica su título y vuelve a la original.
    ${ventana_original}=    Get Window Handle
    Click Link    ${LINK_NUEVA_VENTANA}
    ${ventanas}=    Get Window Handles
    Should Be True    len(${ventanas}) == 2
    Switch Window    NEW
    Wait Until Page Contains    New Window    timeout=${TIMEOUT_GLOBAL}
    Title Should Be    ${TITULO_NUEVA_VENTANA}
    Tomar Screenshot    nueva_ventana
    Close Window
    Switch Window    ${ventana_original}
    Log    Ventana secundaria verificada y cerrada    level=INFO
```

---

### Paso C2 — Crear los Tests de Componentes Avanzados

**Instrucciones:**

1. Crea el archivo `tests/03_advanced_tests.robot`:

```robotframework
*** Settings ***
Documentation     Suite de pruebas para componentes avanzados de The Internet.
...               Cubre: alertas JavaScript, iframes con TinyMCE y múltiples ventanas.
Resource          ../resources/common/browser.resource
Resource          ../resources/pages/AdvancedPage.resource

Suite Setup       Abrir Navegador Chrome
Suite Teardown    Cerrar Todos Los Navegadores
Test Teardown     Tomar Screenshot En Fallo

*** Test Cases ***
TC-ADV-01: Aceptar Alerta Simple JavaScript
    [Documentation]    Verifica que un alert() se puede aceptar y el resultado es correcto.
    [Tags]    alertas    javascript    smoke
    Navegar A Página De Alertas
    ${resultado}=    Disparar Y Aceptar Alerta Simple
    Should Contain    ${resultado}    You successfully clicked an alert

TC-ADV-02: Aceptar Confirm Dialog
    [Documentation]    Verifica el flujo de aceptar un confirm() de JavaScript.
    [Tags]    alertas    javascript
    Navegar A Página De Alertas
    ${resultado}=    Disparar Y Aceptar Confirm
    Should Contain    ${resultado}    You clicked: Ok

TC-ADV-03: Rechazar Confirm Dialog
    [Documentation]    Verifica el flujo de rechazar un confirm() de JavaScript.
    [Tags]    alertas    javascript
    Navegar A Página De Alertas
    ${resultado}=    Disparar Y Rechazar Confirm
    Should Contain    ${resultado}    You clicked: Cancel

TC-ADV-04: Ingresar Texto En Prompt Dialog
    [Documentation]    Verifica que se puede ingresar texto en un prompt() de JavaScript.
    [Tags]    alertas    javascript
    Navegar A Página De Alertas
    ${resultado}=    Disparar Prompt Con Texto    Robot Framework RFCP
    Should Contain    ${resultado}    You entered: Robot Framework RFCP

TC-ADV-05: Leer Contenido Del iFrame
    [Documentation]    Verifica que se puede acceder al contenido dentro de un iframe.
    [Tags]    iframe    smoke
    Navegar A Página De iFrame
    ${texto}=    Obtener Texto Del iFrame
    Should Not Be Empty    ${texto}
    Log    Contenido del iframe verificado: ${texto}    level=INFO

TC-ADV-06: Escribir En Editor iFrame
    [Documentation]    Verifica que se puede modificar el contenido del editor en iframe.
    [Tags]    iframe
    Navegar A Página De iFrame
    ${texto_nuevo}=    Set Variable    Prueba E2E con Robot Framework 7
    ${texto_escrito}=    Escribir En iFrame    ${texto_nuevo}
    Should Contain    ${texto_escrito}    ${texto_nuevo}

TC-ADV-07: Abrir Y Verificar Nueva Ventana
    [Documentation]    Verifica el manejo de múltiples ventanas del navegador.
    [Tags]    ventanas    smoke
    Navegar A Página De Ventanas
    Abrir Nueva Ventana Y Verificar
    Title Should Be    The Internet
```

2. Ejecuta la suite avanzada:

```bash
robot --outputdir results \
      --log results/log_advanced.html \
      --report results/report_advanced.html \
      tests/03_advanced_tests.robot
```

**Salida esperada:**

```
==============================================================================
03 Advanced Tests
==============================================================================
TC-ADV-01: Aceptar Alerta Simple JavaScript                       | PASS |
TC-ADV-02: Aceptar Confirm Dialog                                 | PASS |
TC-ADV-03: Rechazar Confirm Dialog                                 | PASS |
TC-ADV-04: Ingresar Texto En Prompt Dialog                        | PASS |
TC-ADV-05: Leer Contenido Del iFrame                              | PASS |
TC-ADV-06: Escribir En Editor iFrame                              | PASS |
TC-ADV-07: Abrir Y Verificar Nueva Ventana                        | PASS |
==============================================================================
7 tests, 7 passed, 0 failed
==============================================================================
```

**Verificación:** Confirma en `results/log_advanced.html` que los screenshots de alertas muestran el estado del navegador antes y después de manejar cada diálogo.

---

### Paso C3 — Crear el Script de Ejecución de la Suite Completa

**Objetivo:** Crear scripts multiplataforma para ejecutar las tres suites en secuencia y generar un reporte unificado.

**Instrucciones:**

1. Crea el script para Linux/macOS — `run_all.sh`:

```bash
#!/usr/bin/env bash
# run_all.sh — Ejecuta la suite completa E2E del laboratorio 07-00-01
# Uso: ./run_all.sh [headless]

set -e

HEADLESS=${1:-false}
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
RESULTS_DIR="results/run_${TIMESTAMP}"

echo "========================================================"
echo "  Lab 07-00-01: Suite E2E Web Completa"
echo "  Timestamp: ${TIMESTAMP}"
echo "  Modo headless: ${HEADLESS}"
echo "========================================================"

mkdir -p "${RESULTS_DIR}/screenshots"

robot \
    --outputdir "${RESULTS_DIR}" \
    --log "${RESULTS_DIR}/log_completo.html" \
    --report "${RESULTS_DIR}/report_completo.html" \
    --output "${RESULTS_DIR}/output.xml" \
    --variable MODO_HEADLESS:${HEADLESS} \
    --variable DIR_SCREENSHOTS:"${RESULTS_DIR}/screenshots" \
    --loglevel DEBUG \
    tests/

echo ""
echo "✅ Ejecución completada. Reporte: ${RESULTS_DIR}/report_completo.html"
```

2. Crea el script para Windows — `run_all.ps1`:

```powershell
# run_all.ps1 — Ejecuta la suite completa E2E del laboratorio 07-00-01
# Uso: .\run_all.ps1 [-Headless]
param(
    [switch]$Headless
)

$HeadlessValue = if ($Headless) { "True" } else { "False" }
$Timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$ResultsDir = "results\run_$Timestamp"

Write-Host "========================================================"
Write-Host "  Lab 07-00-01: Suite E2E Web Completa"
Write-Host "  Timestamp: $Timestamp"
Write-Host "  Modo headless: $HeadlessValue"
Write-Host "========================================================"

New-Item -ItemType Directory -Path "$ResultsDir\screenshots" -Force | Out-Null

robot `
    --outputdir $ResultsDir `
    --log "$ResultsDir\log_completo.html" `
    --report "$ResultsDir\report_completo.html" `
    --output "$ResultsDir\output.xml" `
    --variable "MODO_HEADLESS:$HeadlessValue" `
    --variable "DIR_SCREENSHOTS:$ResultsDir\screenshots" `
    --loglevel DEBUG `
    tests\

Write-Host ""
Write-Host "Ejecucion completada. Reporte: $ResultsDir\report_completo.html"
```

3. Da permisos de ejecución (Linux/macOS):

```bash
chmod +x run_all.sh
```

4. Ejecuta la suite completa:

```bash
# Linux / macOS
./run_all.sh false

# Windows (PowerShell)
.\run_all.ps1

# Modo headless (cualquier plataforma)
# Linux/macOS:
./run_all.sh true
# Windows:
.\run_all.ps1 -Headless
```

**Salida esperada:**

```
========================================================
  Lab 07-00-01: Suite E2E Web Completa
  Timestamp: 20240115_143022
  Modo headless: false
========================================================
...
==============================================================================
Tests                                                                 | PASS |
18 tests, 18 passed, 0 failed
==============================================================================
✅ Ejecución completada. Reporte: results/run_20240115_143022/report_completo.html
```

**Verificación:** Abre el reporte unificado en el navegador. Verifica que las tres suites aparecen como nodos separados con sus tests respectivos, todos en verde.

---

## Validación y Pruebas

### Lista de Verificación Final

Ejecuta las siguientes verificaciones para confirmar que el laboratorio está completo:

**1. Verificar la estructura del proyecto:**

```bash
# Linux / macOS
find . -name "*.robot" -o -name "*.resource" -o -name "*.py" | sort

# Windows (PowerShell)
Get-ChildItem -Recurse -Include *.robot,*.resource,*.py | Select-Object FullName
```

**Salida esperada (mínima):**

```
./resources/common/browser.resource
./resources/common/driver_setup.py
./resources/common/variables.resource
./resources/pages/AdvancedPage.resource
./resources/pages/DashboardPage.resource
./resources/pages/FormPage.resource
./resources/pages/LoginPage.resource
./tests/01_login_tests.robot
./tests/02_forms_tables_tests.robot
./tests/03_advanced_tests.robot
```

**2. Ejecutar solo los tests con tag `smoke`:**

```bash
robot --outputdir results/smoke \
      --include smoke \
      --log results/smoke/log_smoke.html \
      --report results/smoke/report_smoke.html \
      tests/
```

**Salida esperada:** Exactamente 7 tests con tag `smoke` deben ejecutarse y pasar (TC-LOGIN-01, TC-LOGIN-04, TC-FORM-01, TC-FORM-03, TC-TABLE-01, TC-ADV-01, TC-ADV-05, TC-ADV-07).

**3. Verificar screenshots generados:**

```bash
# Linux / macOS
ls -la results/screenshots/ | wc -l

# Windows (PowerShell)
(Get-ChildItem results\screenshots -Filter *.png).Count
```

**Resultado esperado:** Al menos 20 archivos `.png` generados.

**4. Verificar el reporte HTML:**

Abre `results/report_completo.html` (o el último directorio `run_*`) y confirma:
- [ ] Las tres suites aparecen con estado PASS
- [ ] El total de tests es 18 (4 + 7 + 7)
- [ ] Los screenshots están embebidos en `log_completo.html`
- [ ] El tiempo de ejecución total es razonable (< 10 minutos en modo visible)

**5. Verificar modo headless:**

```bash
./run_all.sh true   # Linux/macOS
.\run_all.ps1 -Headless   # Windows
```

Los 18 tests deben pasar también en modo headless.

---

## Resolución de Problemas

### Problema 1: ChromeDriver incompatible con la versión de Chrome instalada

**Síntoma:**
```
selenium.common.exceptions.SessionNotCreatedException: Message: session not created: 
This version of ChromeDriver only supports Chrome version 114
Current browser version is 120.0.6099.109
```

**Causa:** La versión de ChromeDriver en el sistema no coincide con la versión de Chrome instalada. Esto ocurre frecuentemente cuando Chrome se actualiza automáticamente pero el driver no.

**Solución:**

```bash
# Opción 1 (recomendada): Forzar la actualización de webdriver-manager
pip install --upgrade webdriver-manager
python -c "
from webdriver_manager.chrome import ChromeDriverManager
from webdriver_manager.core.os_manager import ChromeType
path = ChromeDriverManager().install()
print(f'Driver instalado en: {path}')
"

# Opción 2 (fallback manual para entornos corporativos sin internet):
# 1. Verificar versión de Chrome
google-chrome --version   # Linux
# o: /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --version  # macOS
# o: Menú Chrome → Ayuda → Acerca de Google Chrome  # Windows

# 2. Descargar el driver correspondiente desde:
#    https://googlechromelabs.github.io/chrome-for-testing/
# 3. Colocar el ejecutable en ./drivers/ y añadir al PATH:
export PATH=$PATH:$(pwd)/drivers   # Linux/macOS
$env:PATH += ";$(Get-Location)\drivers"  # Windows PowerShell
```

---

### Problema 2: `Wait Until Element Is Visible` falla por timeout en cargas lentas

**Síntoma:**
```
ElementNotVisibleException: Message: element not interactable
SeleniumLibrary.ElementKeywords - FAIL
Element with locator 'css:div.flash.success' not visible after 15 seconds.
```

**Causa:** La aplicación `the-internet.herokuapp.com` está alojada en un servidor gratuito de Heroku que puede tener latencia alta (5-30 segundos de "cold start") si no ha recibido tráfico recientemente. El timeout global de 15 segundos puede ser insuficiente.

**Solución:**

```robotframework
# Opción 1: Aumentar el timeout global en variables.resource
${TIMEOUT_GLOBAL}         30s   # Aumentar de 15s a 30s

# Opción 2: Agregar un "warm-up" en el Suite Setup de cada suite
Suite Setup    Run Keywords
...    Abrir Navegador Chrome
...    AND    Calentar Servidor

*** Keywords ***
Calentar Servidor
    [Documentation]    Hace una petición inicial para "despertar" el servidor de Heroku.
    Go To    ${URL_BASE}
    Wait Until Page Contains Element    css:h1.heading    timeout=45s
    Log    Servidor calentado correctamente    level=INFO

# Opción 3: Usar la alternativa local con Docker (ver Apéndice A)
```

Si el problema persiste, verifica la conectividad:

```bash
curl -I https://the-internet.herokuapp.com
# Debe retornar HTTP/2 200 en < 10 segundos
```

---

## Limpieza

Una vez completado el laboratorio, realiza los siguientes pasos para liberar recursos:

**1. Cerrar todos los procesos de Chrome:**

```bash
# Linux / macOS
pkill -f "Google Chrome" 2>/dev/null || true
pkill -f chromedriver 2>/dev/null || true

# Windows (PowerShell)
Get-Process chrome -ErrorAction SilentlyContinue | Stop-Process -Force
Get-Process chromedriver -ErrorAction SilentlyContinue | Stop-Process -Force
```

**2. Archivar los resultados:**

```bash
# Comprimir resultados para conservar evidencias
# Linux / macOS
tar -czf lab-07-resultados-$(date +%Y%m%d).tar.gz results/

# Windows (PowerShell)
Compress-Archive -Path results -DestinationPath "lab-07-resultados-$(Get-Date -Format yyyyMMdd).zip"
```

**3. Limpiar archivos temporales de webdriver-manager (opcional):**

```bash
# El caché de webdriver-manager se almacena en:
# Linux/macOS: ~/.wdm/
# Windows: C:\Users\<usuario>\.wdm\
# Solo limpiar si hay problemas de espacio en disco:
python -c "
import os, shutil
wdm_cache = os.path.expanduser('~/.wdm')
if os.path.exists(wdm_cache):
    shutil.rmtree(wdm_cache)
    print('Caché de webdriver-manager eliminado')
"
```

**4. Desactivar el entorno virtual:**

```bash
deactivate
```

**5. Verificar que no quedan procesos activos:**

```bash
# Linux / macOS
ps aux | grep -E "chrome|robot" | grep -v grep

# Windows (PowerShell)
Get-Process | Where-Object {$_.Name -match "chrome|robot"}
```

---

## Resumen

En este laboratorio implementaste un flujo de automatización E2E completo sobre una aplicación web real, aplicando los conceptos de la lección 7.1 sobre configuración de sesiones de navegador y extendiéndolos con patrones profesionales de diseño.

### Logros del Laboratorio

| Área                        | Lo que implementaste                                                    |
|-----------------------------|-------------------------------------------------------------------------|
| **Configuración de entorno**| Gestión automática de ChromeDriver con `webdriver-manager`             |
| **Page Object Pattern**     | 4 archivos `.resource` separados por página/dominio funcional          |
| **Selectores robustos**     | CSS selectors para elementos estáticos, XPath para tablas dinámicas    |
| **Sincronización**          | `Wait Until Element Is Visible` y `Wait Until Page Contains Element`   |
| **Evidencias automáticas**  | Screenshots en pasos críticos y en fallos, integrados en el reporte    |
| **Componentes avanzados**   | Alertas JS (alert/confirm/prompt), iframes y múltiples ventanas        |
| **Suite completa**          | 18 tests organizados en 3 suites con scripts multiplataforma           |

### Estructura Final del Proyecto

```
lab-07-e2e-web/
├── resources/
│   ├── common/
│   │   ├── browser.resource        # Gestión del ciclo de vida del navegador
│   │   ├── driver_setup.py         # Módulo Python para webdriver-manager
│   │   └── variables.resource      # Variables globales compartidas
│   └── pages/
│       ├── LoginPage.resource      # Page Object: Login
│       ├── DashboardPage.resource  # Page Object: Navegación principal
│       ├── FormPage.resource       # Page Object: Formularios y tablas
│       └── AdvancedPage.resource   # Page Object: Alertas, iframes, ventanas
├── tests/
│   ├── 01_login_tests.robot        # Suite: Login (4 tests)
│   ├── 02_forms_tables_tests.robot # Suite: Formularios y tablas (7 tests)
│   └── 03_advanced_tests.robot     # Suite: Componentes avanzados (7 tests)
├── results/                        # Reportes y screenshots generados
├── run_all.sh                      # Script de ejecución (Linux/macOS)
├── run_all.ps1                     # Script de ejecución (Windows)
└── venv/                           # Entorno virtual Python
```

### Conexión con el Siguiente Laboratorio

Los Page Objects y la estructura de proyecto creados en este laboratorio serán la base del **Laboratorio 07-00-02**, donde integrarás pruebas API con `RequestsLibrary` y combinarás flujos web + API en una suite E2E híbrida. La arquitectura de capas (`resources/common/` + `resources/pages/`) se extenderá con `resources/api/` para las keywords de llamadas REST.

---

## Apéndice A: Alternativa Local con Docker

Si `the-internet.herokuapp.com` no está disponible, ejecuta la aplicación localmente:

```bash
# Requisito: Docker instalado y en ejecución
docker pull gprestes/the-internet

# Iniciar el servidor local en el puerto 7080
docker run -d -p 7080:5000 --name the-internet gprestes/the-internet

# Verificar que el servidor está activo
curl -I http://localhost:7080
# Debe retornar HTTP/1.1 200 OK

# Actualizar la variable URL_BASE en variables.resource:
# ${URL_BASE}    http://localhost:7080

# Al finalizar el laboratorio, detener el contenedor:
docker stop the-internet && docker rm the-internet
```

---

## Referencias

- [Documentación oficial de SeleniumLibrary](https://robotframework.org/SeleniumLibrary/SeleniumLibrary.html)
- [The Internet — Aplicación de práctica](https://the-internet.herokuapp.com)
- [webdriver-manager para Python](https://github.com/SergeyPirogov/webdriver_manager)
- [Guía de selectores CSS — MDN Web Docs](https://developer.mozilla.org/es/docs/Web/CSS/CSS_selectors)
- [Tutorial de XPath — W3Schools](https://www.w3schools.com/xml/xpath_intro.asp)
- [Robot Framework User Guide — Keywords y Resources](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#resource-and-variable-files)
- [Estándar W3C WebDriver](https://www.w3.org/TR/webdriver2/)

---
LAB_END---
