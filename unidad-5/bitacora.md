# Unidad 5
## Bitácora de proceso de aprendizaje

### Actividad 1
#### Paso 1
- En ACSII se traduce cada caracter
- 1.67903
- ACSII --> "1" "." "6" ... "3": 7 Caracteres
-            31  2E  ...     33: 7 Bytes
- Se traducen segun lo qque signifique cada caracter en ACSII

- En codigo binario, se suele ver mas pequeño. por lo cual consume menos memoria, pero la traduccion del ACSII requiere mas espacio en memoria.

- Se tiene que especificar si se envia en big endian o little endian, big siendo con el dato de mas peso primero y el little siendo el dato con menos peso primero.

- 524 --> 02 0c -> Big endian
- 524 --> 0c 02 -> Little endian

#### ¿Qué ventajas y desventajas ves en usar un formato binario en lugar de texto ASCII?
- Que el binario es mas compacta su traducción que en ACSII, ya que en este ultimo se suele traducir caracter por caracter, su traducción toma más tiempo y al ser tan extenso consume mucho más memoria.

#### Si xValue=500, yValue=524, aState=True, bState=False, ¿cómo se vería el paquete en hexadecimal? (Pista: convierte cada valor según su tipo y anota los bytes en orden.) Respuesta esperada: ```01 F4 02 0C 01 00```

#### Paso 2
#### ¿Por qué el protocolo ASCII de la unidad anterior no tenía este problema de sincronización? (Pista: piensa en qué rol cumplía el carácter \n.)
- No se tenia este problema en ASCII ya que el caracter \n nos ayudaba con esto, saltando de linea cuando el segmento de los datos o la trama se repetian.

#### ¿Por qué en binario no podemos usar \n como delimitador?
- No se puede usar \n en binario ya que este carácter pertenece a ACSII y no a binario, por lo que el sistema binario lo puede interpretar como un carácter más y no un sincronizador.

#### Paso 3
checksum = sum(data) % 256: suma todos los bytes de datos y lo ajusta a 1 byte (0–255).
packet = b'\xAA' + data + bytes([checksum]): concatena header + datos + checksum.

## Bitácora de aplicación 

### MicrobitBinaryAdapter.js 
```
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class MicrobitBinaryAdapter extends BaseAdapter {
    constructor({ path, baud = 115200, verbose = false } = {}) {
        super();
        this.path = path;
        this.baud = baud;
        this.port = null;
        this.buf = Buffer.alloc(0);
        this.verbose = verbose;
    }

    async connect() {
        if (this.connected) return;
        if (!this.path) throw new Error("serialPort is required for microbit device mode");

        this.port = new SerialPort({
            path: this.path,
            baudRate: this.baud,
            autoOpen: false,
        });

        await new Promise((resolve, reject) => {
            this.port.open((err) => (err ? reject(err) : resolve()));
        });

        this.connected = true;
        this.onConnected?.(`serial open ${this.path} @${this.baud}`);

        this.port.on("data", (chunk) => this._onChunk(chunk));
        this.port.on("error", (err) => this._fail(err));
        this.port.on("close", () => this._closed());
    }

    async disconnect() {
        if (!this.connected) return;
        this.connected = false;

        if (this.port && this.port.isOpen) {
            await new Promise((resolve, reject) => {
                this.port.close((err) => {
                    if (err) reject(err);
                    else resolve();
                });
            });
        }

        this.port = null;
        this.buf = Buffer.alloc(0);
        this.onDisconnected?.("serial closed");
    }

    getConnectionDetail() {
        return `serial open ${this.path}`;
    }

    _onChunk(chunk) {
        if (!Buffer.isBuffer(chunk)) {
            chunk = Buffer.from(chunk);
        }

        this.buf = Buffer.concat([this.buf, chunk]);

        const HEADER = 0xAA;
        const FRAME_SIZE = 8;

        while (this.buf.length >= FRAME_SIZE) {
            const headerIdx = this.buf.indexOf(HEADER);

            if (headerIdx < 0) {
                if (this.verbose) {
                    console.log("Bad data: sync lost, dropping buffer");
                }
                this.buf = Buffer.alloc(0);
                return;
            }

            if (headerIdx > 0) {
                if (this.verbose) {
                    console.log(`Bad data: skipping ${headerIdx} byte(s) before header`);
                }
                this.buf = this.buf.slice(headerIdx);
            }

            if (this.buf.length < FRAME_SIZE) return;

            const frame = this.buf.slice(0, FRAME_SIZE);

            if (frame[0] !== HEADER) {
                this.buf = this.buf.slice(1);
                continue;
            }

            const x = frame.readInt16BE(1);
            const y = frame.readInt16BE(3);
            const btnA = frame[5];
            const btnB = frame[6];
            const recvChk = frame[7];

            const calculatedChk =
                (frame[1] + frame[2] + frame[3] + frame[4] + frame[5] + frame[6]) & 0xff;

            if (calculatedChk !== recvChk) {
                console.warn(
                    `Corrupt binary frame discarded: expected ${calculatedChk}, got ${recvChk}, raw: ${frame.toString("hex").toUpperCase()}`
                );
                this.buf = this.buf.slice(1);
                continue;
            }

            if (x < -2048 || x > 2047 || y < -2048 || y > 2047) {
                console.warn(
                    `Out-of-range frame discarded: x=${x}, y=${y}, raw: ${frame.toString("hex").toUpperCase()}`
                );
                this.buf = this.buf.slice(1);
                continue;
            }

            this.onData?.({
                x: x | 0,
                y: y | 0,
                btnA: btnA === 1,
                btnB: btnB === 1,
            });

            this.buf = this.buf.slice(FRAME_SIZE);
        }

        if (this.buf.length > 4096) {
            this.buf = this.buf.slice(-7);
        }
    }

    _fail(err) {
        this.onError?.(String(err?.message || err));
        this.disconnect();
    }

    _closed() {
        if (!this.connected) return;
        this.connected = false;
        this.port = null;
        this.buf = Buffer.alloc(0);
        this.onDisconnected?.("serial closed (event)");
    }

    async writeLine(line) {
        if (!this.port || !this.port.isOpen) return;
        await new Promise((resolve, reject) => {
            this.port.write(line, (err) => (err ? reject(err) : resolve()));
        });
    }

    async handleCommand(cmd) {
        if (cmd?.cmd === "setLed") {
            const x = Math.max(0, Math.min(4, Math.trunc(cmd.x)));
            const y = Math.max(0, Math.min(4, Math.trunc(cmd.y)));
            const v = Math.max(0, Math.min(9, Math.trunc(cmd.value)));
            await this.writeLine(`LED,${x},${y},${v}\n`);
        }
    }
}

module.exports = MicrobitBinaryAdapter;
```

### bridgeServer.js 
```
//   Uso:
//     node bridgeServer.js --device sim --wsPort 8081 --hz 30
//     node bridgeServer.js --device microbit --wsPort 8081 --serialPort COM5 --baud 115200
//     node bridgeServer.js --device microbitcopia
//     node bridgeServer.js --device microbitBinary 

//   WS contract:
//    * bridge To client:
//        {type:"status", state:"ready|connected|disconnected|error", detail:"..."}
//        {type:"microbit", x:int, y:int, btnA:bool, btnB:bool, t:ms}
//    * client To bridge:
//        {cmd:"connect"} | {cmd:"disconnect"}
//        {cmd:"setSimHz", hz:30}
//        {cmd:"setLed", x:2, y:3, value:9}


const { WebSocketServer } = require("ws");
const { SerialPort } = require("serialport");
const SimAdapter = require("./adapters/SimAdapter");
const MicrobitAsciiAdapter = require("./adapters/MicrobitASCIIAdapter");
const MicrobitBinaryAdapter = require("./adapters/MicrobitBinaryAdapter");
const MicrobitASCIIAdapterCopia = require("./adapters/MicrobitASCIIAdapterCopia");

const log = {
  info: (...args) => console.log(`[${new Date().toISOString()}] [INFO]`, ...args),
  warn: (...args) => console.warn(`[${new Date().toISOString()}] [WARN]`, ...args),
  error: (...args) => console.error(`[${new Date().toISOString()}] [ERROR]`, ...args)
};


function getArg(name, def = null) {
  const i = process.argv.indexOf(`--${name}`);
  if (i >= 0 && i + 1 < process.argv.length) return process.argv[i + 1];
  return def;
}

function hasFlag(name) {
  return process.argv.includes(`--${name}`);
}

function nowMs() { return Date.now(); }

function safeJsonParse(s) {
  try {
    return JSON.parse(s);

  } catch (e) {
    log.warn("Failed to parse JSON: ", s, e);
    return null;
  }
}

function broadcast(wss, obj) {
  const text = JSON.stringify(obj);
  for (const client of wss.clients) {
    if (client.readyState === 1) client.send(text);
  }
}

function status(wss, state, detail = "") {
  broadcast(wss, { type: "status", state, detail, t: nowMs() });
}

const DEVICE = (getArg("device", "sim") || "sim").toLowerCase();
const WS_PORT = parseInt(getArg("wsPort", "8081"), 10);
const SERIAL_PATH = getArg("serialPort", null);
const BAUD = parseInt(getArg("baud", "115200"), 10);
const SIM_HZ = parseInt(getArg("hz", "30"), 10);
const VERBOSE = hasFlag("verbose");

async function findMicrobitPort() {
  const ports = await SerialPort.list();
  const microbit = ports.find(p =>
    p.vendorId && parseInt(p.vendorId, 16) === 0x0D28
  );
  return microbit?.path ?? null;
}

async function createAdapter() {
  if (DEVICE === "microbit") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
    return new MicrobitAsciiAdapter({ path, baud: BAUD, verbose: VERBOSE });
  }

  if (DEVICE === "microbitbinary") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`microbitBinary found at ${path}`);
    return new MicrobitBinaryAdapter({ path, baud: BAUD, verbose: VERBOSE });
  }

  if (DEVICE === "microbitcopia") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);

    return new MicrobitASCIIAdapterCopia({ path, baud: BAUD, verbose: VERBOSE });
  }

  return new SimAdapter({ hz: SIM_HZ, verbose: VERBOSE });
}

async function main() {
  const wss = new WebSocketServer({ port: WS_PORT });
  log.info(`WS listening on ws://127.0.0.1:${WS_PORT} device=${DEVICE}`);

  const adapter = await createAdapter();

  adapter.onConnected = (detail) => {
    log.info(`[ADAPTER] Device Connected: ${detail}`);
    status(wss, "connected", detail);
  };

  adapter.onDisconnected = (detail) => {
    log.warn(`[ADAPTER] Device Disconnected: ${detail}`);
    status(wss, "disconnected", detail);
  };

  adapter.onError = (detail) => {
    log.error(`[ADAPTER] Device Error: ${detail}`);
    status(wss, "error", detail);
  };

  adapter.onData = (d) => {
    broadcast(wss, {
      type: "microbit",
      x: d.x,
      y: d.y,
      btnA: !!d.btnA,
      btnB: !!d.btnB,
      t: nowMs(),
    });
  };

  status(wss, "ready", `bridge up (${DEVICE})`);

  wss.on("connection", (ws, req) => {
    log.info(`[NETWORK] Remote Client connected from ${req.socket.remoteAddress}. Total clients: ${wss.clients.size}`);

    const state = adapter.connected ? "connected" : "ready";

    const detail = adapter.connected
      ? adapter.getConnectionDetail()
      : `bridge (${DEVICE})`;

    ws.send(JSON.stringify({ type: "status", state, detail, t: nowMs() }));

    ws.on("message", async (raw) => {
      const msg = safeJsonParse(raw.toString("utf8"));
      if (!msg) return;

      if (msg.cmd === "connect") {
        log.info(`[NETWORK] Client requested adapter connect`);

        if (adapter.connected) {
          log.info(`[HW-POLICY] Adapter already open. Sending current status to incoming client.`);
          ws.send(JSON.stringify({ type: "status", state: "connected", detail: adapter.getConnectionDetail(), t: nowMs() }));
          return;
        }

        try {
          await adapter.connect();
        } catch (e) {
          const detail = `connect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }

      if (msg.cmd === "disconnect") {
        log.info(`[NETWORK] Client requested adapter disconnect`);
        if (wss.clients.size > 1) {
          log.info(`[HW-POLICY] Adapater kept open. Shared with ${wss.clients.size - 1} other active client(s).`);
          ws.send(JSON.stringify({ type: "status", state: "disconnected", detail: "logical disconnect only", t: nowMs() }));
          return;
        }

        try {
          await adapter.disconnect();
        } catch (e) {
          const detail = `disconnect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }

      if (msg.cmd === "setSimHz" && adapter instanceof SimAdapter) {
        log.info(`Setting Sim Hz to ${msg.hz}`);
        await adapter.handleCommand(msg);
        status(wss, "connected", `sim hz=${adapter.hz}`);
        return;
      }

      if (msg.cmd === "setLed") {
        try {
          await adapter.handleCommand?.(msg);
        } catch (e) {
          const detail = `command failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }
    });

    ws.on("close", () => {
      log.info(`[NETWORK] Remote Client disconnected. Total clients left: ${wss.clients.size}`);
      if (wss.clients.size === 0) {
        log.info("[HW-POLICY] No more remote clients. Auto-disconnecting adapter device to free resources...");
        adapter.disconnect();
      }
    });
  });

  if (DEVICE === "sim") {
    await adapter.connect();
  }
}

main().catch((e) => {
  log.error("Fatal:", e);
  process.exit(1);
}); 
```

### Microbit Editor 
```
from microbit import *

uart.init(115200)
display.set_pixel(0,0,9)

while True:

    x = accelerometer.get_x()
    y = accelerometer.get_y()

    a = 1 if button_a.is_pressed() else 0
    b = 1 if button_b.is_pressed() else 0

    # Convertir a 16 bits (big endian)
    xh = (x >> 8) & 0xFF
    xl = x & 0xFF

    yh = (y >> 8) & 0xFF
    yl = y & 0xFF

    # Checksum
    chk = (xh + xl + yh + yl + a + b) % 256

    # Crear paquete completo de 8 bytes
    packet = bytearray(8)
    packet[0] = 0xAA
    packet[1] = xh
    packet[2] = xl
    packet[3] = yh
    packet[4] = yl
    packet[5] = a
    packet[6] = b
    packet[7] = chk

    uart.write(packet)

    sleep(100)
```

## Bitácora de reflexión
