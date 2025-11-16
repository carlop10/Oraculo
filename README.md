
# Implementación – Relayer FastAPI + ESP32 + Ngrok + Sepolia**

## **1. Introducción**

A continuacion se describe la implementación que se realizó de un sistema capaz de:

* Capturar lecturas de temperatura y humedad desde un ESP32 (Wokwi).
* Enviarlas a un servidor **FastAPI** que actúa como *relayer*.
* Subir los datos a **IPFS (Pinata)**.
* Firmar y enviar transacciones a la red **Ethereum Sepolia**.
* Visualizar las transacciones en **Etherscan.io**.

El despliegue se realiza **localmente** con **VSCode + Uvicorn**, y se expone a internet mediante **ngrok**, como alternativa a Render.

---

## **2. Arquitectura General**

```
DHT22 (sensor)
        │
ESP32 (Wokwi)
        │  HTTP POST
        ▼
Ngrok (túnel público)
        │
FastAPI local (Relayer)
        │
Web3.py → Smart Contract en Sepolia
        │
IPFS (Pinata)
```

---

## **3. Componentes Utilizados**

### **3.1 Hardware / Simulación**

* ESP32
* Sensor DHT22 (simulado en Wokwi)

### **3.2 Backend**

* Python 3 + FastAPI
* Web3.py 7.x
* Uvicorn
* Requests
* Dotenv
* Ngrok (modo HTTP)

### **3.3 Servicios externos**

* RPC de Alchemy (Sepolia)
* Pinata (para IPFS)
* Etherscan (verificación on-chain)

---

## **4. Implementación del Relayer FastAPI**

El relayer realiza tres funciones esenciales:

1. **Recibe lecturas del ESP32**
2. **Sube el JSON original a IPFS → obtiene el CID**
3. **Firma una transacción storeReading(...) y la envía a Sepolia**

Variables necesarias en `.env`:

```
RPC_URL=https://eth-sepolia.g.alchemy.com/v2/TU_API_KEY
PRIVATE_KEY=0xTU_PRIVATE_KEY
CONTRACT_ADDRESS=0x50268060AAd99FEdB907080Ec8138E9f4C5A0e2d
CHAIN_ID=11155111

PINATA_JWT=TU_JWT_PINATA
PINATA_URL=https://api.pinata.cloud/pinning/pinJSONToIPFS
```

El servidor se inicia con:

```
uvicorn relayer:app --host 0.0.0.0 --port 8000 --reload
```

---

## **5. Exponiendo el servidor con Ngrok**

Ngrok inicialmente generó URLs HTTPS no compatibles con ESP32 debido a limitaciones TLS de MicroPython.

La solución fue forzar una URL **HTTP**:

```
ngrok http 8000 --scheme http
```

Ejemplo de URL resultante:

```
http://xxxxx.ngrok-free.dev
```

Esa URL se configuró en el código del ESP32:

```python
RELAYER_BASE_URL = "http://xxxxx.ngrok-free.dev"
LECTURAS_URL     = RELAYER_BASE_URL + "/api/lecturas"
```

---

## **6. Implementación en ESP32 (Wokwi)**

El ESP32 realiza:

1. Conexión WiFi
2. Lectura del sensor DHT22
3. Envío periódico (cada 10 s) hacia el relayer:

   ```python
   resp = requests.post(LECTURAS_URL, data=json.dumps(payload), headers=headers)
   ```
4. Manejo de LEDs para mostrar estado de envío

La comunicación funciona correctamente en HTTP.

---

## **7. Evidencia de funcionamiento (Logs y Transacciones)**

FastAPI reporta en consola lo siguiente:

```
[DEBUG] Payload recibido: {...}
[INFO] Subido a Pinata, CID: Qm...
[INFO] Tx enviada: 9e7edea66c4d...
[INFO] Tx minada en bloque: 9644250
```

Esto confirma:

* El JSON fue recibido correctamente.
* Fue cargado en IPFS.
* La transacción fue firmada con tu wallet.
* Fue minada en la red Sepolia.

Transacciones disponibles en:

```
https://sepolia.etherscan.io/tx/0xHASH
```

---

## **8. Conclusiones**

* El ESP32 puede comunicarse correctamente con un servidor FastAPI local.
* Ngrok se configuró exitosamente para permitir tráfico HTTP compatible con MicroPython.
* Las lecturas fueron enviadas, almacenadas en IPFS y registradas en la blockchain Sepolia.
* El sistema funciona de forma completa sin necesidad de Render.
