````markdown
# TASK BRIEFING: Unified ESP32-S3 Firmware — Simultaneous Wi-Fi Captive Portal (PWA Delivery) & Coded PHY BLE Bridge

## 🎯 Executive Objective
Unify the ESP32-S3 co-processor firmware on the **Arduino UNO R4 WiFi** so that it simultaneously operates:
1. **Wi-Fi SoftAP & Captive DNS Web Server** (`SSID: LightLoop-Setup`, `Password: liftlift`, `IP: 192.168.4.1`): Serving the complete embedded, level-9 gzipped Progressive Web App (311.5 KB) directly from ESP32 flash ROM with zero internet access required.
2. **Bluetooth Low Energy (BLE) Peripheral & AT Bridge** (`LightLoop-<fwMinor>` / `LightLoop-492`): Maintaining active Web Bluetooth GATT service with the 16-byte binary protocol (CMD, MOD, WND, TAP, SAV, LND) bridging to the Renesas RA4M1 microcontroller over high-speed internal UART (`Serial2`).

Currently, these two capabilities exist as mutually exclusive binary images. Your goal is to eliminate this either/or limitation by leveraging the dual-core Xtensa LX7 architecture and FreeRTOS Wi-Fi/Bluetooth coexistence on the ESP32-S3.

---

## 📐 Hardware & Architectural Topology

```
+-----------------------------------------------------------------------------------+
|                            ARDUINO UNO R4 WIFI                                    |
|                                                                                   |
|  [ Renesas RA4M1 MCU ] (Cortex-M4F @ 48 MHz, 32 KB SRAM, 256 KB Flash)           |
|    - Runs light_loop_900.ino: 900-LED WS2812B bit-bang engine & 12x8 LED Matrix    |
|    - Communicates with ESP32-S3 over internal UART (Serial2 @ 115200 or higher)   |
|                                 ▲                                                 |
|                                 │ Internal UART Bridge                            |
|                                 ▼                                                 |
|  [ ESP32-S3 Co-Processor ] (Dual-Core @ 240 MHz, 512 KB SRAM, 4 MB Flash)         |
|    - Core 0: Wi-Fi SoftAP ("LightLoop-Setup") + DNS + HTTP WebServer (Port 80)    |
|              Serves embedded 311.5 KB PWA from flash ROM (Zero Internet)          |
|    - Core 1: BLE GATT Peripheral ("LightLoop-492") + UART Bridge to RA4M1        |
|              Native Wi-Fi / Bluetooth hardware coexistence                        |
+-----------------------------------------------------------------------------------+
```

---

## 🔍 Existing Code Assets & Working References

Before writing new code, review and inspect the following working reference assets in the repository:

1. **Standalone Captive Portal Prototype**:
   - `scratch/test_portal/test_portal.ino`: Proven working implementation of SoftAP (`LightLoop-Setup`), `DNSServer` on port 53, and `WebServer` on port 80 with captive portal probe redirection (`/generate_204`, `/gen_204`, etc.).
2. **Asset Compression & Packaging Pipeline**:
   - `scripts/package_embedded_pwa.py`: Packs `dist/` web assets with `gzip -9` and generates `embedded_pwa_data.h` and `pwa_manifest.json`.
   - `tools/esp32/firmware/embedded_pwa_data.h`: The zero-copy C header containing the 311.5 KB compressed PWA byte array table.
3. **Flashing & Automation Tooling**:
   - `scripts/deploy_esp32_portal.py`: Script compiling with `arduino-cli` and flashing via `esptool.py`.
   - `tools/esp32/firmware/UNOR4-WIFI-S3-CODED-ALL.bin`: The existing Coded PHY BLE AT bridge firmware.
   - `scripts/flash-esp32-bridge.sh` and `scripts/restore-esp32-stock.sh`: Bridge flash scripts.
4. **Main Renesas RA4M1 Firmware**:
   - `firmware/light_loop_900/light_loop_900.ino`: Production 900-LED firmware. Uses `ArduinoBLE` talking to the ESP32-S3 via AT commands over internal `Serial2`.

---

## 🛠️ Mandatory Technical Invariants & Rules

1. **Strict 31-Byte BLE Advertising Limit**:
   - Bluetooth Low Energy standard primary advertising packets are limited to 31 bytes total.
   - Broadcast **ONLY** the local advertised name (e.g. `LightLoop-492`). Do NOT call `BLE.setAdvertisedService()` in advertising packets, as UUID + Name exceeds 31 bytes and crashes the co-processor radio.
2. **UUID Specifications**:
   - Service UUID: `'19b10000-e8f2-537e-4f6c-d104768a1214'`
   - Control Characteristic UUID: `'19b10001-e8f2-537e-4f6c-d104768a1214'`
3. **Dual-Core FreeRTOS Task Separation**:
   - Pin the Wi-Fi stack and HTTP WebServer to **Core 0** (`xTaskCreatePinnedToCore`).
   - Pin the BLE GATT handler and high-speed UART bridge to **Core 1** to prevent web serving page transfers from starving the UART buffer and dropping LED animation frames.
4. **Watchdog Protection & Non-Blocking HTTP Transfers**:
   - The Renesas RA4M1 has a hardware watchdog timer (`WDT.refresh()`) that expires in 4000ms.
   - The ESP32-S3 must NEVER block internal serial responses or lock up during heavy HTTP downloads (such as when an Android device downloads the 203 KB main JavaScript chunk).
5. **Captive Portal Endpoints**:
   - Must intercept: `/generate_204`, `/gen_204`, `/connecttest.txt`, `/ncsi.txt`, `/hotspot-detect.html`, `/canonical.html` and return HTTP 302 redirecting to `http://192.168.4.1/`.
   - Web assets must be served with:
     - `Content-Encoding: gzip`
     - `Cache-Control: public, max-age=31536000, immutable`
     - `ETag` matching the asset hash.

---

## 📋 Implementation Plan

### Step 1: Create Unified ESP32-S3 Firmware Source
- Create a dedicated workspace directory: `firmware/esp32_unified_bridge/`.
- Merge the SoftAP / Captive Portal WebServer from `scratch/test_portal/test_portal.ino` with the BLE AT bridge logic.
- Include `embedded_pwa_data.h` so that static assets are mapped directly into the ESP32-S3's 4MB flash partition.

### Step 2: Build & Flash Pipeline
- Codify compilation via `arduino-cli` using FQBN `esp32:esp32:esp32s3:CDCOnBoot=cdc`.
- Output a merged binary `UNOR4-WIFI-S3-UNIFIED.bin`.
- Create a deployment script `scripts/deploy_unified_esp32.sh` to flash the ESP32-S3 using `esptool.py`.

### Step 3: Verification & Coexistence Testing
- Test that `LightLoop-Setup` appears on Wi-Fi and successfully serves `http://192.168.4.1/` (delivering HTTP 200 gzipped assets).
- Test that `LightLoop-492` simultaneously advertises over Bluetooth Low Energy.
- Perform a live stress test: Connect an Android phone via Wi-Fi and download the PWA, while concurrently connecting Chrome Web Bluetooth and scrubbing LED parameters to verify zero frame-rate degradation on the 900-LED strip.
````
