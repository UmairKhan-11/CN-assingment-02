# CN-assingment-02


**Course:** Computer Networks (Comp-352)  
**Semester:** 5th (Fall 2025)  
**Assignment:** 1

---

### Group Information
**Class:** BSAIF23-Green

**Group Members:**

| Name | Registration Number |
| :--- | :--- |
| Muhammad Umair Khan | B23F0001A1025 |
| Sohaib Khan | B23F0001A1045 |
| Malik Adnan | B23F0922AI038 |



# SPCP Temperature & Humidity Monitoring System

**Project Type:** Real-time IoT Monitoring
**Sensors:** Temperature (LM35/DHT11/DHT22), Humidity (DHT11/DHT22)
**Controller:** Arduino Uno
**Communication Protocol:** SPCP (Software-based Sensor-to-Server Communication Protocol)
**Display:** PC Application (Python Tkinter / C# WinForms / Java Swing / Streamlit)

---

## 1. Introduction

This project implements a real-time IoT system that reads **temperature and humidity** using sensors connected to an Arduino and sends the readings to a PC/server for live monitoring. Communication is based on **SPCP**, a simple **plain-text protocol** designed for easy debugging, reliable data transmission, and future scalability.

**Objectives:**

* Read temperature and humidity simultaneously.
* Ensure reliable message delivery using ACK/NACK.
* Provide a **user-friendly display** on PC.
* Make the system easy to replicate and extend for beginners.

---

## 2. Application Functional Requirements

| Requirement           | Description                                                    |
| --------------------- | -------------------------------------------------------------- |
| Sensor Reading        | Read temperature & humidity using LM35/DHT11/DHT22.            |
| Data Transmission     | Send readings to server at configurable intervals.             |
| ACK/NACK              | Server confirms reception (ACK) or requests resend (NACK).     |
| Error Handling        | Resend corrupted or lost data.                                 |
| Live Monitoring       | Display readings instantly on PC.                              |
| Data Logging          | Optionally log readings to CSV/database.                       |
| Configurable Interval | Example: every 5 seconds, adjustable remotely via CMD message. |

---

## 3. Application Non-Functional Requirements

* **Reliability:** Works even with lost or delayed messages.
* **Low Latency:** Display updates near real-time.
* **Scalability:** More sensors can be added later without major changes.
* **Simplicity:** Designed for beginners to implement easily.
* **Maintainability:** Extendable and well-documented code.
* **Cost-efficient:** Uses low-cost components.

---

## 4. Hardware Architecture

| Component    | Quantity | Purpose                  |
| ------------ | -------- | ------------------------ |
| Arduino Uno  | 1        | Main controller          |
| LM35/DHT11   | 1        | Temperature sensor       |
| DHT11/DHT22  | 1        | Humidity sensor          |
| USB Cable    | 1        | Connect Arduino to PC    |
| Jumper Wires | 3–5      | Wiring                   |
| Breadboard   | Optional | Easier wiring            |
| PC/Server    | 1        | Receives & displays data |

**Connections:**

**LM35:**

| Pin | Connects To   |
| --- | ------------- |
| VCC | 5V            |
| GND | GND           |
| OUT | Analog Pin A0 |

**DHT11/DHT22:**

| Pin  | Connects To   |
| ---- | ------------- |
| VCC  | 5V            |
| GND  | GND           |
| DATA | Digital Pin 2 |

---

## 5. Software Architecture

**Flow:**

```
Sensors → Arduino (pack messages) → Server → Display App (show & log)
```

**Arduino Side:**

1. Read temperature & humidity.
2. Pack readings into **SPCP messages**.
3. Send messages via serial.
4. Wait for ACK/NACK.
5. Resend if necessary.

**Server Side:**

1. Receive SPCP messages via serial port.
2. Validate and parse messages.
3. Log readings and forward to display app.
4. Respond with ACK/NACK/ERR/CMD.

**Display Application:**

1. Connect to server to receive forwarded readings.
2. Parse messages.
3. Update UI: labels, textboxes, or graphs.
4. Optionally save readings to CSV or database.

---

## 6. SPCP Protocol

### 6.1 Message Format

#### **Sensor → Arduino → Server (Reading Message)**

```
[SEQ]|[COMMAND]|[SENSOR_ID]|[VALUE]|[UNIT]
```

**Field Details:**

| Field     | Size / Bits      | Description                                                                     |
| --------- | ---------------- | ------------------------------------------------------------------------------- |
| SEQ       | 16-bit (0–65535) | Sequence number; increments each message; ensures order and detects duplicates. |
| COMMAND   | 8-bit ASCII      | Message type, e.g., `DATA`.                                                     |
| SENSOR_ID | 8–16 bytes ASCII | Unique sensor name: `TEMP1`, `HUM1`.                                            |
| VALUE     | 16–32-bit float  | Sensor reading (temperature °C / humidity %).                                   |
| UNIT      | 2–3 bytes ASCII  | Measurement unit: `C` (temperature) or `%` (humidity).                          |

**Example Messages:**

```
001|DATA|TEMP1|27.5|C
002|DATA|HUM1|55.0|%
```

---

#### **Server → Arduino (Response Message)**

```
[SEQ]|[COMMAND]|[SENSOR_ID]|[INFO]
```

**Field Details:**

| Field     | Size / Bits      | Description                                     |
| --------- | ---------------- | ----------------------------------------------- |
| SEQ       | 16-bit           | Matches the SEQ of the received message.        |
| COMMAND   | 8-bit ASCII      | Response type: `ACK`, `NACK`, `ERR`, or `CMD`.  |
| SENSOR_ID | 8–16 bytes ASCII | Identifies which sensor receives this response. |
| INFO      | 16–64 bytes      | Optional info: error reason or command details. |

**Example Responses (SENSOR_ID included in all):**

```
001|ACK|TEMP1|  
002|NACK|HUM1|Checksum Error  
003|ERR|TEMP1|Invalid Format  
  
```


---

### 6.2 Message Types

#### **A. Sensor → Server**

| Type | Format Example | Purpose |       |      |    |                                  |
| ---- | -------------- | ------- | ----- | ---- | -- | -------------------------------- |
| DATA | `001           | DATA    | TEMP1 | 27.5 | C` | Send sensor reading to server.   |
| DATA | `002           | DATA    | HUM1  | 55.0 | %` | Send humidity reading to server. |

**Explanation:**

* **SEQ:** Ensures message order and avoids duplication.
* **COMMAND:** `"DATA"` indicates sensor reading.
* **SENSOR_ID:** Unique sensor identifier.
* **VALUE:** Measurement value.
* **UNIT:** Measurement unit.

---

#### **B. Server → Sensor**

| Type | Format Example | Purpose |       |                 |                                            |
| ---- | -------------- | ------- | ----- | --------------- | ------------------------------------------ |
| ACK  | `001           | ACK     | TEMP1 | `               | Confirms successful message reception.     |
| NACK | `002           | NACK    | HUM1  | Checksum Error` | Requests resend due to corrupted message.  |
| ERR  | `003           | ERR     | TEMP1 | Invalid Format` | Indicates parsing or format error.         |


**Explanation:**

* **SEQ:** Matches the sensor's message sequence number.
* **COMMAND:** Response type (`ACK`, `NACK`, `ERR`, `CMD`).
* **SENSOR_ID:** Specifies which sensor receives this response.
* **INFO:** Error reason or command instructions.

**Reliability Note:** Sequence numbers combined with ACK/NACK ensure no readings are missed or duplicated. Each sensor uses a unique `SENSOR_ID`, allowing multiple sensors to communicate simultaneously.

---

## 7. Arduino Implementation Steps

1. Install Arduino IDE.
2. Connect sensors to Arduino (LM35/DHT11/DHT22).
3. Load required libraries for DHT/LM35 sensors.
4. Configure sensor pins in code.
5. Initialize serial communication: `Serial.begin(9600)`.
6. Read temperature & humidity values.
7. Pack readings into SPCP messages:

```cpp
String message = String(seq) + "|" + "DATA" + "|" + sensorID + "|" + String(value) + "|" + unit;
Serial.println(message);
```

8. Wait for ACK/NACK; if timeout occurs, resend.
9. Repeat based on the configured interval (e.g., every 5 seconds).

---

## 8. Server Implementation Steps

1. Open serial port (`COM3` / `/dev/ttyUSB0`).
2. Read incoming messages line by line.
3. Split message using `|`.
4. Validate fields: SEQ, COMMAND, SENSOR_ID, VALUE/UNIT.
5. If valid, log/display reading and send `ACK`.
6. If corrupted, send `NACK` with error info.
7. Optionally send `CMD` to adjust sensor settings.
8. Forward validated readings to display application.

---

## 9. Display Application Steps

1. Connect to server to receive forwarded readings.
2. Parse SPCP messages.
3. Update UI: labels, textboxes, or graphs.
4. Optionally save readings to CSV or database.
5. Recommended tools: Python Tkinter, C# WinForms, Java Swing, Streamlit.

---

## 10. Reliability Features

* **Sequence Numbers:** Maintain order & detect duplicates.
* **ACK/NACK:** Confirm or request resend of messages.
* **Timeouts:** Arduino resends if no response received.
* **ERR Messages:** Server notifies Arduino about parsing errors.

---

## 11. End-to-End Workflow

1. Sensors measure temperature & humidity.
2. Arduino forms SPCP messages and sends via serial.
3. Server validates messages, logs, and forwards to display app.
4. Server responds with `ACK`/`NACK`/`ERR`/`CMD` (SENSOR_ID included).
5. Arduino resends if required.
6. Display application shows live readings in real-time.

---

## 12. Developer Notes

* **Temperature ranges:** LM35 (-55°C to 150°C), DHT11 (0°C–50°C).
* **Humidity ranges:** DHT11 (20–90%), DHT22 (0–100%).
* Use unique `SENSOR_ID` for each sensor.
* SPCP is plain-text for easy debugging.
* Future upgrades: Wi-Fi (ESP8266/ESP32), MQTT protocol, web dashboards.

---




