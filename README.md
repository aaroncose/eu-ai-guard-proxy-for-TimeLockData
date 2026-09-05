<p align="center">
  <img src="assets/banner.jpeg" alt="EU AI Guard Proxy Banner" width="100%">
</p>

# EU AI Guard Proxy · Gobernanza de Agentes con Evidencia TimeLockData

[![tests](https://github.com/aaroncose/eu-ai-guard-proxy/actions/workflows/tests.yml/badge.svg?branch=main)](https://github.com/aaroncose/eu-ai-guard-proxy/actions/workflows/tests.yml)
[![License: Source-Available](https://img.shields.io/badge/License-Source--Available-blue.svg)](LICENSE)
[![Compliance](https://img.shields.io/badge/EU%20AI%20Act-Art.%2012%20%7C%2014%20%7C%2019%20%7C%2026(6)-emerald.svg)](#)
[![Evidence](https://img.shields.io/badge/Evidence-TimeLockData-orange.svg)](https://docs.timelockdata.com)
[![eIDAS](https://img.shields.io/badge/eIDAS-RFC%203161%20TSA-informational.svg)](#)
[![Python](https://img.shields.io/badge/Python-3.12-blue.svg)](#)

Pasarela que se coloca entre las aplicaciones de IA de una empresa y el proveedor de modelos. Revisa cada acción que el agente propone antes de que llegue a ejecutarse, y convierte cada paso de esa revisión en una prueba sellada en TimeLockData.

El reparto es estricto. El proxy actua como capa de gobernanza y TimeLockData se encarga de sellar lso hitos.

## Qué problema resuelve

Un agente de IA con permiso para mover dinero, borrar registros o escribir a clientes plantea dos preguntas distintas.

**Quién autoriza cada operación.** Un registro que se escribe después de los hechos no impide nada. El control tiene que interponerse mientras la orden está en tránsito.

**Quién acredita después qué ocurrió.** Un log propio lo reescribe su dueño en cinco minutos. La prueba solo vale ante un tercero cuando la custodia alguien ajeno a quien tomó la decisión.

## Cómo funciona

El usuario sigue trabajando con su ChatGPT, su Claude o la aplicación que ya use. Lo único que cambia es la dirección a la que apunta esa aplicación.

```mermaid
flowchart LR
    E(["Empleado<br/>en su chat de siempre"])
    M["Proveedor<br/>de modelo"]
    H(["Aprobador"])
    C["Aplicación cliente<br/>ejecuta la acción"]
    A(["Auditor externo"])

    P["<b>EU AI GUARD PROXY</b><br/><br/>Abre el expediente<br/>Retiene la orden<br/>Evalúa la política versionada"]
    V{"Veredicto"}
    T[("<b>TIMELOCKDATA</b><br/>evidencia sellada")]

    E   ==>|"1 · petición"| P
    P  <==>|"2 · propone una acción"| M
    P   ==> V
    V   ==>|"PERMITIR"| C
    V   ==>|"REVISIÓN HUMANA"| H
    V   ==>|"BLOQUEAR"| X["La orden no sale<br/>del proxy"]
    H   ==>|"autoriza"| C
    C   ==>|"3 · resultado"| P
    P   ==>|"4 · mensaje al usuario"| E

    P  -.->|"hitos 1, 2 y 3<br/>antes de ejecutar"| T
    H  -.->|"hito 4"| T
    C  -.->|"hitos 5 y 6"| T
    X  -.->|"violación y cierre"| T
    T   ==>|"consulta pública<br/>sin credenciales"| A

    classDef core fill:#0d47a1,stroke:#0d47a1,color:#ffffff
    classDef seal fill:#f9a825,stroke:#c17900,color:#1a1a1a
    classDef gate fill:#fff3cd,stroke:#c17900,color:#3d2c00
    classDef stop fill:#ffdad6,stroke:#c62828,color:#5c0011
    classDef run fill:#d7f5df,stroke:#2e7d32,color:#0a3d1c
    classDef ext fill:#f2f4f7,stroke:#667085,color:#101828

    class P core
    class T seal
    class V gate
    class X stop
    class C run
    class E,M,H,A ext
```

Las flechas gruesas son el camino de la operación. Las punteadas son las pruebas que salen hacia TimeLockData. Los tres primeros sellos existen antes de que la orden llegue a la aplicación cliente.

### Quién actúa en cada momento

El diagrama recorre el caso más completo, una operación que supera el umbral y pasa por autorización. Los otros desenlaces recortan pasos sobre este mismo camino.

```mermaid
sequenceDiagram
    autonumber

    box rgb(242,244,247) Lado del cliente
        actor E as Empleado
        participant C as Aplicación cliente
    end
    participant M as Proveedor de modelo
    box rgb(227,238,255) Gobernanza · decide
        participant P as EU AI Guard Proxy
    end
    box rgb(255,244,214) Evidencia · acredita
        participant T as TimeLockData
    end
    actor A as Aprobador

    E->>P: transfiere 75.000 € a la cuenta X
    activate P
    P->>M: reenvía la petición
    M-->>P: propone execute_bank_transfer
    Note over P: La orden queda retenida.<br/>La aplicación cliente no la ha visto.

    rect rgb(255,248,225)
        P->>T: hito 1 · acción propuesta
        P->>T: hito 2 · política y versión aplicada
        P->>T: hito 3 · decisión · REVISIÓN HUMANA
    end

    P-->>E: pendiente de autorización · exec_a1b2c3
    deactivate P

    A->>P: autoriza
    activate P
    P->>T: hito 4 · aprobación humana
    P->>T: hito 5 · inicio de ejecución
    P-->>C: entrega la orden
    deactivate P

    C->>C: ejecuta la transferencia
    C->>P: informa del resultado
    activate P
    P->>T: hito 6 · resultado
    P->>T: cierre con el recuento de pasos
    P-->>E: operación completada
    deactivate P

    Note over T: Cada hito incorpora el digest del anterior.<br/>Suprimir uno, reordenarlos o cortar la traza rompe la cadena.
```

El bloque sombreado es la parte que Bartolomé describe como evidencia previa. Esas tres pruebas existen antes de que la orden salga del proxy, así que ninguna acción llega a ejecutarse sin dejar constancia de qué se pretendía, con qué reglas y con qué decisión.

El proxy decide y TimeLockData acredita. Ninguno de los dos hace el trabajo del otro.

Los tres veredictos terminan con un mensaje al usuario en su propio chat. El proxy habla el idioma de la API del proveedor, así que ese aviso llega como si lo hubiera escrito el asistente, sin instalar nada en el lado del cliente.

### El expediente de cada operación

Cada acción abre un `execution_id`, el número de expediente que comparten todos sus pasos. Ese expediente sobrevive entre peticiones, ya que la aprobación humana llega en un momento distinto al de la propuesta, y el resultado de la ejecución en otro.

```text
exec_a1b2c3d4e5f6::01::ACTION_INTENT::723f8de5f133d68553f09bcb47a748c1
exec_a1b2c3d4e5f6::02::POLICY_VERSION_APPLIED::339c02664eac7b3405385e4ee3dfccba
exec_a1b2c3d4e5f6::03::POLICY_DECISION::a765ddf384d2f4a2f5c96a9191e6821f
exec_a1b2c3d4e5f6::04::HUMAN_APPROVAL_DECISION::1c48dc423086f32e780d9e83e43add1a
exec_a1b2c3d4e5f6::05::EXECUTION_STARTED::4d81129fc30a8dd2656013b7c73875ed
exec_a1b2c3d4e5f6::06::EXECUTION_RESULT::e58c1a94b7d0f2361c8ab04d9e5f7231
exec_a1b2c3d4e5f6::07::EXECUTION_TRACE_CLOSED::d72e43bca3400c9b6e5d293dae1a2edd
```

Cada línea lleva el expediente, el número de orden, qué ocurrió y la huella del contenido truncada a 32 caracteres. La huella es un código que cambia por completo si se altera una sola coma de lo que representa.

`GET /merkle/verify` de TimeLockData devuelve el identificador, la transacción de blockchain y la fecha, sin devolver los metadatos de la evidencia. Meter la huella dentro del propio identificador hace que esa consulta acredite el contenido, y no solo que el registro existe.

`POLICY_VERSION_APPLIED` va antes que `POLICY_DECISION`. La regla queda registrada con anterioridad al veredicto que dicta.

### Política versionada

Las reglas viven en `config/governance_policy_v1.json`, fuera del código. Declara qué herramientas están permitidas, con qué límites, qué hitos son de sellado obligatorio y qué secuencia exacta corresponde a cada desenlace.

El fichero se hashea al cargarlo. Esa huella viaja en cada evidencia e identifica la versión que dictó cada veredicto, de modo que un auditor detecta si alguien cambió las reglas después.

Las comprobaciones son declarativas y sirven para cualquier sector cambiando solo el JSON.

```json
{
  "id": "amount_threshold",
  "field": "amount",
  "operator": "max",
  "value": 1000,
  "on_fail": "HUMAN_REVIEW",
  "reason": "El importe supera el limite que se aprueba de forma automatica"
}
```

Operadores disponibles: `required`, `in`, `not_in`, `max`, `min`, `max_exclusive`, `min_exclusive`, `matches`, `not_matches`. Un `BLOCK` prevalece siempre sobre un `HUMAN_REVIEW`, con independencia del orden en que estén escritas las reglas.

### Los cinco desenlaces

Toda operación termina con un mensaje al usuario en su propio chat. El proxy habla el idioma de la API del proveedor, así que el aviso llega como si lo hubiera escrito el asistente, sin instalar nada en el lado del cliente.

| Desenlace | Qué recibe el usuario | HTTP |
|---|---|---|
| `ALLOW` | Respuesta normal, sin señales de que hay un proxy en medio | 200 |
| `BLOCK` | Motivo del bloqueo y número de expediente | 403 |
| `HUMAN_REVIEW_APPROVED` | Aviso de que espera autorización, y la orden sale cuando la repite | 202 y luego 200 |
| `HUMAN_REVIEW_REJECTED` | La denegación queda sellada y la orden no sale | 202 |
| `HUMAN_REVIEW_EXPIRED` | El plazo venció sin decisión y el expediente se cierra solo | 202 |
| `EXECUTION_RESULT_MISSING` | La aplicación no informó del resultado dentro del plazo | 200 |
| `ABORTED_EVIDENCE_FAILURE` | La prueba obligatoria no llegó a emitirse y la acción se detuvo | 503 |

Un bloqueo es una prueba tan válida como una ejecución, y es la que demuestra que el control funcionó.

### Sellado obligatorio

La política declara `on_failure: abort`. Un hito obligatorio que no llega a sellarse detiene la operación antes de que la orden salga hacia la aplicación cliente. El dinero se mueve solo cuando existe prueba de que se mueve.

Los fallos de red y los errores 5xx del servidor se reintentan. Los 4xx, que señalan un problema en la petición, se devuelven en el primer intento.

### Detección de huecos

El último hito es un sello de cierre que declara cuántos pasos hubo y cuál era la huella final. Cortar la traza por el final deja los eslabones restantes intactos, y ese recuento es el que delata la supresión.

Suprimir el hito de aprobación humana de una traza produce tres incidencias independientes. Eslabón roto en el paso siguiente, hueco frente a la secuencia que la política exige, y descuadre del recuento declarado en el cierre.

### Privacidad

A TimeLockData viajan la huella del hito, la huella del hito anterior, el número de expediente, el orden, el tipo de hito, los metadatos de la política y la marca de tiempo. El IBAN, el importe y el beneficiario se quedan en el ledger local con el enmascarado de datos personales ya aplicado.

## Límites del sistema

**Gobierna lo que propone el agente, no todo lo que hace la empresa.** Una persona que entra en el CRM y borra un cliente a mano queda fuera del alcance.

**El bloqueo depende de que la aplicación cliente respete el 403.** Lo que no se puede saltar es la prueba, ya que el intento queda sellado igualmente.

**Todo el tráfico tiene que pasar por el proxy.** El cliente reparte a sus aplicaciones la clave del proxy y revoca las claves directas del proveedor. Con las claves directas en circulación, cualquiera cambia una línea de configuración y se salta la capa.

**Vigila lo que las reglas digan.** Comprueba importes, destinos, herramientas y permisos. No juzga si el modelo alucina ni si el texto es sesgado.

**El streaming se gobierna reteniendo solo lo necesario.** Los primeros fragmentos se retienen hasta saber qué trae la respuesta. El texto de conversación se suelta y sigue fluyendo en directo, con un retraso de milisegundos. Una llamada a herramienta corta el envío, se recompone entera y pasa por la política antes de que nada salga hacia el cliente.

## Capas heredadas del proxy original

Estas capas siguen en el código y se activan por configuración. En la integración con TimeLockData quedan apagadas, ya que el sello de tiempo cualificado y el anclaje los aporta la propia notaría.

| Capa | Variable | Estado por defecto |
|---|---|---|
| Sello de tiempo eIDAS RFC 3161 contra TSA | `S3_ENABLED` | Apagada |
| Anclaje en Sigstore Rekor | `S3_ENABLED` | Apagada |
| Firma asimétrica ECDSA NIST P-256 | `S3_ENABLED` | Apagada |
| Almacenamiento WORM S3 Object Lock | `S3_ENABLED` | Apagada |
| Lote diario de medianoche | `S3_ENABLED` | Apagada |

El encadenamiento SHA-256 del tráfico y el enmascarado de datos personales siguen activos siempre.

## Requisitos

- Python 3.12
- Docker y Docker Compose para despliegue contenerizado
- SQLite en desarrollo, PostgreSQL 15+ en producción

## Instalación

```bash
python3.12 -m venv venv
source venv/bin/activate
pip install -e ".[dev]"
cp .env.example .env
```

## Configuración

| Variable | Valor por defecto | Descripción |
|---|---|---|
| `HOST` | `0.0.0.0` | Dirección de escucha del proxy |
| `PORT` | `8000` | Puerto de escucha del proxy |
| `UPSTREAM_BASE_URL` | `https://api.openai.com/v1` | Endpoint de inferencia del proveedor |
| `UPSTREAM_API_KEY` | `""` | Clave del proveedor de inferencia |
| `DATABASE_URL` | `sqlite+aiosqlite:///...` | Conexión asíncrona a base de datos |
| `PROXY_API_KEY` | `sk-guard-local-dev-key` | Clave que usan las aplicaciones cliente |
| `PROXY_PUBLIC_URL` | `http://localhost:8000` | URL del proxy. En producción con https, con el certificado en el balanceador |
| `GOVERNANCE_ENABLED` | `True` | Activa el control de acciones y el expediente |
| `GOVERNANCE_POLICY_PATH` | `config/governance_policy_v1.json` | Fichero de política vigente |
| `GOVERNANCE_APPROVAL_KEY` | `sk-approval-local-dev-key` | Clave de los endpoints de aprobación, distinta de la del proxy |
| `TIMELOCK_ENABLED` | `False` | Con `False` los hitos se sellan en modo simulado |
| `TIMELOCK_BASE_URL` | `https://api.timelockdata.com/api/v1` | API de TimeLockData |
| `TIMELOCK_API_KEY` | `""` | Credencial de TimeLockData |
| `TIMELOCK_COMPANY_ID` | `""` | Identificador de cuenta en TimeLockData |
| `TIMELOCK_EVIDENCE_OWNER` | `eu-ai-guard-proxy` | Propietario inicial de cada evidencia |
| `TIMELOCK_CATEGORY_ID` | `""` | Categoría existente a reutilizar. Vacío para resolverla al arrancar |
| `TIMELOCK_SUBCATEGORY_ID` | `""` | Subcategoría existente a reutilizar |
| `TIMELOCK_SEAL_TIMEOUT` | `5.0` | Plazo del sellado por hito, que bloquea la respuesta al usuario |
| `TIMELOCK_SEAL_MAX_ATTEMPTS` | `2` | Reintentos del sellado por hito |
| `S3_ENABLED` | `False` | Activa las capas heredadas y el lote diario |

La clave de aprobación es distinta de la del proxy para que quien opera el agente no pueda autorizar las acciones que ese mismo agente propone.

Sin credenciales de TimeLockData el sistema funciona en modo simulado. La traza se construye igual, los hitos quedan marcados `SIMULATED` y ninguna petición sale hacia la API.

## Demostración

```bash
python run_governance_demo.py
```

Recorre los tres desenlaces con un proveedor de modelo simulado dentro del proceso, sin necesidad de clave de OpenAI ni de servidor levantado. Deja las trazas en `demo_output/`.

| Escenario | Acción del agente | Veredicto |
|---|---|---|
| 1 | Transferir 250 € a un proveedor habitual | PERMITIR |
| 2 | Transferir 800 € a una cuenta vetada | BLOQUEAR |
| 3 | Transferir 75.000 € | REVISIÓN HUMANA |

## Servicios

```bash
uvicorn proxy.main:app --host 0.0.0.0 --port 8000
streamlit run dashboard/app.py
```

O con Docker Compose.

```bash
docker compose up -d --build
```

- Proxy de inferencia y gobernanza [http://localhost:8000](http://localhost:8000)
- Consola de expedientes y aprobaciones [http://localhost:8501](http://localhost:8501)

## Integración con aplicaciones cliente

Sin SDK propietario. Se redefine `base_url` en el cliente estándar.

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="sk-guard-local-dev-key"
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Transfiere 250 euros al proveedor"}],
    tools=[...],
    extra_headers={
        "X-App-ID": "banca-agente",
        "X-User-ID": "empleado-77"
    }
)
```

La respuesta trae la cabecera `X-Execution-Id` con el número de expediente. Tras ejecutar la herramienta, la aplicación informa del resultado.

```python
import httpx

httpx.post(
    f"http://localhost:8000/api/v1/governance/executions/{execution_id}/result",
    json={"success": True, "summary": "Transferencia emitida", "result_reference": "TRX-99812"},
    headers={"Authorization": "Bearer sk-guard-local-dev-key"}
)
```

## Endpoints

### Inferencia

- `POST /v1/chat/completions` intercepta, enmascara datos personales, gobierna la acción propuesta y registra la interacción
- `GET /v1/models` modelos expuestos

### Gobernanza

- `GET /api/v1/governance/policy` política vigente y su huella, sin credenciales
- `GET /api/v1/governance/pending` acciones que esperan decisión humana, con `X-Approval-Key`
- `POST /api/v1/governance/executions/{id}/approve` decisión de quien autoriza, con `X-Approval-Key`
- `POST /api/v1/governance/executions/{id}/result` resultado que informa la aplicación cliente, con la clave del proxy
- `GET /api/v1/governance/executions/{id}` traza completa con el enlace público de cada hito, sin credenciales

### Auditoría

- `GET /api/v1/audit/verify` verificación matemática de la secuencia histórica de tráfico
- `GET /api/v1/audit/export` expediente ZIP con registros canónicos, manifiesto firmado y clave pública

### Salud

- `GET /healthz` estado operativo
- `GET /livez` liveness probe

## Verificación independiente

```bash
python verify_governance.py --url http://localhost:8000 <execution_id> config/governance_policy_v1.json
python verify_governance.py demo_output/<execution_id>.trace.json config/governance_policy_v1.json
```

Opera sobre la traza que devuelve el endpoint público, sin credenciales y sin acceso a la base de datos del proxy.

| # | Comprobación | Detalle |
|---|---|---|
| 0 | Origen de la política | La huella del fichero aportado se recalcula y se contrasta con la de la traza |
| 1 | Coherencia de la cadena | Numeración correlativa y cada eslabón apuntando al hito precedente |
| 2 | Secuencia | Los hitos registrados cubren los que la política exige para el desenlace, en orden |
| 3 | Sello de cierre | El recuento de hitos y la huella final coinciden con el sello |
| 4 | Huella incrustada | La huella del identificador corresponde al contenido declarado |
| 5 | Anclaje | Cada evidencia existe en TimeLockData, vía `GET /merkle/verify` |

Las comprobaciones 0 a 4 operan sobre datos que aporta quien entrega la traza, de modo que un informe fabricado desde cero puede superarlas. La 5 se resuelve contra TimeLockData, donde la fecha y la transacción de blockchain quedan fuera del alcance de quien opera el proxy. Ahí está el valor de que la evidencia la custodie un tercero.

Códigos de salida: 0 verificado, 1 error de uso, 2 verificación fallida.

## Estructura

```text
config/
  governance_policy_v1.json     Política versionada
proxy/
  routes/proxy_routes.py        Intercepción del tráfico
  routes/governance_routes.py   Aprobación, resultado y consulta de trazas
  security/policy_engine.py     Evaluación declarativa de políticas
  security/timelock_notary.py   Sellado de hitos en TimeLockData
  security/crypto_chain.py      Huellas y encadenamiento
  security/dlp_filter.py        Enmascarado de datos personales
  services/governance_flow.py   Orquestación de la gobernanza
  services/governance_trace.py  Ciclo de vida del expediente
  storage/                      Capas heredadas, apagadas por defecto
dashboard/app.py                Consola de expedientes y aprobaciones
verify_governance.py            Verificador independiente
run_governance_demo.py          Los tres escenarios
```

## Pruebas manuales

```bash
python tests/manual_client.py        # inferencia con PII, filtrado DLP y encadenamiento
python tests/manual_tamper.py        # alteración directa de un registro para probar la detección
python tests/manual_shield_test.py   # llamada con SQL destructivo contra el guardarraíl
python tests/manual_trigger_batch.py # consolidación forzada del lote diario
```

## Suite automatizada

```bash
pytest -v
```

## Referencias

- [Documentación de TimeLockData](https://docs.timelockdata.com)
- [Reglamento UE 2024/1689 (EU AI Act)](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)

## Licencia y contacto empresarial (Source-Available)

Este proyecto está publicado bajo un modelo de **Licencia Source-Available / Propiedad Comercial Privada** (consulte el archivo [LICENSE](LICENSE) para conocer los términos completos).

* **Finalidad:** Demostración pública de arquitectura, portafolio técnico y evaluación de ingeniería para profesionales, auditores y responsables de gobernanza.
* **Restricciones:** Queda estrictamente prohibido el uso en producción, la explotación comercial, la copia, redistribución o modificación de este software sin un acuerdo de licencia comercial previo.

### Consultoría e integraciones empresariales

Si su organización o equipo de producto requiere:
- Licenciamiento comercial para despliegue productivo on-premise o cloud.
- Adaptación a medida de la pasarela de gobernanza para arquitecturas de agentes complejas.
- Auditoría técnica y preparación ante inspecciones del Reglamento Europeo de IA (EU AI Act).

**Contacto directo con el autor:**
- **Autor:** Aarón Cortés Serrano
- **LinkedIn:** [linkedin.com/in/aaroncortes](https://www.linkedin.com/in/aaroncortesseerrano/)
- **GitHub:** [github.com/aaroncose](https://github.com/aaroncose)
