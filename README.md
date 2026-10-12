# MQTT_Client_Examples

Proyecto de **PLCnext Engineer** (Phoenix Contact) que muestra cómo implementar un cliente MQTT en un controlador PLCnext, suscribirse a un tópico, extraer un valor de un payload JSON y graficarlo en una página HMI local.

## 1. Datos generales del proyecto

| Campo | Valor |
|---|---|
| Nombre del proyecto | `MQTT_Client_Examples` |
| Herramienta | PLCnext Engineer 9.0.355.0 |
| Versión de producto / perfil | 2026.0 |
| Controlador de destino | Phoenix Contact **AXC F 2152** (Rev. ≥ 00/2026.0.0) |
| Librería principal | **MQTT_Client (Version 5.0)** — librería oficial de Phoenix Contact para PLCnext |

## 2. Hardware configurado

El proyecto incluye una configuración de hardware (`CFG_...`) con:

- **Controlador:** AXC F 2152 (hostname de ejemplo: `axc-f-2152-1`)
- **Bus Axioline F local**, con los módulos:
  - `AXL F AI2 AO2 1H` — módulo analógico (2 entradas / 2 salidas)
  - `AXL F DI8/1 DO8/1 1H` — módulo digital (8 entradas / 8 salidas)
- Maestros adicionales ya preconfigurados en la plantilla de hardware (no necesariamente usados por el ejemplo MQTT): **PROFINET** (PN_Master), **Inline Bus** (IB_Master / IB_RemoteMaster) y **EtherNet/IP** (EIP_Master)
- Servidor de **alarmas** (`ALARM_Server`) y servidor **HMI** (`HMI_Server`) activos
- Un área **Safety** está declarada en el proyecto pero vacía (sin lógica de seguridad configurada)

## 3. Estructura lógica

```
Logical Elements/
└── HiveMQ.pou/              ← único POU activo
    ├── Code.ld               (red Ladder/FBD, lógica principal)
    └── ClockPulse.st         (pulso de reloj 1 Hz, excluido del build)
```

El POU **`HiveMQ`** se instancia como programa **`Main`**, ejecutado en la tarea cíclica **`MQTT`** (100 ms, `ESM1`). A pesar del nombre del POU, la lógica no es específica de HiveMQ: usa el bloque genérico `MQTT_Client_5`, por lo que funciona contra **cualquier broker MQTT estándar** (HiveMQ, Mosquitto, EMQX, etc.).

## 4. Qué hace la lógica (resumen funcional)

1. **Conexión al broker** (`MQTT_Client_31 : MQTT_CLIENT_5`):
   - `strURI := '192.168.1.1'`
   - `strClientID := 'TESTDEVICE123456'`
   - `xCleanSession := TRUE`
   - `xSSL := FALSE` (conexión sin TLS en este ejemplo)
   - Habilitado con `xConnect`, reseteable con `xReset`
2. **Suscripción a un tópico** (`MQTT_Subscribe_1 : MQTT_Subscribe_2`):
   - `strTopic := 'Ejemplo/PData'`
   - `diQOS := 0`
   - Se activa automáticamente una vez conectado (`xConnected`)
   - El reintento ante error se gestiona con `xErrorSubAutoAck`
3. **Procesamiento del mensaje recibido:**
   - Al llegar un mensaje nuevo (`xNewMessage`), se localiza la clave `'pdin'` dentro del payload (bloques `FIND`, `MID`, `TO_INT`)
   - El valor entero extraído se escala (`× 0.1`) para obtener una magnitud con un decimal
   - El resultado se escribe en la variable global **`Temperatura_IOT : REAL`**
   - Un contador (`CTU2`) lleva la cuenta de mensajes recibidos (`iTotalReceivedMsgs`)

## 5. Variables principales

| Variable | Tipo | Descripción |
|---|---|---|
| `xConnect` | BOOL (global) | Habilita la conexión al broker |
| `xReset` | BOOL | Resetea la conexión MQTT |
| `xConnected` | BOOL | Indica conexión activa con el broker |
| `xSubscribedBasic` | BOOL | Indica suscripción activa al tópico |
| `xErrorSubAutoAck` | BOOL | Rearme automático de errores de suscripción |
| `strMessage` | `strMsg` | Último payload recibido |
| `iTotalReceivedMsgs` | INT | Contador de mensajes recibidos |
| `Temperatura_IOT` | REAL (global, salida) | Valor numérico extraído del payload (p. ej. una temperatura) |
| `xClockPulse` | BOOL | Pulso de 1 Hz generado por `TON1`/`TON2` |

> El bloque `MQTT_Client_5` expone además entradas no usadas en este ejemplo pero disponibles en la librería: usuario/contraseña, certificados cliente/servidor (TLS mutuo), *Last Will and Testament* (LWT), `KeepAliveInterval`, `ConnectionTimeout`, cifrado (`strEnabledCipherSuites`, `strALPN`), etc.

## 6. Visualización (HMI local)

El proyecto incluye una página HMI (`Pagina.hmi`) con un gráfico de tendencia (`HmiChart`) que traza en tiempo real la variable `Main.Temperatura_IOT`, útil para verificar visualmente los datos recibidos por MQTT.

## 7. Ejemplos adicionales (en la papelera de reciclaje)

Dentro del proyecto, en la **Papelera de reciclaje** (`RecycleBin`, visible solo en PLCnext Engineer), se conservan dos POU adicionales que fueron eliminados pero son recuperables:

- **`AWS.pou`** — ejemplo de conexión a **AWS IoT Core** (usa `MQTT_Client_3`, `MQTT_Publish_1`, `MQTT_Subscribe_1` con autenticación por certificado: `strCertificate`, `strPrivateKey`, `strCA`)
- **`Azure.pou`** — ejemplo de conexión a **Azure IoT Hub** (misma base que el de AWS, más el bloque específico `MQTT_Azure_1`)

Ambos usan versiones más antiguas de los bloques de la librería MQTT (`_1`/`_3`) frente al `MQTT_Client_5` usado en el ejemplo activo. Si se desean recuperar, se puede restaurar el POU desde la papelera de reciclaje del árbol de proyecto en PLCnext Engineer.

## 8. Cómo usar este proyecto

1. Abrir el archivo `.pcwex` con **PLCnext Engineer 2026.0** (build 9.0.355.0) o posterior.
2. Verificar que la librería **MQTT_Client (Version 5.0)** esté instalada/disponible (PLCnext Store o carpeta local de librerías).
3. Ajustar según el entorno real:
   - IP/host del broker (`strURI`, actualmente `192.168.1.1`)
   - `strClientID` (actualmente un valor de ejemplo, `TESTDEVICE123456`)
   - Tópico de suscripción (`strTopic`, actualmente `Ejemplo/PData`)
   - Formato esperado del payload (actualmente busca una clave `pdin`)
4. Verificar/ajustar la configuración de hardware si el controlador físico no es un AXC F 2152, o si no se usan los módulos Axioline configurados.
5. Compilar y descargar el proyecto al controlador.
6. Activar `xConnect := TRUE` para iniciar la conexión y suscripción.
7. Abrir la página HMI (`Pagina`) vía WebVisu para ver la variable `Temperatura_IOT` graficada.

## 9. Notas y recomendaciones

- Este es un proyecto de **ejemplo/demostración**: la IP del broker, el `ClientID` y el tópico son valores de prueba y deben reemplazarse antes de cualquier uso productivo.
- La conexión activa va **sin TLS** (`xSSL := FALSE`). Para entornos productivos o brokers en la nube (AWS IoT, Azure IoT Hub, HiveMQ Cloud, etc.) se recomienda habilitar TLS y, si corresponde, autenticación por certificado, usando las entradas que ya expone `MQTT_Client_5` (`xSSL`, `strCertificate`, `strPrivateKey`, `strMQTTCA`, `strServerCertAuth`).
- El archivo `ClockPulse.st` está excluido del build; se conserva como referencia de un generador de pulso de 1 Hz con dos `TON`.
- El área **Safety** del proyecto está vacía; no es necesaria para el funcionamiento del ejemplo MQTT.
