# Light Loop 26 (v2.545.0) — Builder & Software Engineer Onboarding Guide

Welcome to **Light Loop 26**! This guide bridges the gap between our **embedded C++ / React TypeScript software stack** and the **physical soldering & assembly work** we are tackling on the bench.

---

## 1. Where We Are Right Now & The Bench Gameplan

- **Software & Firmware Status (`v2.545.0`)**: Feature-complete and hardware-verified. The 60–72.4 FPS animation engine (40 modes, 7 live modulators, 12 universal landing overlays), Dual-Core ESP32-S3 Wi-Fi Captive Portal + BLE GATT bridge, Web Studio / Mobile / Pixel Fold UI, and interactive BLE Soldering Checklist are all live.
- **Our Immediate Hardware / Soldering Objective**:
  1. **Bench Station A (ElectroCookie Proto Shield REV3.3)**: Populate and solder the 3.3V-to-5V high-speed line driver shield that mounts onto the Arduino UNO R4 WiFi (`electro-cookie-interactive-wiring.html`).
  2. **Tubes 1 & 2 (two 5.0m tubes × two opposite-direction 900-LED strips = 3,600 physical LEDs)**:
     - Solder the **+5V (Red) and GND (White) power injection taps** along the 5m strips (`wiring-horizontal.html`), using the **+4 pad GND offset** so wires sit flat in the 3D-printed `wire_lock` channel without crossing.
     - Solder the **data entry taps (`T1-A..D`, `T2-A..D`)** and, for **8-Tap Mode (72.4 FPS)**, cut the thin copper `DIN` trace between LED 449 and LED 450 at the 2.5m midpoint of each strip while keeping `+5V` and `GND` continuous.
  3. **Live Hardware Continuity & Joint Verification (`checklist.html`)**: Use the 51-step interactive checklist (synced via Web Bluetooth to the Arduino) to light up the exact test LED at each solder joint (`LIGHT LED 1`, `LIGHT LED 451`, etc.) as we go.

---

## 2. Physical Topology & Why the Circuit Is Built This Way

### A. 2 Tubes × 2 Opposite Strips (Coiled 3-Turn Geometry)
- Each 5.0-meter tube houses **two 900-LED WS2812B strips mounted back-to-back** (1,800 LEDs per tube; 3,600 LEDs total across both tubes):
  - **Strip A (Forward `▶`)**: Data flows from Base (`0.0m`, LED 0) to Far End (`5.0m`, LED 899).
  - **Strip B (Reverse `◀`)**: Data flows from Far End (`5.0m`, LED 1799) back toward Base (`0.0m`, LED 900).
- Physically, each 5m tube coils into **3 concentric turns of 300 LEDs (`coilCount = 3`)**. In firmware, `applyPixelCoiled(led, r, g, b)` projects animations across all 3 turns in lockstep.

### B. 4-Tap (36.2 FPS) vs. 8-Tap (72.4 FPS) Parallel Drive
At the standard WS2812B data rate of **800 kHz** ($1.25\,\mu\text{s}$ per bit $\times 24\text{ bits} = 30\,\mu\text{s}$ per LED):
- Driving **900 LEDs** in series per pin takes $900 \times 30\,\mu\text{s} = \mathbf{27.0\text{ ms}}$ per frame (**~36.2 FPS** max).
- By cutting `DIN` at **LED 450** (the 2.5m midpoint) and feeding the second half from its own GPIO pin (**8-Tap Mode**), each pin only shifts **450 LEDs** in parallel:
  $$450 \times 30\,\mu\text{s} = \mathbf{13.5\text{ ms}}\text{ per frame} \implies \mathbf{72.4\text{ FPS}}$$

### C. Why the ElectroCookie Board Uses 6× PN2222A Emitter-Followers + 2× Direct 330Ω Lines
- **Short Base Lines (`D2 → T1-A`, `D4 → T2-A`, `< 15 cm` wire)**:
  - The wire from the Arduino enclosure to the base of Strip 1A / Strip 3A is very short (`<15 cm`), so a single **`330 Ω` series damping resistor** on Column 10 of the ElectroCookie shield cleanly terminates reflections.
- **Long Mid-Tap & Far-End Lines (`D3, D5, D6, D7, D11, D12`, `2.5m – 5.0m` wire runs)**:
  - Driving several meters of parallel cable inside the tube presents high capacitive load and line impedance.
  - Each of these 6 channels uses a **PN2222A NPN Emitter-Follower buffer** on the ElectroCookie shield:
    1. **`100 Ω` Base Resistor**: From the Arduino GPIO header pin into the Base 3-hole silver bar.
    2. **`PN2222A` (TO-92)**: Collector (Pin 3) tied directly to the center **`+5V` rail**; Base (Pin 2) in the Base bar; Emitter (Pin 1) in the adjacent Emitter bar.
    3. **`330 Ω` Emitter Pull-Down Resistor**: From the Emitter bar to the center **`GND` rail** for crisp falling edges during the strict $T_{0H} = 0.25\,\mu\text{s}$ (12 CPU clock cycles at 48 MHz) window.
    4. **`22 Ω` Series Output Resistor**: Bridges the Emitter bar to the Output bar to match the characteristic impedance of the long tube data wire.

| Arduino Pin | Tap Label | Tube & Strip Segment | LED Index Range | Mode Active | Driver Stage on ElectroCookie Shield |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`D2`** | **`T1-A`** | Tube 1 • Strip 1A Base (`▶`) | `0 .. 449` *(or `0..899` in 4-Tap)* | **4-Tap & 8-Tap** | Direct **`330 Ω`** (Col 10 Lower Row) |
| **`D3`** | **`T1-B`** | Tube 1 • Strip 1B Mid (`▶`) | `450 .. 899` *(or T2-A in 4-Tap)* | **4-Tap & 8-Tap** | **`Q6` PN2222A** (`100Ω` + `330Ω` + `22Ω`, Lower Row) |
| **`D5`** | **`T1-C`** | Tube 1 • Strip 2A Far End (`◀`) | `900 .. 1349` *(or `900..1799` in 4-Tap)* | **4-Tap & 8-Tap** | **`Q5` PN2222A** (`100Ω` + `330Ω` + `22Ω`, Lower Row) |
| **`D7`** | **`T1-D`** | Tube 1 • Strip 2B Mid (`◀`) | `1350 .. 1799` | **8-Tap Only** | **`Q3` PN2222A** (`100Ω` + `330Ω` + `22Ω`, Upper Row) |
| **`D4`** | **`T2-A`** | Tube 2 • Strip 3A Base (`▶`) | `1800 .. 2249` | **8-Tap Only** | Direct **`330 Ω`** (Col 10 Upper Row) |
| **`D11`** | **`T2-B`** | Tube 2 • Strip 3B Mid (`▶`) | `2250 .. 2699` | **8-Tap Only** | **`Q2` PN2222A** (`100Ω` + `330Ω` + `22Ω`, Upper Row) |
| **`D6`** | **`T2-C`** | Tube 2 • Strip 4A Far End (`◀`) | `2700 .. 3149` *(or `2700..3599` in 4-Tap)* | **4-Tap & 8-Tap** | **`Q4` PN2222A** (`100Ω` + `330Ω` + `22Ω`, Lower Row) |
| **`D12`** | **`T2-D`** | Tube 2 • Strip 4B Mid (`◀`) | `3150 .. 3599` | **8-Tap Only** | **`Q1` PN2222A** (`100Ω` + `330Ω` + `22Ω`, Upper Row) |

---

## 3. Software & Firmware Architecture Map

### A. Dual-MCU Embedded Architecture (Arduino UNO R4 WiFi)
The Arduino UNO R4 WiFi contains two distinct microcontrollers communicating over internal `Serial2` (115,200 baud):

1. **Renesas RA4M1 (48 MHz ARM Cortex-M4F, 256 KB Flash, 32 KB SRAM)**:
   - **Source**: `firmware/light_loop_900/light_loop_900.ino` + `firmware/light_loop_900/ws2812_parallel.S`
   - **Role**:
     - Runs the real-time **40-mode procedural & physics render engine** (`LedController` parity) and the **Universal Landing Compositor (`renderUniversalLandingOverlay`)** which additively overlays 12 touchdown detonations (L1–L12) over any running mode.
     - Executes **cycle-counted parallel port-register bit-banging** (`R_PORT1` / `R_PORT3` via `ARM_DWT_CYCCNT`) to drive multiple WS2812B data lines simultaneously with zero timing jitter.
     - Polls the **Layer-0 Sensory Bus** at 10–20 Hz non-blockingly across all modes: **SparkFun VL53L5CX 64-Zone (8×8) Time-of-Flight LiDAR** (`Wire1`) + **HC-SR04 Ultrasonic Sensor** (`D2`/`D3` or configured trigger/echo) + **DS1307 RTC** (`Wire`).
     - Preserves volatile crash telemetry (`RSTSR0`, `RSTSR1`, `g_crashContext`) in `.noinit` SRAM and displays live status/version (`545`) on the onboard 12×8 red LED matrix.

2. **Espressif ESP32-S3 Co-Processor (Dual-Core 240 MHz Xtensa LX7, 512 KB SRAM, 4 MB Flash)**:
   - **Source**: `firmware/esp32_unified_bridge/esp32_unified_bridge.ino` + `tools/esp32/firmware/embedded_pwa_data.h`
   - **Role**:
     - **Core 0 (Offline Wi-Fi Captive Portal)**: Broadcasts `SSID: LightLoop-Setup` (`Password: liftlift`, `http://192.168.4.1/`) and serves the entire **311.5 KB level-9 gzipped React PWA directly from ESP32 flash ROM**—meaning any phone or laptop in the desert with zero cell/internet service can connect to Wi-Fi and immediately load the full Light Loop Studio web app.
     - **Core 1 (BLE 5.0 Coded PHY GATT Peripheral & UART Bridge)**: Advertises `LightLoop-545` (strictly $\le 31$-byte advertisement payload) and bridges 16-byte GATT commands (`CMD`, `MOD`, `WND`, `TAP`, `SAV`, `TOF`, `CHK`) to the RA4M1 with zero-latency UART buffer detection.

3. **Dedicated Bench Soldering Harness Sketch**:
   - **Source**: `firmware/bench_harness_tester/bench_harness_tester.ino`
   - Standalone diagnostic firmware for testing individual channels, pins, and solder joints on the bench.

---

### B. Web Studio, Mobile App & Soldering Checklist (React 19 + TypeScript + Vite)

All frontend views live in the root repository and share a single state/profile engine (`UserProfileManager.ts`, `version_manifest.ts`):

| File / Entrypoint | URL Route | Purpose |
| :--- | :--- | :--- |
| `index.tsx` | `/` | `RootAppRouter` switching between Studio, Mobile Phone, Classic Desktop, Canvas, and Soldering Checklist views; manages the 4-phase Web Bluetooth post-connect settling handshake ($+550\text{ms}, +900\text{ms}, +1100\text{ms}, +1400\text{ms}$). |
| `LightLoopAdaptiveStudio.tsx` | `/?view=studio` | Primary desktop & tablet control studio: concentric 3-coil 60 FPS ring visualizer, 40-mode selector, 7 live modulators, 5-zone kinetic scrubber, and ToF LiDAR heatmap HUD. |
| `MobilePhoneApp.tsx` | `/?view=phone` | Touch-optimized mobile & Google Pixel Fold controller with single-row precision gesture scrubber and Android native BLE Coded PHY bridge support. |
| `WiringChecklistApp.tsx` | `/?view=checklist` | **51-Step Interactive Soldering Checklist** (`checklist.html`). Syncs a 64-bit completion bitmask to Arduino Data Flash (`CHK` packet) and provides one-tap **`LIGHT LED`** buttons to illuminate the exact LED at each solder joint. |
| `LedController.ts` | *(Shared Engine)* | Pure TypeScript 60 FPS simulation of all 40 firmware modes, multi-coil geometry, and particle physics so browser preview matches physical LEDs 1:1. |

---

### C. 16-Byte Binary Command Protocol (Web Bluetooth GATT & USB Serial)

Service UUID: `19b10000-e8f2-537e-4f6c-d104768a1214` | Characteristic UUID: `19b10001-e8f2-537e-4f6c-d104768a1214`

- **`CMD` (16 Bytes)**: `[0..2]='CMD'`, `[3]=mode (1..40)`, `[4]=speed*10`, `[5]=brightness (0..100)`, `[6]=origin`, `[7]=trail`, `[8]=blocks`, `[9..10]=cursorLED`, `[11]=flags`, `[12..13]=scrubFrame`, `[14]=landingPreset (1..12)`, `[15]=lineLength`.
- **`MOD` (16 Bytes)**: `[0..2]='MOD'`, `[3]=turbineHeads`, `[4]=collisionRatio`, `[5]=phosphorDecay`, `[6]=prismDispersion`, `[7]=viscosity`, `[8]=sparklerDensity`, `[9]=magnesiumWhite`.
- **`WND` (16 Bytes)**: Live anemometer wind speed, gust speed, compass direction ($0..359^\circ$), and Beaufort display style for Mode 38.
- **`TAP` (16 Bytes)**: Configures hardware segment tap count (`4-Tap` vs `6-Tap` vs `8-Tap`) and dual-strip topology (`Mirror`, `Counter-Rotating`, `90° Echo`, `Harmonic`).
- **`CHK` (16 Bytes)**: Reads/writes the 64-bit hardware soldering checklist bitmask in Data Flash and triggers pinpoint joint-test LEDs during soldering.
- **`SAV` (16 Bytes)**: Explicitly commits in-memory preset configuration to Renesas Data Flash in paced 16-byte chunks with interleaved `WDT.refresh()` calls.

---

## 4. Guide to All Files in This Google Drive Folder

1. **`00_START_HERE_BUILDER_GUIDE.html`** *(Open in Browser)* — Interactive visual version of this guide with one-click links to every wiring diagram and checklist.
2. **`electro-cookie-interactive-wiring.html`** *(Open in Browser)* — **Bench Station A**: Interactive ElectroCookie Proto Shield REV3.3 explorer (4-Tap / 8-Tap switcher, clickable D2–D12 channels, 7-step soldering isolator, zoomable PCB photo + CAD grid).
3. **`wiring-horizontal.html`** *(Open in Browser)* — **Tubes 1 & 2**: Horizontal compressed connection & pad map showing the 7 solder zones, center 5-wire channel, and +4 pad `wire_lock` GND offset.
4. **`checklist.html`** *(Open in Browser)* — **51-Step Interactive Soldering Checklist** with wire cut lengths, resistor values, pad numbers, and live BLE `TEST LED` joint verification buttons.
5. **`wiring.html`** *(Open in Browser)* — Full 5-Meter Straight-Line Vertical Blueprint (Mean Well 40A PSU, fuse block, resistor bank, wire lengths, and both 5m tubes).
6. **`wiring-split.html` & `wiring-hybrid.html`** *(Open in Browser)* — Enclosure-to-Tube disconnect harness blueprints (Option A: Split 2-Pin Power + 3-Pin Data vs. Option B: Hybrid 2P+6D connector).
7. **`electrocookie_photorealistic_wiring.png` & `full_board_photo_guide.png`** — High-resolution visual reference renders of the populated ElectroCookie shield.
8. **`electrocookie_light_loop_26.diylc`** — DIY Layout Creator CAD file for the ElectroCookie shield.
9. **`light_loop_features.md`** — Exhaustive Hardware & Software Feature Catalog.
10. **`unified_esp32_firmware_handoff.md`** — Technical spec for the Dual-Core ESP32-S3 Offline Captive Portal PWA + Coded PHY BLE Bridge.
11. **`four_line_segment_driver.md` & `power_protection_and_timing_precision.md`** — Sub-microsecond ARM DWT cycle-counted WS2812 driver math and temporal inrush slew ($di/dt$) power protection invariants.
12. **`3d_models_README.md` & `3d_arc_enclosure_preview.png`** — Mechanical spec for the 3D-printed 5m snap-together arc extrusions (`wire_lock` channel) and weatherproof desert enclosure.
