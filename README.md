# Ejemplo Clientes MQTT

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
- Servidor **HMI** (`HMI_Server`)

## 3. Funcionamiento

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

## 4. Variables principales

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

El bloque `MQTT_Client_5` expone además entradas no usadas en este ejemplo pero disponibles en la librería: usuario/contraseña, certificados cliente/servidor (TLS mutuo), *Last Will and Testament* (LWT), `KeepAliveInterval`, `ConnectionTimeout`, cifrado (`strEnabledCipherSuites`, `strALPN`), etc.

## 5. Visualización (HMI local)

El proyecto incluye una página HMI (`Pagina.hmi`) con un gráfico de tendencia (`HmiChart`) que traza en tiempo real la variable `Main.Temperatura_IOT`, útil para verificar visualmente los datos recibidos por MQTT.

